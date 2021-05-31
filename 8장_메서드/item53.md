# Item 53. 가변인수는 신중히 사용하라

원서 제목: **Use varargs judiciously(분별력 있게, 현명하게)**


### 핵심 정리

> 인수 개수가 일정하지 않은 메서드를 정의해야 한다면 가변인수가 반드시 필요하다. 메서드를 정의할 때 필수 매개변수는 가변인수 앞에 두고, 가변인수를 사용할 때는 성능 문제까지 고려하자.
> -출처:  이펙티브 자바

Swift 문서에 programming language 문서(swift 5.3)에 소개된 Variadic Parameter에 대해 소개합니다.
### Variadic Parameters

``` swift
func functionName(parameter1: Type, parameter2: Type...) -> returnType {
  // implement
  return
}
```

**Variadic**: '임의의 갯수 인수를 받을 수 있는' 

즉, **Variadic parameter** 는 0개 이상의 특정 타입 인수를 받을 수 있는 매개변수를 말하며 메서드가 호출될 때 전달되어 사용 가능합니다. **Variadic parameter** 를 사용하기 위해서는 매개변수 타입 이름 뒤에 `...` 를 붙여 사용합니다. **Variadic parameter** 에 전달된 값은 함수 본문 내에서 적절한 타입의 배열로 사용할 수 있습니다. 예를 들어 `doubleArray` 를 이름으로 갖고있는 `Double...` 타입의 **Variadic parameter** 는 `[Double]` 타입의 상수 배열로 함수 본문 내에서 사용할 수 있습니다.

<u>**Variadic parameter**는 인수 개수가 정해지지 않았을 때 유용합니다.</u> 


### 활용 예시

* 인수가 1개 이상이어야 할 때 가변인수를 제대로 사용하는 방법
```swift
static func min(firstArg: Int, remainArgs: Int...) -> Int {
    var min = firstArg
    for arg in remainArgs {
        if arg < min {
            min = arg
        }
    }
    return min
}
```

* 산술 평균 구하기 (주어진 수의 합을 수의 개수로 나눈 값)

```swift
// 산술 평균 구하기 (주어진 수의 합을 수의 개수로 나눈 값)
func arithmeticMean(_ numbers: Double...) -> Double {
    var total: Double = 0
    for number in numbers {
        total += number
    }
    return total / Double(numbers.count)
}
arithmeticMean(1, 2, 3, 4, 5) // returns 3.0
arithmeticMean(3, 8.25, 18.75) // returns 10.0
```

* 합 구하기
```swift
func sum(integers: Int...) -> Int { 
    return integers.reduce(0,combine: +) 
} 

sum(1,2,3,4,5,6,7,8,9) // returns 45

sum(1,2,3) // returns 6
```

* subView 추가하기
```swift
// 화면 바탕에 깔리는 백그라운드 뷰의 역할을 하는 Canvas 클래스.
class Canvas {
    func add(_ views: UIView...) {
      for view in views {
        self.addSubView(view)
      }
    }
}

let circle = Circle(center: CGPoint(x: 5, y: 5), radius: 5)
let lineA = Line(start: .zero, end: CGPoint(x: 10, y: 10))
let lineB = Line(start: CGPoint(x: 0, y: 10), end: CGPoint(x: 10, y: 0))

let canvas = Canvas()
canvas.add(circle, lineA, lineB)
```

* API에 첨부파일을 보내기
```swift
func send(_ message: Message, attaching attachments: Attachment...) {
    ...
}

// Passing no variadic arguments:
send(message)

// Passing either a single, or multiple variadic arguments:
send(message, attaching: image)
send(message, attaching: image, document)
```

### Variadic Parameters vs Array Parameters

사실, 함수 안에서 Variadic Parameter도 Array로 사용되기 때문에 Variadic Parameter와 Array 둘 중에 어떤 것을 사용하더라도 문제가 없습니다. 그럼 어떤 상황에서 Variadic Parameters를 쓰면 좋은지, Array Parameters를 쓰면 좋은지 비교해봅시다.

* Variadic Parameters의 특징

> **NOTE**
> A function may have at most one variadic parameter.
>
> -출처: the swift programming languageswift 5.3

1. 함수는 최대 하나의 Variadic Parameters를 가질 수 있습니다. 

2. 함수 호출 시 여러 인자들과의 모호성을 피하기 위해 Variadic Parameters는 항상 메서드 인자 목록 마지막에 위치를 합니다.

3. 함수가 하나 이상의 기본 값을 가지는 인자와 Variadic Parameters를 가지고 있다면, 기본값을 가지는 인자 뒤에 Variadic Parameters 가 위치해야 합니다.

4. 매개변수로 넘길 값이 없을 경우 함수 호출시 아규먼트 레이블을 비워도 됩니다.
0개 이상의 매개변수에 대한 입력이 배열로 변환된다고 이해하면 쉽습니다.
   
   ```swift
   func printNums(items: Int...) {
       for item in items {
            print (item)
        }
   }
   printNums() // presents no output, as no items are specified
   printNums(items: 1, 2) // outputs 1 and 2 to the console.
   ```

이 특징을 고려했을 때, 함수에서 Variadic Parameters를 두 개 이상 사용해야 하는 경우나 Variadic Parameters가 함수 인자 목록에서 마지막에 위치하는 것이 코드의 가독성을 해친다면 Array Parameters를 고려하는게 좋습니다.


```swift
// 컴파일 오류가 발생하는 코드
func shoppingList(items: String..., prices: Float...) { ... }
```


* Array Parameters의 특징

1. 매개변수로 넘길 값이 없을 경우에도 함수 호출시 빈 배열을 아규먼트 레이블에 명시해야합니다.

```swift
func printNums(items: [Int]) {
    for item in items {
         print (item)
     }
}
printNums([]) // presents no output, as no items are specified
printNums(items: 1, 2) // outputs 1 and 2 to the console.
```

이 특징을 고려했을 때, 매개변수의 수가 0개인지 그 이상인지 확실하지 않을 경우에는 굳이 빈 배열을 만들지 않아도 되는 Variadic Parameters를 고려하는게 좋습니다.




### 참고

1. [Variadic Parameters - the swift programming language swift 5.3](https://docs.swift.org/swift-book/LanguageGuide/Functions.html#ID166)
2. [Variadic Parameters v Array Parameter [closed]](https://stackoverflow.com/questions/30572738/variadic-parameters-v-array-parameter)
3. [#15 Using variadic parameters](https://github.com/JohnSundell/SwiftTips#15-using-variadic-parameters)
4. [The power of variadic parameters](https://www.swiftbysundell.com/tips/the-power-of-variadic-parameters/)

