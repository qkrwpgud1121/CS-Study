# Class & Struct

## Memory Alignment (메모리 정렬)
- Struct
    - Struct 내부에서 변수를 선언할때 8바이트의 Int, Double을 먼저 선언하고 그아래에 4바이트의 Int32, 그 아래에 1바이트의 Bool을 선언하면 8의 배수인 메모리의 기준선에 맞춰 끼워 맞춰지므로 빈공간없이 저장 할수 있다.
    - 네트워크 통신으로 서버에서 대량의 데이터를 받아올때 사용하는 DTO 같은 Struct는 무조건 정렬을 해주어야 한다. 
    
    ```swift
    struct Person {
        var a: Bool // 1 byte + (7 bytes padding)
        var b: Int  // 8 bytes
        var c: Bool // 1 byte + (7 bytes padding)
    }
    
    print(MemoryLayout<Person>.size)    // 출력: 17
    print(MemoryLayout<Person>.stride)  // 출력: 24
    
    struct Person2 {
        var b: Int  // 8 bytes
        var a: Bool // 1 byte
        var c: Bool // 1 byte + (6 bytes padding)
    }
    
    print(MemoryLayout<Person2>.size)   // 출력: 10
    print(MemoryLayout<Person2>.stride) // 출력: 16
    ```
    
- Class
    - Class는 데이터가 Heap에 저장되고 Stack에는 8Byte의 포인터만 들고있기 때문에 순서를 바꾸더라도 체감되는 성능, 메모리 절약 효과가 미미하다.

## Identity (정체성)

- Struct (값 타입)
    - 완전히 똑같은 복사본을 만든다면 두 Struct간의 **식별 연산자 `===(Identity Operator)`** 자체를 사용할 수없다.
    - 즉 이 두개의 Struct는 **Stack**의 다른 위치에 존재하는 주소가 다른 완전한 별개 이다.
    - 오직 내부의 값이 같은지만 확인하는 `==`만 사용가능 하다.
    - 각각의 Struct 내부의 값을 변경할 수 있다.
    
    ```swift
    struct PersonStruct {
        var name: String
    }
    
    // Code 영역을 이용해 Stack에 첫 번째 실체 생성한다.
    // p1이 원본
    var p1 = PersonStruct(name: "철수")
    
    // p1을 p2에 넣을때 스택에 p1과 완전히 똑같이 생긴 복사본이 생성된다.
    var p2 = p1
    
    // 복사본의 값을 변경
    p2.name = "영희"
    
    print(p1.name) // 출력: 철수 (원본 데이터는 변경 되지 않는다.)
    print(p2.name) // 출력: 영희 (결국 복사본의 데이터만 변경이 된다.)
    ```
    
- Class (참조 타입)
    - **Heap**에 있는 단 하나의 진짜 데이터를 여러 포인터에서 가리킨다.
    - 즉 같은 주소인지 확인하는 **식별 연산자 `===(Identity Operator)`**를 사용할 수 있다.
    - 하나의 포인터에서 Class 내부의 값을 변경한다면 다른 포인터에서도 변경된 값을 바라본다.
    
## Mutability (불변성)

- Struct
    - Stack에 저장되며 만약 `var p1 = PersonStruct()` 으로 선언을 한다면 Stack 내부 변수를 수정을 할수 없는상태가    된다.
    - 이에 내부 변수를 수정하려면 컴파일러에서 오류가 발생하므로 이 때에 사용하는 것이 `mutating`을 사용한다.
    
    - **mutating**
        - 오직 구조체(Struct) 내부의 함수가 자기 자신의 값을 바꿀때만 필요하다.
        
        ```swift
        struct PersonStruct {
            var name: String
            
            // 커파일 오류 발생
            // Cannot assign to property: 'self' is immutable
            func changeNameError() {
                self.name = "영희"
            }
            
            // mutating을 붙여줘야 오류가 발생하지 않음
            mutating func changeNameSuccess() {
                self.name = "영희"
            }
        }
        
        var p1 = PersonStruct(name: "철수")
        print(p1.name) // 출력: 철수
        
        p1.changeNameSuccess()
        print(p1.name) // 출력: 영희
        ```
        
        - 구조체 외부에서 값을 변경할때는 `mutating`이 필요없다.
        
        ```swift
        struct PersonStruct {
            var name: String
        }
        
        // 값을 수정할수 있는 PersonStruct를 Stack에 생성
        var p1 = PersonStruct(name: "철수")
        print(p1.name) // 출력: 철수
        
        // 외부에서 PersonStruct의 값을 수정
        // 외부에서 값을 변경하여 mutating 필요 없음
        p1.name = "영희"
        print(p1.name) // 출력: 영희
        ```
        
        - **왜 내부 함수에서만 `mutating`을 강제하나**
            - 컴파일러에서 Struct의 내부 함수를 실행할때 자기 자신(self)을 함수에 전달한다.
            - 이때 기본적으로 컴파일러는 함수 안에서 실수로 Struct내부의 값(Stack의 데이터)이 변경되면 안되기 때문에읽기 전용으로 전달한다.
            - 이에 컴파일러에게 허락을 받아야 하므로 반드시 `mutating`을 사용해야 한다.
    
- Class
    - Heap에 저장되며 `var c = Car()` 으로 선언을 한다고 해도 Stack에 저장된 포인터 만 수정 할수 없기에 Heap에 있는 데이터를 수정할 수 있다.

## Method Dispatch (속도의 차이)

- Struct
    - 상속이 불가능 하며 오직 하나뿐이다.
    - 컴파일러가 앱을 빌드할 때 **이 구조체는 무조건 이 주소에 있다. 라고 기계어로 정의한다 (Static Dispatch**).
    - 찾아 다니는 과정이 없으니 속도가 Class 보다 훨씬 빠르다.
    
- Class
    - 상속이 가능하며 부모 클래스의 함수를 자식 클래스 함수가 Override 할수 있다.
    - 앱이 실행될때 부모 함수를 실행해야할지 자식 함수를 실행해야 할지 **매번 런타임에 vTable 이라는 것을확인해야한다(Dynamic Dispatch**)
    - 매번 런타임에 vTable이라는 것을 확인해야 하기 때문에 Struct 보다 속도가 느리다.

## 초기화 (Initialization)
- Struct
    - 직접 `init`을 하지 작성하지 않아도 컴파일러가 변수들을 초기화할 수 있는 **Memberwise Initializer**를 만들어 준다.
    
- Class
    - 변수에 기본값이 없다면 무조건 직접 `init`을 작성하여 모든 변수를 초기화 해주어야만 컴파일 오류가 발생하지 않는다.
    
**Class 에서 Memberwise Initializer이 없는 이유는 자동화된 기능이 문제를 일으킬 여지가 있으며 자동적으로 작성되는 것보다 직접 작성하는것이 오히려 편할 수도 있다.**

## Deinitialization
- Struct
    - 함수가 종료가 되면 Stack영역에서 포인터 이동만으로 흔적도 없이 사라지기 때문에 `deinit`자체가 필요가 없다.
- Class
    - Heap영역에서 메모리 해제가 될때 소멸되기 직전 `deinit {}`블록이 호출 된다.
    - 여기서 `Notification`, `Observer`등을 해제하는 동작을 할 수 있다.

## Objective-C 호환성
- Struct
    - 순수한 Swift기반이므로 Objective-C 런타임에서는 Struct를 전혀 인식하지 못한다.
- Class
    - Apple의 오래된 프레임워크나 Objective-C 기반의 코드와 교류를 해야 할때 사용할 수 있다.
    - `@Objc`, `NSObject`등의 상속이 가능하다.
