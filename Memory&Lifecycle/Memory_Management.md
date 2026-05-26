# Automatic Reference Counting


```swift
class Person {
    var name: String
    var apartment: Apartment?
    
    init(name: String) {
        self.name = name
    }
    
    deinit { print("\(name) 메모리에서 소멸됨")}
}

class Apartment {
    var unit: String
    weak var tenant: Person?
    
    init(unit: String) {
        self.unit = unit
    }
    
    deinit { print("\(unit) 메모리에서 소멸됨")}
}

var john: Person? = Person(name: "john")
var xi: Apartment? = Apartment(unit: "xi")

print("johns first RC: \(CFGetRetainCount(john))") // 2
print("xi first RC: \(CFGetRetainCount(xi))") // 2

john?.apartment = xi

print("xi RC Change: \(CFGetRetainCount(xi))") // 3

xi?.tenant = john

print("john RC Change: \(CFGetRetainCount(john))") // 2

john = nil // john 메모리에서 소멸됨

print("xi`s tenant: \(String(describing: xi?.tenant))") // nil

xi = nil // xi 메모리에서 소멸됨
```
