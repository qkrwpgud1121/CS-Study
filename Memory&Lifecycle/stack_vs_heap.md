# Heap & Stack

## 메모리 구조 (RAM)

- Code
    - 소스 코드가 기계어로 저장되는 곳
    - 컴파일 단계에서 결정되며 코드가 수정되지 않도록 Read-Only 형태로 저장됨
    
    
- Data
    - 전역변수(Global), 정적 변수(Static) 가 저장되는 곳
    - 프로그램 시작과 동시에 할당되며 프로그램이 종료될때 해제됨
    - 값이 변경될 수 있어 Read-Write 형태로 저장됨
    
    
- Heap
    - Class, Closure 등 크기가 크고 여러곳에서 **참조 타입**의 실제 데이터가 저장되는 곳
    - 런타임 시에 동적으로 할당된다.
    - 사람이 직접 할당, 해제 할 수 있다 (동적 할당).
    - **사용 후 반드시 메모리 해제를 해줘야 한다.**
    - Swift에서는 **ARC(Automatic Reference Counting)**을 통해 참조 횟수를 추적하여 자동으로 해제해 준다.
        
    - 장점
        - 사용한 메모리 한도 내에서 **제한 없음**
        - 프로그램 내부의 다른곳, 다른 스레드 에서 참조 및 공유가 가능 하다.
        
    - 단점
        - 빈 공간 탐색, 할당으로 인해 스택 보다 속도가 느리다.
        - 모든 스레드가 공유하는 공간으로, 여러 곳에서 동시에 접근할 경우 **경합(Race Condition)**이 발생
        - 경합을 막기위해 **동기화(Lock)** 처리 때문에 추가적인 오버헤드 발생
        - 직접 관리 해주지 않으면 **메모리 누수(Memory Leak)** 발생
        
        
- Stack
    - 지역 변수, 함수의 매개변수, 리턴값, **Struct** 등이 잠시 머무는 곳
    - 컴파일 단계에서 필요한 공간의 크기가 미리 결정되며 실질적 할당은 런타임에 함수가 호출될때 이루어 진다.
    - 함수( `{}`스코프 )가 종료되면 **자동으로 메모리에서 즉각 해제** 된다.
    - Last In First Out 의 데이터 구조를 가지고있다.
    
    - 장점
        - CPU의 스택 포인터만 이동시켜 할당하므로 Heap보다 압도적으로 빠르다 (오버헤드가 0에 가깝다).
        - 메모리를 **직접 해제 해주지 않아도 됨**
        - **각 스레드 마다 독립적인 스택**을 가지고있어 스레드 서로의 간섭을 받지 않아 안전하다 (Thread-Safe).
        
    - 단점
        - 한번에 할당할 수 있는 메모리의 크기에 제한이 있다.
        - 함수 밖을 벗어나면 데이터가 사라지므로 수명이 짧다.
       
    - 예제
    ```swift
    struct Person { // 컴파일 단계에서 코드영역에 저장
    
        // 컴파일 단계에서 데이터 영역에 자리만 만듬 실제로 데이터를 메모리에 올리지는 않음
        static let country: String = "대한민국"
        var age: Int
        
        // lazy 변수는 반드시 var로 선언해야함 ( 처음에는 값이 없으며 나중에 값이 변경이 되기 때문에 )
        // lazy 변수는 반드시 초기값을 설정해야함 ( 클로저를 많이 사용함 )
        lazy var profileImage: String = {
            print("이미지 다운로드 중..")
            return "sampleImage.png"
        }()
    }

    // 비어있던 데이터 영역의 country 자리에 "대한민국" 이라는 문자열을 메모리에 올림
    // 앱이 꺼질때 까지 메모리를 사용함
    print(Person.country)
    
    // 이때 P는 Stack에 저장되며 age(8바이트) 공간에 30이 할당됨
    // 여기서 Stack에 p의 상태 : [ age: 30 | profileImage: nil ]
    var p = Person(age: 30)
    
    // 구조체 내부의 static 변수는 데이터 영역에 저장되므로 메모리의 크기를 잡아 먹지 않음
    // 구조체 내부에 lazy 변수가 존재 하기 때문에 컴파일 단계에서 미리 Stack에 공간을 할당을 해놓음
    print("메모리 크기 : \(MemoryLayout.size(ofValue: p))")
    
    // 여기서 lazy 변수의 값을 불러오게 됨
    // Stack의 p 변수 안에 존재하는 profileImage이 nil 인것을 확인
    // 컴파일 단계에서 코드 영역에 저장했던 클로저를 실행
    // String 형식의 profileImage를 Heap에 저장
    // Stack의 p 변수 안에 존재하는 profileImage의 값을 Heap의 포인터값을 저장
    // 여기서 Stack에 p의 상태 : [ age: 30 | profileImage: 0x1234(Heap 포인터) ]
    print("첫 번째 호출")
    print("프로필 이미지 :\(p.profileImage)")
    
    // Stack의 p 변수 안에 존재하는 profileImage의 값이 Heap 포인터 인것을 확인
    // 클로저는 무시하고 바로 Heap으로 가서 "sampleImage.png"를 읽어옴
    print("첫 번째 호출")
    print("프로필 이미지 :\(p.profileImage)")
    ```
        

## Heap, Stack 사용

- Stack을 기본으로 사용
    - 데이터가 복잡한 상속이나 참조가 필요하지 않다면 무조건 Struct를 만들어 Stack에 할당하는 것이 성능과 안전성에서 유리하다.
    - 한계 이상으로 재귀 함수를 호출하거나 너무 큰 Struct를 만들면 **Stack Overflow**가 발생한다.
    
    
- Heap은 Stack외 사용
    - 앱 전체에서 하나의 상태를 공유해야할때 사용 ex) NetworkManager
    - 런타임에 데이터의 크기가 동적으로 변하거나, 함수가 끝나도 데이터가 살아 있어야 할때 사용
    - 무한정으로 객체를 생성하고 헤제하지 못하면 **Out Of Memory, Heep Overflow**가 발생한다.


## Heap, Stack 물리적 관계
- Heap과 Stack은 하나의 RAM 공간 안에서 빈 메모리 영역을 위아래로 공유한다.
    - Heap은 낮은 메모리 주소(`0x00000000`) 부터 높은 주소 방향으로 할당 된다.
    - Stack은 높은 메모리 주소(`0xffffffff`) 부터 낮은 주소 방향으로 할당 된다.
    - **결국 Heap과 Stack이 만나 영역을 침범하는 순간 메모리가 가득 찼다는 뜻이며 Crash가 발생한다**


## 예외 상황

- Struct의 내부에 Class가 존재한다면
    - Struct 자체는 Stack에 저장된다.
    - 하지만 Struct 내부의 Class는 실제 데이터가 아니라 Heap을 가리키는 포인터(주소값)일 뿐이다.
    - 복사된 Struct는 완전히 새로운 것이지만 내부의 Class는 Heap내부의 원본을 가리킨다.
    - Struct를 썻음에도 불구하고 두 변수가 하나의 데이터를 공유하게 되어 값이 변경되는 오류(Reference Semantics)가 발생한다.
    
- Array, String, Dictionary
    - Stack은 컴파일시 크기가 정해진다 하지만 컬렉션 타입들는 앱을 사용하다보면 크기가 계속 늘어날수도 있다.
    - 해서 컬렉션 타입들 자체는 Stack에 포인터, 길이 정보 정도만 저장 한다.
    - Array, String, Dictionary 내부의 거대한 데이터는 Heap에 저장하게 된다.
    - 컬렉션 타입들은 이름만 Struct이며 실제로는 Heap의 메모리를 사용하고 **Copy-on-Write**라는 것을 사용한다.
