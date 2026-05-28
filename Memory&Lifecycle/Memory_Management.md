# ARC (Automatic Reference Counting)

- `자동 참조 계수`라고도 하며 iOS에 도입된 **메모리 관리 시스템**이다.
- ARC는 메모리 영역 주 `Heap`영역의 관리를 도와주는 시스템이다.

- 인스턴스, 클로저 등의 `참조 타입`은 자동으로 `Heap`에 할당이 된다.

```swift
class Person {
    var name: String
    var age: Int
    
    init(name: String, age: Int) {
        self.name = name
        self.age = age
    }
    
}

func printAddress() {
    var john = Person(name: "john", age: 29)
    
    withUnsafePointer(to: &john) { pointer in
        print("john 변수의 Stack 주소: \(pointer)")                      // 0x000000016fdfee48
    }
    
    let johnsHeapAddress = Unmanaged.passUnretained(john).toOpaque()
    print("john 변수의 실제 데이터가 있는 Heap 주소: \(johnsHeapAddress)")    // 0x00000001007f6150
    
    var may = john
    
    withUnsafePointer(to: &may) { pointer in
        print("may 변수의 Stack 주소: \(pointer)")                       // 0x000000016fdfee20
    }
    
    let maysHeapAddress = Unmanaged.passUnretained(may).toOpaque()
    print("may 변수의 실제 데이터가 있는 Heap 주소: \(maysHeapAddress)")      // 0x00000001007f6150
    
    print(john.name, john.age)                                        // john 29
    print(may.name, may.age)                                          // john 29

}

**Stack의 john -> Heap의 Person 인스턴스를 참조**
**Stack의 may -> Heap의 Person 인스턴스를 참조**

결론: 두개의 객체가 하나의 class 참조한다. 변수는 Stack에 저장되지만 Heap에 저장된 하나의 Person 인스턴스를 참조한다.
```

- `Heap`의 메모리는 사용후에 반드시 메모리 해제를 해줘야 한다.
- `Stack`에 저장되었던 변수가 함수종료 시점에 소멸하면 **`Heap`에 저장 되어있던 class 인스턴스의 메모리 해제를 `ARC`가 자동으로 해준다.**

### **ARC (Automatic Reference Counting)란 class Instance가 더 이상 필요 하지 않을때 메모리를 자동으로 해제 한다.**

## Garbage Collection(Java) VS Reference Counting(Swift)

| | GC | RC |
| :-- | :-- | :-- |
| 작동 시기 | 런타임(프로그램 실행 중 주기적으로 작동) | 컴파일 타임(컴파일 시점에서 참조, 해제가 결정됨) |
| 주체 | Garbage Collector | Swift Compiler |
| **순환 참조** | 낮음 | 개발자가 직접 해결해야함 |
| CPU 부하 | 주기적으로 메모리를 확인해야 하므로 오버헤드 발생 | 참조 카운트만 계산하므로 오버헤드 적음 |
| 메모리 해제 | 가득 찰때까지 기다리다가 한번에 정리 (언제 지워질지 모름) | **RC가 0이 되는 즉시 해제 (예측 가능)** |
| **치명적 단점** | Garbage Collector가 돌아갈 때 앱이 순간적으로 버벅임 발생 | 개발자의 실수로 순환 참조 발생 시 영구적인 메모리 누수 발생 |

