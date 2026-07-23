# Retain Cycle (순환 참조 Closure)

## Retian Cycle과 Auto Reference Count 의 관계

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
