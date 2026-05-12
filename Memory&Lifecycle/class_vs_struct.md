# Class & Struct

## Identity

- Struct (값 타입)
    - 완전히 똑같은 복사본을 만든다면 두 Struct간의 **식별 연산자 `===(Identity Operator)`** 자체를 사용할 수 없다.
    - 즉 이 두개의 Struct는 **Stack**의 다른 위치에 존재하는 주소가 다른 완전한 별개 이다.
    - 오직 내부의 값이 같은지만 확인하는 `==`만 사용가능 하다.
    - 각각의 Struct 내부의 값을 변경할 수 있다.
    
- Class (참조 타입)
    - **Heap**에 있는 단 하나의 진짜 데이터를 여러 포인터에서 가리킨다.
    - 즉 같은 주소인지 확인하는 **식별 연산자 `===(Identity Operator)`**를 사용할 수 있다.
    - 하나의 포인터에서 Class 내부의 값을 변경한다면 다른 포인터에서도 변경된 값을 바라본다.
    
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

## Mutability

- Struct
    - Stack에 저장되며 만약 var p1 = PersonStruct() 

