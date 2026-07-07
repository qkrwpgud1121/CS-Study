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
    
    **순환 참조**
        - Strong으로 서로를 참조했을때 변수에 `nil`을 할당하여 인스턴스의 참조가 제거 되었을때 인스턴스가 서로의 RC를 붙잡고 있는 현상
        - 이런 경우 서로의 `deinit`이 호출 되지 않으면서 **순환 참조**가 발생한다.
        - 결국 변수에 `nil`을 할당이 되면서 인스턴스에 접근할 수 있는 방법도 없기 때문에 메모리에서 해제를 하지 못한다.
        - 해당 경우 `Memory Leak`이 발생을 하면서 메모리를 지속적으로 갉아 먹는다.
        - 결국 아이폰의 OS에서 메모리를 비정상적으로 많이 사용한다고 생각하여 앱을 강제 종료 시킨다 이것이 **OOM (Out Of Memory) Crash** 이다.
    
    ```swift
    class Person {
        var name: String
        var age: Int
        var apartment: Apartment?
        
        init(name: String, age: Int) {
            self.name = name
            self.age = age
        }
        
        deinit {
            print("deinit Person")
        }
        
    }
    
    class Apartment {
        var unit: String
        var tenant: Person?
    
        init(unit: String) {
            self.unit = unit
        }
    
        deinit {
            print("deinit Apartment")
        }
    }
    
    func printAddress() {
        
        var john: Person? = .init(name: "john", age: 29)    // Class Person RC + 1,     Class Person RC = 1
        var unit302: Apartment? = .init(unit: "302")        // Class Apartment RC + 1,  Class Apartment RC = 1
        
        unit302?.tenant = john              // Class Person RC + 1,                     Class Person RC = 2
        john?.apartment = unit302           // Class Apartment RC + 1,                  Class Apartment RC = 2
        
        print(CFGetRetainCount(john))       // CFGetRetainCount로 인한 RC + 1,            Class Person RC = 3
        print(CFGetRetainCount(unit302))    // CFGetRetainCount로 인한 RC + 1,            Class Apartment RC = 3
        
        /*
        print(CFGetRetainCount(john))      // CFGetRetainCount 종료로 인한 RC - 1,        Class Person RC = 2
        print(CFGetRetainCount(unit302))   // CFGetRetainCount 종료로 인한 RC - 1,        Class Apartment RC = 2
        */
        
        // 순환 참조 발생
        john = nil      // Class Person RC - 1,         Class Person RC = 1
        unit302 = nil   // Class Apartment RC - 1,      Class Apartment RC = 1
        
        // Person 인스턴스 내부의 선언된 apartment 에서 Apartment 인스턴스를 그대로 참조하고 있음
        // Apartment 임스턴스 내부에 선언된 tenant 에서 Person 인스턴스를 그대로 참조 하고 있음
        
        // Stack 메모리에서 Heap에 저장되어있는 인스턴스를 참조하는게 없기 때문에 Heap에 저장되어있는 Person, Apartment 인스턴스의 RC를 제어할 수 없음
        
    }
    ```
    
    **순환 참조를 막기위해 사용하는 것이 Weak, Unowned가 있다.**
    
### Weak (약함 참조)
    - 인스턴스를 참조하지만 객체의 생명주기에 관여하고 싶지 않을때 사용
    - 인스턴스의 Weak Refence Count만 + 1 한다.
    - 참조하던 인스턴스의 Strong Reference Count가 0이 되어 메모리 해제가 되면 자동으로 `nil`을 할당하여 메모리에서 해제 한다.
    - 언제든지 값이 `nil`로 바뀔수 있어야 하므로 반드시 `var`변수로 선언을 해야하며 `Optional`타입이어야만 한다.
    
    ```swift
    class Person {
        var name: String
        var age: Int
        weak var apartment: Apartment?
        
        init(name: String, age: Int) {
            self.name = name
            self.age = age
        }
        
        deinit {
            print("deinit Person")
        }
        
    }
    
    class Apartment {
        var unit: String
        var tenant: Person?
    
        init(unit: String) {
            self.unit = unit
        }
    
        deinit {
            print("deinit Apartment")
        }
    }
    
    func printAddress() {
    
        var john: Person? = .init(name: "john", age: 29)    // Class Person RC + 1,     Class Person RC = 1
        var unit302: Apartment? = .init(unit: "302")        // Class Apartment RC + 1,  Class Apartment RC = 1
        
        unit302?.tenant = john              // Class Person RC + 1,                     Class Person RC = 2
        john?.apartment = unit302           // Person 내부의 Apartment 참조를 weak로 선언했기 때문에 Apartment의 RC가 오르지 않는다., Class Apartment RC = 1
        
        print(CFGetRetainCount(john))       // CFGetRetainCount로 인한 RC + 1,            Class Person RC = 3
        print(CFGetRetainCount(unit302))    // CFGetRetainCount로 인한 RC + 1,            Class Apartment RC = 2
        
        /*
        print(CFGetRetainCount(john))      // CFGetRetainCount 종료로 인한 RC - 1,        Class Person RC = 2
        print(CFGetRetainCount(unit302))   // CFGetRetainCount 종료로 인한 RC - 1,        Class Apartment RC = 1
        */
        
        // 순환 참조 발생
        john = nil      // Class Person RC - 1,         Class Person RC = 1
        unit302 = nil   // Class Apartment RC - 1,      Class Apartment RC = 0
        
        // 1. Apartment의 RC가 0이 되어 Heap 메모리에서 해제 된다. (deinit Apartment)
        // 2. Apartment가 메모리에서 헤재되며 tenant 변수도 함께 제거된다.
        // 3. tenant가 제거되며 Person 역시 RC = 0 이되므로 역시 Heap에서 메모리가 해제가 된다. (deinit Person)
        // 4. 이때 Person에서 weak var apartment는 Side Table에 의해 안전하게 nil로 변환된다.
        
        // 출력 결과
        // deinit Apartment
        // deinit Person
    }

    ```

    **weak를 어디에 선언을 해줘야 하나**
    - 1. 수직 관계 ( 부모, 자식 / 갑, 을 )
        - 이런 경우 을 및 자식에 `weak` 를 선언 하여 자식이 부모의 RC를 올리지 않도록 한다.
        - 갑이 소멸될경우 자식들도 같이 소멸되어야 하기 때문이다.
        - 만약 반대로 갑 및 부모에 `weak`를 선언한다면 을, 자식은 생성이 되자마자 소멸되어 버리는 현상이 발생한다.
        
        ```swift
        class ViewController {
    
            var popup: CustomPopup?
            
            func showPopup() {
                popup = CustomPopup()
                
                popup?.delegate = self
            }
            
            func popupDidClose() {
                
                print("팝업이 닫혔습니다.")
                
                self.popup = nil
            }
            
            deinit { print("갑(화면) 메모리 해제 완료") }
        }
        
        class CustomPopup {
            
            weak var delegate: ViewController?
            
            func closeButton() {
                delegate?.popupDidClose()
            }
            
            deinit { print("을(팝업) 메모리 해제 완료") }
        }
        
        func view() {
            
            var vc: ViewController? = .init()   // 갑의 위치인 ViewController의 RC + 1
            
            vc?.showPopup()                     // 을의 위치인 CustomPopup의 RC + 1
                                                // 여기서 CustomPopup에서 ViewController를 weak 선언하여 바라만 보게 함으로 ViewController의 RC가 증가하지 않는다.
            
                                                // ViewController의 RC = 1, CustomPopup의 RC = 1
                                                
            
            vc?.popup?.closeButton()            // 을의 위치인 CustomPopup의 RC - 1
                                                // 여기서 실무에서 사용하는 delegate 패턴을 보여준다.
                                                // ViewController에 있는 popupDidClose에 콜백을 사용할 수 있다.
            
                                                // ViewController의 RC = 1, CustomPopup의 RC = 0
            
            vc = nil                            // 갑의 위치인 ViewController의 RC - 1
                                                // ViewController가 메모리에서 해제된다
            
                                                /*
                                                예외 상황
                                                vc?.popup?.closeButton() 를 하지않고 바로 ViewController가 메모리에서 해제가 되는 상황에서도
                                                CustomPopup은 ViewController를 weak를 사용해서 바라만보도록 설정하였으므로 ViewController의 RC가 0이 되어 메모리에서 해제가 되면 CustomPopup의 RC를 잡고있던 ViewController가 제거되면서
                                                팝업을 사용자가 닫지 않고 예외상황으로 메인뷰가 먼저 닫혀도 팝업의 RC가 0이므로 자동으로 메모리에서 해제되어 메모리 누수가 발생하지 않는다.
                                                */
                                                
                                                // ViewController의 RC = 0, CustomPopup의 RC = 0
        }
        ```
        
    - 2. 평행 관계 ( 자식, 자식 / 을, 을 )
        - 웬만하면 자식과 자식끼리는 서로 직접적으로 참조할 수 없도록 하고 부모의 통제를 받도록 하는것이 좋다.
        - 양방향 링크드 리스트처럼 반드시 서로를 참조를 해야한다면 데이터가 흘러가는 정방향 (앞 -> 뒤)은 `Strong`, 역방향 (뒤 -> 앞)은 `weak`를 선언한다.
        - 즉 시간의 흐름에 따라 도미노 처럼 소멸 되도록 한다.
        
        ```swift
        class Node {
    
            var value: String
            
            var next: Node?
            
            weak var prev: Node?
            
            init(value: String) {
                self.value = value
            }
            
            deinit { print("\(value) 노드 메모리 해제 완료") }
        }
        
        func linkedList() {
            
            var firstNode: Node? = .init(value: "앞")        // firstNode의 Node 인스턴스의 RC + 1
            let secondeNode: Node? = .init(value: "중간")     // secondeNode의 Node 인스턴스의 RC + 1
            let thirdNode: Node? = .init(value: "뒤")        // thirdNode의 Node 인스턴스의 RC + 1
            
                                                            // firstNode의 RC = 1, secondeNode의 RC = 1, thirdNode의 RC = 1
            
            firstNode?.next = secondeNode                   // secondeNode의 Node 인스턴스의 RC + 1
            secondeNode?.next = thirdNode                   // thirdNode의 Node 인스턴스의 RC + 1
            
                                                            // firstNode의 RC = 1, secondeNode의 RC = 2, thirdNode의 RC = 2
            
            thirdNode?.prev = secondeNode                   // thirdNode의 prev는 weak로 선언 하였기 때문에 secondNode의 RC를 변경하지 않는다.
            secondeNode?.prev = firstNode                   // secondNode의 prev는 weak로 선언 하였기 때문에 firstNode의 RC를 변경하지 않는다.
        
                                                            // firstNode의 RC = 1, secondeNode의 RC = 2, thirdNode의 RC = 2
            
            firstNode = nil                                 // firstNode의 RC = 0, secondeNode의 RC = 1, thirdNode의 RC = 2
            
                                                            /*
                                                            firstNode = nil을 할때와 함수가 종료되어 지역 변수가 메모리에서 해제될때의 차이
                                                            1. firstNode = nil할때 지역변수 firstNode가 완전히 제거 되는것이 아닌 내부의 포인터 값이 제거된다.
                                                            2. 이때 Heap에 저장되어있는 firstNode가 참조하고있던 인스턴스의 RC 값은 0이 된다 즉 Stack과 Heap의 참조가 끊어진다.
                                                            3. 함수가 종료될때 지역변수가 Stack 에서 완전한 메모리 해제 된다.
                                                            4. 즉 firstNode = nil을 하지 않아도 Heap에서의 메모리 누수는 발생하지 않는다.
                                                            */
            
        }                                                   // 함수가 종료됨에 따라 지역변수인 firstNode, secondeNode, thirdNode 가 Stack에서 메모리 해제를 하며 참조를 하던 Heap의 각각의 Node 인스턴스의 RC - 1
        ```
    
### Unowned (미소유 참조)
    - Weak와 마찬가지로 생명주기에는 관여하지 않지만 변수가 인스턴스를 참조하는 중에는 인스턴스가 메모리에서 해제되지 않는다고 확신할 때 사용
    - 인스턴스의 Unowned Reference Count만 + 1 한다.
    - 참조하는 인스턴스의 Reference Count가 0이 되어 메모리에서 해제가 되더라도 변수의 값이 nil로 바뀌지 않고 기존 인스턴스의 주소값을 그대로 가지고 있다.
    - 해서 변수에 접근 하는 순간 앱의 크래시가 발생한다.
    - nil이 될수 없다고 가정하므로 기본적으로는 Non-Optional 타입으로 선언하며 값이 바뀔일이 없다면 let으로도 선언이 가능하다.
    - ex) `unowned let unownedRef: Person = john!`

### 비교표

| | Strong | Weak | Unowned |
| :-- | :-- | :-- | :-- |
| 선언 방식 | `var`, `let` | `weak var` | `unowned var`, `unowned let` |
| RC Count | +1 | 증가 하지 않음 | 증가 하지 않음 |
| Optional | 둘다 가능 | **Optional** | Non-Optional (Swift 5.0 부터 Optional 가능) |
| 대상 소멸시 상태 | 소멸하지 않음 | 자동으로 `nil`변환 | 소멸된 주소 그대로 유지 |
| 소멸 후 접근 | 해당 사항 없음 | `nil` 반환 | Crash 발생 |

