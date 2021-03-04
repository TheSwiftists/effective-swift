# Item.60 정확한 답이 필요하다면 float과 double은 피하라

## 부동소수 타입 float, double

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

## 금융 계산에 부동소수 타입을 사용한 경우

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

## 금융 계산에 Decimal 타입을 사용한 경우

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

## 금융 계산에 정수타입을 사용한 경우

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

## Decimal의 반올림 모드 

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