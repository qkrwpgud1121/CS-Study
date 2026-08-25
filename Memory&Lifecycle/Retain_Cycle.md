# Retain Cycle (순환 참조 Closure)

## Retain Cycle과 ARC(Auto Reference Count) 의 관계

- Swift의 메모리 관리 시스템인 ARC의 규칙으로 인해 발생하는 메모리 누수 즉 **순환 참조**를 **Retain Cycle** 이라고 하며 이것을 해결하기 위해 ARC에서 제공하는 `weak`, `unowned`를 사용하여 순환 참조가 발생하여 메모리 누수가 되는 상황을 방지 할수 있다.

### Retain Cycle
- 두 개 이상의 참조 타입(인스턴스 <> 인스턴스, 인스턴스 <> 클로저)이 서로를 강한 참조를 하여 서로를 메모리에서 강하게 붙잡고 있는 상황
- 서로의 **ARC** 의 **참조 카운트(RC)**가 앱이 종료가 될때까지 0으로 변하지 않아 메모리에서 `deinit` 되지 못하여 **메모리 누수(Memory Leak)**를 일으키는 상황이 발생 

### Memory Management ARC(Auto Reference Count)
- Swift의 Heap 영역에서 메모리를 관리하는 시스템이며 객체가 메모리에서 언제까지 살아있어야 하는지를 **참조 카운트(RC)**로 추적한다.
- 참조 타입이 참조될 때마다 ARC가 **참조 카운트(RC)**를 1 증가시키고 참조가 해제될때 RC를 1 감소 시키며 RC가 0이 될때 메모리에서 해제한다.
- Retain Cycle을 막기 위해서는 ARC에서 제공하는 참조를 하지만 RC는 증가시키지 않는 `weak`(약한 참조, 참조 대상이 메모리에서 해제될시 자동으로 `nil`을 할당)와 `unowned`(미소유 참조)를 사용하여 순환 참조를 방지할 수 있다.


## Retain Cycle 종류

### Instance <> Instance
```swift
class Person {
    var myDog: Dog?
}

class Dog {
   weak var owner: Person? // Instance <> Instance 의 Retain Cycle 에 대한 weak 선언
}

let jason = Person()
let baduk = Dog()

jason.myDog = baduk
baduk.owner = jason

jason.myDog = nil
baduk.owner = nil
```

- 위의 코드에서 Dog 인스턴스의 owner 변수 앞 `weak`가 선언 되어있지 않다면 하단에 있는 각각의 인스턴스 변수에 nil을 할당을 한다고 해도 Person과 Dog가 서로의 RC를 1로 만들고 있어 메모리 누수가 발생 한다.

### Instance <-> Closure
```swift
class ImageDownloader {
    var onDownloadComplete: (() -> Void)?
    
    func download(completion: @escaping () -> Void) {
        // 전달받은 클로저를 자신의 프로퍼티에 강하게 저장 한다.
        self.onDownloadComplete = completion
        
        // 가상의 다운로드 작업 (3초 뒤 완료된다고 가정)
        DispatchQueue.main.asyncAfter(deadline: .now() + 3.0) {
            self.onDownloadComplete?()
        }
    }
}

class ProfileViewController: UIViewController {
    var userName = "호두 아키텍트"
    
    // ImageDownloader 인스턴스를 생성하고 강하게 소유 한다.
    let downloader = ImageDownloader()
    
    func updateProfile() {
        // 다운로드 메서드를 실행하며 클로저를 넘겨준다.
        downloader.download { [weak self] in
            guard let self else { return }      // self 의 옵셔널 방어
            // 완료 시점에 UI를 업데이트
            print("\(self.userName)님의 프로필 사진 업데이트 완료!")
        }
    }
}
```

- 여기서는 ImageDownloader의 download의 Closure에 `[weak self]` 를 사용하지 않는다면 ProfileViewController에서 먼저 ImageDownloader 인스턴스를 붙잡으면서 시작을 하여 updateProfile을 실행하며 ImageDownloader의 download로 Closure를 넘겨주며 그걸 받은 ImageDownloader 에서는 Closure를 붙잡고 Closure는 다시 ProfileViewController를 붙잡는 형식의 Retain Cycle이 발생하게 된다.
- 흐름: ProfileViewController -> ImageDownloader -> download Closure -> ProfileViewController

## Retain Cycle 해결 방법

### 클로저에서 Side Table을 활용한 Retain Cycle 방어

- `[weak self]`
    - 탈출 클로저에서 `self`를 사용해 객체와 클로저가 강하게 붙잡고 있는 것을 방지 하기 위해 클로저 에 `[weak self]` in 을 사용하여 `Side Table`이라는 본체를 대신한 대리자를 통해 본체가 소멸됬을때 클로저의 `guard let self else { return }`나 `self?.`에 `nil`을 반환하여 안전하게 메모리에서 해제될수 있도록 한다.
    - 또한 `guard let self = self else { return }`을 사용하여 클로저 내부의 `self?.`옵셔널 미리 해제 하여 클로저 내부에서는 `?`없이 `self.`로 편하게 접근할 수 있도록 해준다.
    
- `[unowned self]`
    - `[unowned self]`역시 Retain Cycle을 방지하지만 Side Table을 생성하지 않아 본체가 소멸됐을때 안전하게 `nil`을 반환할수 없다.
    - 따라서 비동기 작업중에 본체가 이미 메모리에서 해제되었는데 뒤늦게 클로저가 실행이 될경우 존재 하지 않는 메모리에 접근하려 시도하면서 앱이 즉시 강제 종료 되는 Crash가 발생한다.

## 클로저의 종류와 예외 상황

### 탈출 클로저에서만 Retain Cycle이 발생하는 이유

- 비탈출 클로저 (`filter`, `map`, `forEach`) 같은 클로저 즉 고차함수 들은 함수가 실행될때 사용되고 함수가 종료되면 메모리에서 즉시 해제가 되기 때문에 Retain Cycle이 발생하지 않는다.
- 탈출 클로저 (`@Escaping`, `Completion`)에서는 함수가 종료되었는데도 나중에 실행되기 위해 메모리의 어딘가 저장되어 남아있는 클로저 이기 때문에 `self`를 사용하여 강하게 참조를 한다면 객체와 클로저가 서로를 놓지 못하여 Retain Cycle이 발생한다.

### UI Component의 lazy var(지연 초기화)에서의 클로저

```swift
class ProfileHeaderView: UIView {
    var userName = "호두 아키텍트"

    // 클로저를 이용해 라벨을 초기화합니다.
    lazy var greetingLabel: UILabel = {
        let label = UILabel()
        label.text = "\(self.userName)님, 환영합니다!"
        return label
    }()
}
```

- 위에서의 UI Component의 lazy var(지연 초기화)의 클로저 에서는 Retain Cycle이 발생하지 않는다.
- `lazy var`에 할당된 클로저는 greetingLabel이 처음 호출되는 시점에 딱 한 번만 실행된다.
- 이후 label을 생성해 반환한 직후 클로저 자체는 메모리에서 즉시 해제되므로 클로저가 `self`를 붙잡지 않는다.

- 흐름: ProfileHeaderView -> greetingLabel 반환 후 즉시 클로저 소멸

### 시스템의 소유하는 비동기 처리(`DispatchQueue`, `UIView.animate`)

```swift
class APIService {
    func fetchWeather(completion: @escaping (String) -> Void) {
        // 서버 통신을 3초 뒤에 완료한다고 가정합니다.
        DispatchQueue.main.asyncAfter(deadline: .now() + 3.0) {
            completion("맑음")
        }
    }
}

class WeatherViewController: UIViewController {
    var weatherStatus = "로딩중..."
    let apiService = APIService() // 뷰컨트롤러가 apiService를 강하게 소유 한다.

    func loadWeather() {
        // apiService를 통해 날씨를 불러 온다.
        apiService.fetchWeather { result in
            self.weatherStatus = result
            print("현재 날씨: \(self.weatherStatus)")
        }
    }
}
```

- 위의 코드에서는 WeatherViewController에서 APIService를 소유 한다.
- 하지만 APIService는 전달받은 클로저를 자신의 프로퍼티로 저장하지 않고 그대로 `DispatchQueue`로 넘겨준다.
- `DispatchQueue`에서 3초 동안 클로저를 소유하며 작업이 끝나면 클로저를 놓아주어 메모리에서 해제 되며 Retain Cycle이 발생하지 않는다.

- 흐름
    - ViewController: WeatherViewController -> APIService
    - Closure: DispatchQueue -> Closure -> WeatherViewController
