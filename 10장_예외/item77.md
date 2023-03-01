# Item 77. 예외를 무시하지 말라



### 책 요약

> API 설계자가 메서드 선언에 예외를 명시하는 이유는 그 메서드를 사용할 때 적절한 조치를 취해달라고 말하는 것 입니다.
>
> 메서드 호출을 try 문으로 감싼 후 catch 블록에서 아무일도 하지 않으면 예외를 무시하는 것 입니다.
>
> 예외는 문제 상황에 잘 대처하기 위해 존재하는데 catch 블록을 비워두면 예외가 존재할 이유가 없어집니다. 
>
> 예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓도록 합시다.
>
> 예측할 수 있는 예외 상황이든 프로그래밍 오류든, 빈 catch 블록으로 못 본척 지나치면 그 프로그램은 오류를 내재한 채 동작하게 됩니다. 그러다가 어느 순간 문제의 원인과 아무 상관없는 곳에서 갑자기 죽어버릴 수도 있습니다. 
>
> 예외를 적절히 처리하면 오류를 완전히 피할 수도 있습니다. 무시하지 않고 바깥으로 전파되게만 놔둬도 최소한 디버깅 정보를 남긴 채 프로그램이 신속히 중단되게는 할 수 있습니다.



### Swift에서의 Error Handling

swift programming language guide에서는 오류가 발생할 때, 일부 주변 코드는 오류의 처리를 책임져야 한다고 언급하고 있습니다. (예를 들어, 문제를 수정하거나 대안적 접근법을 시도하거나 사용자에게 고장을 알리는 것)

여기서 제시하는 스위프트에서 오류를 처리하는 방법은 네 가지가 입니다. 

1. 함수에서 함수를 호출하는 코드로 오류를 전파 (Propagating Errors Using Throwing Functions)
2. do-catch 문을 사용하여 오류를 처리 (Handling Errors Using Do-Catch)
3. 오류를 Optional 값으로 처리 (Converting Errors to Optional Values)
4. 오류가 발생하지 않는다고 주장 (Disabling Error Propagation)

함수가 오류를 범하면 프로그램의 흐름이 변경되므로 코드에서 오류를 범할 수 있는 위치를 신속하게 식별할 수 있어야 합니다. 코드에서 이러한 위치를 식별하려면 오류를 던질 수 있는 함수, 메서드 또는 이니셜라이저를 호출하는 코드 조각 앞에 try 키워드(또는 try?` or `try! 변형)를 사용하라고 명시하고 있습니다. 

> Note (참고용)
>
> NOTE
>
> Error handling in Swift resembles exception handling in other languages, with the use of the `try`, `catch` and `throw` keywords. Unlike exception handling in many languages—including Objective-C—error handling in Swift doesn’t involve unwinding the call stack, a process that can be computationally expensive. As such, the performance characteristics of a `throw` statement are comparable to those of a `return` statement.



#### 1. Propagating Errors Using Throwing Functions

```swift
enum VendingMachineError: Error {
    case invalidSelection
    case insufficientFunds(coinsNeeded: Int)
    case outOfStock
}

struct Item {
    var price: Int
    var count: Int
}

class VendingMachine {
    var inventory = [
        "Candy Bar": Item(price: 12, count: 7),
        "Chips": Item(price: 10, count: 4),
        "Pretzels": Item(price: 7, count: 11)
    ]
    var coinsDeposited = 0

    func vend(itemNamed name: String) throws {
        guard let item = inventory[name] else {
            throw VendingMachineError.invalidSelection  // throw 예시 ⭐️ 
        }

        guard item.count > 0 else {
            throw VendingMachineError.outOfStock  // throw 예시 ⭐️ 
        }

        guard item.price <= coinsDeposited else {
            throw VendingMachineError.insufficientFunds(coinsNeeded: item.price - coinsDeposited)   // throw 예시 ⭐️ 
        }

        coinsDeposited -= item.price

        var newItem = item
        newItem.count -= 1
        inventory[name] = newItem

        print("Dispensing \(name)")
    }
}

// 사용
let favoriteSnacks = [
    "Alice": "Chips",
    "Bob": "Licorice",
    "Eve": "Pretzels",
]
func buyFavoriteSnack(person: String, vendingMachine: VendingMachine) throws {
    let snackName = favoriteSnacks[person] ?? "Candy Bar"
    try vendingMachine.vend(itemNamed: snackName) // try 사용 ⭐️ 
}

```

`vend(itemNamed:)` 메서드는 모든 오류를 전파하므로 이 메서드를 호출하는 모든 코드는 `do-catch` 문 또는 `try?` 또는 `try!` 을 사용하여 오류를 처리하거나 오류를 계속 전파합니다.

#### 2. Handling Errors Using Do-Catch

Do-catch 문을 사용하여 코드 블록을 실행하여 오류를 처리할 수 있습니다. 만약 'do' 절의 코드에 의해 에러가 발생한다면, 그것은 'catch' 절과 일치하여 그 중 어느 하나가 에러를 처리할 수 있는지를 결정한다. 

오류가 발생하면 즉시 실행은 캐치 절로 전송되며, 캐치 절은 전파를 계속할 것인지 여부를 결정합니다. 일치하는 패턴이 없으면 최종 캐치 절에 의해 오류가 발생하고 로컬 오류 상수에 바인딩됩니다. 오류가 발생하지 않으면 'do' 문의 나머지 문이 실행됩니다.



```swift
var vendingMachine = VendingMachine()
vendingMachine.coinsDeposited = 8
func nourish(with item: String) throws {
    do {                                            // do - catch 예시 ⭐️ 
        try vendingMachine.vend(itemNamed: item)
    } catch is VendingMachineError {                // do - catch 예시 ⭐️ 
        print("Couldn't buy that from the vending machine.")
    }
}

do {                                             // do - catch 예시 ⭐️ 
    try nourish(with: "Beet-Flavored Chips")
} catch {                                        // do - catch 예시 ⭐️ 
    print("Unexpected non-vending-machine-related error: \(error)")
}
// Prints "Couldn't buy that from the vending machine."


func eat(item: String) throws {
    do {                                        // do - catch 예시 ⭐️ 
        try vendingMachine.vend(itemNamed: item)
    } catch VendingMachineError.invalidSelection, VendingMachineError.insufficientFunds, VendingMachineError.outOfStock {                // do - catch 예시 ⭐️ 
        print("Invalid selection, out of stock, or not enough money.")
    }
}
```

#### 3. Converting Errors to Optional Values

try?를 사용하여 오류를 Optional 값으로 변환합니다. try? 식을 평가하는 동안 오류가 발생하는 경우 식 값은 'nil'입니다. try?'를 사용하면 모든 오류를 동일한 방식으로 처리하고자 할 때 간결한 오류 처리 코드를 작성할 수 있습니다.

```swift
func fetchData() -> Data? {
    if let data = try? fetchDataFromDisk() { return data }
    if let data = try? fetchDataFromServer() { return data }
    return nil
}
```

#### 4. Disabling Error Propagation

때때로 던지기(throwing) 기능이나 방법은 실제로 런타임에 오류를 던지지 않는다는 것을 알 수 있습니다. 이러한 경우 표현식 앞에 `try!`를 작성하여 오류 전파를 비활성화하고 오류가 발생하지 않는다는 런타임 어설션(assertion, 주장)으로 호출을 마무리할 수 있습니다. (하지만 실제로 오류가 발생하면 런타임 오류가 발생합니다.)

예를 들어 다음 코드는 이미지 로드(경로:) 함수를 사용하여 지정된 경로에 이미지 리소스를 로드하거나 이미지를 로드할 수 없는 경우 오류를 발생시킵니다. 이 경우 이미지가 애플리케이션과 함께 제공되므로 런타임에 오류가 발생하지 않으므로 오류 전파를 사용하지 않도록 설정하는 것이 좋습니다.

```swift
let photo = try! loadImage(atPath: "./Resources/John Appleseed.jpg")
```



### 참고 

[Error Handling - the swift programming language swift 5.4](https://docs.swift.org/swift-book/LanguageGuide/ErrorHandling.html)

