# Retain Cycle (순환 참조 Closure)

## Retain Cycle과 Auto Reference Count 의 관계

- Swift의 메모리 관리 시스템인 ARC의 규칙으로 인해 발생하는 메모리 누수 즉 **순환 참조**를 **Retain Cycle** 이라고 하며 이것을 해결하기 위해 ARC에서 제공하는 `weak`, `unowned`를 사용하여 순환 참조가 발생하여 메모리 누수가 되는 상황을 방지 할수 있다.

### Retain Cycle
- 두 개 이상의 참조 타입(인스턴스 <-> 인스턴스, 인스턴스 <-> 탈출 클로저)이 서로를 강한 참조를 하여 서로를 메모리에서 강하게 붙잡고 있는 상황
- 서로의 **ARC** 의 **참조 카운트(RC)**가 앱이 종료가 될때까지 0으로 변하지 않아 메모리에서 `deinit` 되지 못하여 **메모리 누수(Memory Leak)**를 일으키는 상황이 발생 

### Memory Management (Auto Reference Count)
- Swift의 Heap 영역에서 메모리를 관리하는 시스템이며 객체가 메모리에서 언제까지 살아있어야 하는지를 **참조 카운트(RC)**로 추적한다.
- 참조 타입이 참조될 때마다 ARC가 **참조 카운트(RC)**를 1 증가시키고 참조가 해제될때 RC를 1 감소 시키며 RC가 0이 될때 메모리에서 해제한다.
- Retain Cycle을 막기 위해서는 ARC에서 제공하는 참조를 하지만 RC는 증가시키지 않는 `weak`(약한 참조, 참조 대상이 메모리에서 해제될시 자동으로 `nil`을 할당)와 `unowned`(미소유 참조)를 사용하여 순환 참조를 방지할 수 있다.


## Retain Cycle 종류

### Instance <-> Instance
```swift
class Person {
    var myDog: Dog?
}

class Dog {
   weak var owner: Person?
}

let jason = Person()
let baduk = Dog()

jason.myDog = baduk
baduk.owner = jason
```

### Instance <-> Closure
```swift
class iOSDeveloper {
    let name: String
    var developApp: (() -> Void)?
    
    init(name: String) {
        self.name = name
        print("\(name) 초기화 완료")
    }
    
    deinit { print("\(name) 메모리 헤제 완료") }
    
    func startDevelop() {
        developApp = { [weak self] in
            guard let self = self else { return }
            print("\(self.name) starting develop")
        }
    }
}

func startDevelop() {
        
    var dev: iOSDeveloper? = iOSDeveloper(name: "iOS 2년차 아키택트")
    
    dev?.startDevelop()
    dev?.developApp?()
    
    dev = nil
}
```

## 비탈출 클로저와 탈출 클로저의 차이

### 탈출 클로저에서만 Retain Cycle이 발생하는 이유

- 비탈출 클로저 (`filter`, `map`, `forEach`) 같은 클로저 즉 고차함수 들은 함수가 실행될때 사용되고 함수가 종료되면 메모리에서 즉시 해제가 되기 때문에 Retain Cycle이 발생하지 않는다.
- 탈출 클로저 (`@Escaping`, `Completion`)에서는 함수가 종료되었는데도 나중에 실행되기 위해 메모리의 어딘가 저장되어 남아있는 클로저 이기 때문에 `self`를 사용하여 강하게 참조를 한다면 객체와 클로저가 서로를 놓지 못하여 Retain Cycle이 발생한다.

### 클로저에서 Side Table을 활용한 Retain Cycle 방어

- 탈출 클로저에서 `self`를 사용해 객체와 클로저가 강하게 붙잡고 있는 것을 방지 하기 위해 클로저 에 `[weak self]` in 을 사용하여 `Side Table`이라는 본체를 대신한 대리자를 통해 본체가 소멸됬을때 클로저의 `guard let self = self else { return }`나 `self?.`에 `nil`을 반환하여 안전하게 메모리에서 해제될수 있도록 한다.
- 또한 `guard let self = self else { return }`을 사용하여 클로저 내부의 `self?.`옵셔널 미리 해제 하여 클로저 내부에서는 `?`없이 `self.`로 편하게 접근할 수 있도록 해준다.
