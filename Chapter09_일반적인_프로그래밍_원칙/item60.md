# Item.60 정확한 답이 필요하다면 float과 double은 피하라

## 부동소수 타입 float, double

float과 double 타입은 과학과 공학 계산용으로 설계되었습니다. 이진 부동소수점 연산에 쓰이며, 넓은 범위의 수를 빠르게 정밀한 '근사치'로 계산하도록 세심하게 설계되었습니다. **따라서 정확한 결과가 필요할 때는 사용하면 안됩니다. float과 double타입은 특히 금융 관련 계산과는 맞지 않습니다.** 0.1 혹은 10의 음의 거듭제곱 수(10^-1,10^-2 등)를 표현할 수 없기 때문입니다.

```swift
print(1.03 - 0.42) 

// 출력 결과 
// 0.6100000000000001
```
=> 실제로 계산해보면 결과는 0.61이지만 부동소수점 오차 때문에 0.6100000000000001이 출력됩니다.

```swift
print(1.00 - 9 * 0.10)  

// 출력 결과 
// 0.09999999999999998
```
=> 실제로 계산해보면 결과는 0.1이지만 부동소수점 오차 때문에 0.09999999999999998이 출력됩니다. 

결과값을 출력하기 전에 반올림하면 해결되리라 생각할지 모르지만, 반올림을 해도 틀린 답이 나올 수 있습니다.  

* 참고로 Swift에서 소수(ex. 1.03, 0.42)를 타입을 명시하지 않고 사용하면 타입 추론으로 Double 타입이 됩니다.

## 금융 계산에 부동소수 타입을 사용한 경우

주머니에는 1달러가 있고, 선반에 10센트, 20센트, 30센트, ... 1달러짜리의 맛있는 사탕이 놓여 있다고 해봅시다. 10센트짜리부터 하니씩, 살 수 있을 때까지 사봅시다. 사탕을 몇 개나 살 수 있고, 잔돈은 얼마나 남을까요? 다음은 이 문제의 답을 구하는 '어설픈' 코드입니다.

```swift
var funds: Double = 1.00
var itemsBought: Int = 0

var price = 0.1
while funds >= price {
    funds -= price
    itemsBought += 1
    price += 0.1
}

print("\(itemsBought) 개 구입")
print("잔돈(달러): \(funds)")

// 출력 결과 
// 3 개 구입
// 잔돈(달러): 0.3999999999999999
```

프로그램을 실행해보면 사탕 3개를 구입한 후 잔돈은 0.3999999999999999 달러가 남게 됩니다. 물론 잘못된 결과입니다! 
이 문제를 올바로 해결하려면 어떻게 해야 할까요? 

**금융 계산에는**

* **자바의 경우 `BigDecimal`, 스위프트의 경우 `Decimal` 사용하기**
* **자바의 경우 `int` 혹은 `long`, 스위프트의 경우 `Int` 사용하기**

**이렇게 두 가지 방법이 있습니다.**

## 금융 계산에 Decimal 타입을 사용한 경우

코코아 Foundation 프레임워크는 10진수 관련 계산할 때 유용한 `NSDecimalNumber`(swift 로는 Decimal) 클래스를 제공합니다. 

```swift 
public init?(string: String, locale: Locale? = nil)
```

자바의 BigDecimal 처럼 스위프트도 Decimal 도 문자열을 받는 생성자가 있습니다. 문자열에 불필요한 값이 포함된다면 생성자는 nil을 반환합니다.

```swift
guard let tenCents = Decimal(string: "0.1") else { return }
guard var funds = Decimal(string: "1.00") else { return }

var itemsBought: Int = 0
var price: Decimal = tenCents
while price.isLessThanOrEqualTo(funds) {
    funds -= price
    itemsBought += 1
    price += tenCents
}

print("\(itemsBought) 개 구입")
print("잔돈(달러): \(funds)")

// 출력 결과 
// 4 개 구입
// 잔돈(달러): 0
```
=> 이 프로그램을 실행하면 사탕 4개를 구입한 후 잔돈은 0달러가 남습니다. 드디어 올바른 답이 나왔습니다.

하지만 Decimal을 이용한 십진 계산에도 장단점은 있습니다. 

* 장점: 정수와 소수를 정확하게 나타내며, 부동소수 타입보다는 느리지만 그럼에도 빠릅니다.  
* 단점: 부동 소수 타입보다는 한 자리수 이상 느립니다.

이 [글](https://forums.swift.org/t/provide-native-decimal-data-type/4003/4)을 참고했습니다. 

## 금융 계산에 정수타입을 사용한 경우

Decimal의 대안으로 Int 타입을 쓸 수 있습니다. 그럴 경우 다룰 수 있는 값의 크기가 제한되고, 소수점을 직접 관리해야 합니다.
이번 예에서는 모든 계산을 달러 대신 센트로 수행하면 이 문제가 해결됩니다. 다음은 이 방식으로 구현해본 코드입니다. 

```swift
var funds: Int = 100
var itemsBought: Int = 0
var price: Int = 10

while funds >= price {
    funds -= price
    itemsBought += 1
    price += 10
}

print("\(itemsBought) 개 구입")
print("잔돈(달러): \(funds)")

// 출력결과 
// 4 개 구입
// 잔돈(달러): 0
```

## Decimal의 반올림 모드: NSDecimalNumber.RoundingMode

Decimal도 자바의 BigDecimal처럼 반올림 모드(RoundingMode)를 제공합니다. 
자바의 BigDecimal은 반올림 모드가 8가지이지만, 스위프트의 Decimal은 4가지 모드를 제공합니다.

```swift
public enum RoundingMode : UInt {    
    case plain = 0
    case down = 1
    case up = 2
    case bankers = 3
}
```

## Decimal의 반올림 메소드

Deciamal는 또한 반올림 모드(RoundingMode)를 인수로 사용하는 반올림 메소드를 제공합니다.

> `NSDecimalNumber rounding(accordingToBehavior:) method` 을 사용하는 경우

```swift
let scale: Int16 = 3

let behavior = NSDecimalNumberHandler(roundingMode: .plain, scale: scale, raiseOnExactness: false, raiseOnOverflow: false, raiseOnUnderflow: false, raiseOnDivideByZero: true)

let roundedValue1 = NSDecimalNumber(value: 0.6844).rounding(accordingToBehavior: behavior)
let roundedValue2 = NSDecimalNumber(value: 0.6849).rounding(accordingToBehavior: behavior)

print(roundedValue1) 
print(roundedValue2) 

// 출력 결과 
// 0.684
// 0.685
```

> `NSDecimalRound(_:_:_:_:) function` 을 사용하는 경우

```swift
var value1 = Decimal(0.6844)
var value2 = Decimal(0.6849)

var roundedValue1 = Decimal()
var roundedValue2 = Decimal()

NSDecimalRound(&roundedValue1, &value1, scale, NSDecimalNumber.RoundingMode.plain)
NSDecimalRound(&roundedValue2, &value2, scale, NSDecimalNumber.RoundingMode.plain)

print(roundedValue1)
print(roundedValue2)

// 출력 결과 
// 0.684
// 0.685
```

### 참고

* [Provide native Decimal data type](https://forums.swift.org/t/provide-native-decimal-data-type/4003/1)
* [Decimal](https://developer.apple.com/documentation/foundation/decimal)
* [NSDecimalNumber](https://developer.apple.com/documentation/foundation/nsdecimalnumber)
* [NSDecimalNumber.RoundingMode](https://developer.apple.com/documentation/foundation/nsdecimalnumber/roundingmode)
* [Rounding a double value to x number of decimal places in swift
](https://stackoverflow.com/questions/27338573/rounding-a-double-value-to-x-number-of-decimal-places-in-swift)
