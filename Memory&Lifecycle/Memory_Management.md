# ARC (Automatic Reference Counting)

## Garbage Collection(Java) VS Reference Counting(Swift)

| | GC | RC |
| :-- | :-- | :-- |
| 작동 시기 | 런타임(프로그램 실행 중 주기적으로 작동) | 컴파일 타임(컴파일 시점에서 참조, 해제가 결정됨) |
| 주체 | Garbage Collector | Swift Compiler |
| **순환 참조** | **GC가 스스로 제거함** | 개발자가 직접 해결해야함 |
| CPU 부하 | 주기적으로 메모리를 확인해야 하므로 오버헤드 발생 | 참조 카운트만 계산하므로 오버헤드 적음 |
| 메모리 해제 | 가득 찰때까지 기다리다가 한번에 정리 (언제 지워질지 모름) | **RC가 0이 되는 즉시 해제 (예측 가능)** |
| **치명적 단점** | Garbage Collector가 돌아갈 때 앱이 순간적으로 버벅임 발생 | 개발자의 실수로 순환 참조 발생 시 영구적인 메모리 누수 발생 |


## 참조 타입

- 인스턴스, 클로저 등의 `참조 타입`은 자동으로 `Heap`에 할당이 된다.

```swift
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
    deinit {
        print("deinit Person")
    }
    
}

func printAddress() {
    var john = Person(name: "john", age: 29)                          // Class Person RC + 1
    
    withUnsafePointer(to: &john) { pointer in
        print("john 변수의 Stack 주소: \(pointer)")                      // 0x000000016fdfee48
    }
    
    let johnsHeapAddress = Unmanaged.passUnretained(john).toOpaque()
    print("john 변수의 실제 데이터가 있는 Heap 주소: \(johnsHeapAddress)")    // 0x00000001007f6150
    
    var may = john                                                    // Class Person RC + 1
    
    withUnsafePointer(to: &may) { pointer in
        print("may 변수의 Stack 주소: \(pointer)")                       // 0x000000016fdfee20
    }
    
    let maysHeapAddress = Unmanaged.passUnretained(may).toOpaque()
    print("may 변수의 실제 데이터가 있는 Heap 주소: \(maysHeapAddress)")      // 0x00000001007f6150
    
    print(john.name, john.age)                                        // john 29
    print(may.name, may.age)                                          // john 29

}   // 함수 종료와 함께 john, may 지역 변수 제거됨
    // john이 사라지면서 Person의 RC -1
    // may가 사라지면서 Person의 RC -1
    // Person 클래스의 deinit 발생

**Stack의 john -> Heap의 Person 인스턴스를 참조**
**Stack의 may -> Heap의 Person 인스턴스를 참조**

결론: 두개의 객체가 하나의 class 참조한다. 변수는 Stack에 저장되지만 Heap에 저장된 하나의 Person 인스턴스를 참조한다.
```

- `Heap`의 메모리는 사용후에 반드시 메모리 해제를 해줘야 한다.
- `Stack`에 저장되었던 변수가 함수종료 시점에 소멸하면 **`Heap`에 저장 되어있던 class 인스턴스의 메모리 해제를 `ARC`가 자동으로 해준다.**

## **ARC (Automatic Reference Counting)의 메모리 관리**

- `자동 참조 계수`라고도 하며 iOS에 도입된 **메모리 관리 시스템**이다.
- ARC는 메모리 영역 주 `Heap`영역의 관리를 도와주는 시스템이다.

- `Reference Count`즉 참조 수를 계산하여 참조 수가 0이 되면 메모리에서 제거한다.
- 모든 인스턴스는 각자의 `Reference Count`값을 가지고 있다. 

### `Reference Count`값 증가

1. 인스턴스 생성 및 할당
    - `Heap`에 새로운 인스턴스를 생성하고, 이를 변수, 상수에 처음 할당할때 RC + 1
    - ex) `var john = Person(name: "john", age: 29)`
    
2. 기존 인스턴스를 다른 변수에 대입 (참조 공유)
    - 이미 생성된 인스턴스를 다른 변수에 대입하여 여러개의 변수가 하나의 인스턴스를 동시에 참조할때 RC + 1
    - ex) `var may = john`
    
### `Reference Count`값 감소

1. 변수에 `nil` 할당
    - 인스턴스를 참조하던 변수에 `nil`을 할당하여 참조를 명시적으로 끊을때 RC - 1
    - ex) `john = nil`
    
2. 다른 인스턴스 대입
    - 인스턴스를 참조하던 변수에 다른 인스턴스를 덮어씌우면 기존 인스턴스의 참조가 끊어져 기존의 인스턴스 RC - 1
    - `may = Person(name: "Tom", age: 30)`
    
    - ex) 기존의 may 변수가 참조하던 `Person(name: "john", age: 29)`의 RC - 1, 새로운 인스턴스인 `Person(name: "Tom", age: 30)`의 RC + 1
    
3. 스코프의 종료
    - 지역 변수가 선언된 함수나 조건문 등의 `{ ~ }` 중괄호 블록이 끝나면 Stack에서 지역변수를 메모리에서 제거 하기 때문에 이때 변수가 가지고 있던 참조도 제거되며 참조 인스턴스의 RC - 1
    
### `Reference Count` 0 (메모리 해제)
    - 위의 RC 의 값이 0이 되는 즉시 `ARC`는 해당 인스턴스를 `Heap`메모리에서 완전히 제거한다.
    - 제거 되기 직전 인스턴스 내부에서 `deinit { }`이 자동으로 호출되며 제거된다.


## Strong, Weak, Unowned

### Strong (강함 참조)
    - 가장 기본적인 참조 방식
    - 변수를 선언할 때 앞에 아무것도 선언을 하지 않으면 무조건 `Strong` 참조가 된다.
    - 인스턴스의 Strong Reference Count를 직접 + 1 한다.
    - ex) `var john = Person(name: "john", age: 29)`
    
### Weak (약함 참조)
    - 인스턴스를 참조하지만 객체의 생명주기에 관여하고 싶지 않을때 사용
    - 인스턴스의 Weak Refence Count만 + 1 한다.
    - 참조하던 인스턴스의 Strong Reference Count가 0이 되어 메모리 해제가 되면 자동으로 `nil`을 할당하여 메모리에서 해제 한다.
    - 언제든지 값이 `nil`로 바뀔수 있어야 하므로 반드시 `var`변수로 선언을 해야하며 `Optional`타입이어야만 한다.
    - ex) `weak var may: Person? = john`

### Unowned (미소유 참조)
    - Weak와 마찬가지로 생명주기에는 관여하지 않지만 변수가 인스턴스를 참조하는 중에는 인스턴스가 메모리에서 해제되지 않는다고 확신할 때 사용
    - 인스턴스의 Unowned Reference Count만 + 1 한다.
    - 참조하는 인스턴스의 Reference Count가 0이 되어 메모리에서 해제가 되더라도 변수의 값이 nil로 바뀌지 않고 기존 인스턴스의 주소값을 그대로 가지고 있다.
    - 해서 변수에 접근 하는 순간 앱의 크래시가 발생한다.
    - nil이 될수 없다고 가정하므로 기본적으로는 Non-Optional 타입으로 선언하며 값이 바뀔일이 없다면 let으로도 선언이 가능하다.
    - ex) `unowned let unownedRef: Person = john!`
