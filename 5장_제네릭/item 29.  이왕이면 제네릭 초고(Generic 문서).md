# item 29.  이왕이면 제네릭 초고

클라이언트에서 직접 형변환해야 하는 타입보다 제네릭 타입이 더 안전하고 쓰기 편리합니다. 따라서 새로운 타입을 설계할 때는 형변환 없이도 사용할 수 있도록 하는 것이 좋고 그러기 위해 기존 타입 중 제네릭이었어야 하는 게 있다면 제네릭 타입으로 변경하는 것을 권장하고 있습니다.

### 예시

* 제네릭 타입을 사용하지 않았을 경우

```swift
struct IntStack {
    var items = [Int]()
    mutating func push(_ item: Int) {
        items.append(item)
    }
    mutating func pop() -> Int {
        return items.removeLast()
    }
}
```



* 제네릭 타입을 사용했을 경우

```swift
struct Stack<Element> {
    var items = [Element]()
    mutating func push(_ item: Element) {
        items.append(item)
    }
    mutating func pop() -> Element {
        return items.removeLast()
    }
}
```



### 참고

* [Generic - The Swift Programming Language](https://docs.swift.org/swift-book/LanguageGuide/Generics.html#ID184) 