# item 63. 문자열 연결은 느리니 주의하라

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단입니다. 그런데 한 줄짜리 출력값이나 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괜찮지만, 많은 양의 반복적인 사용을 하는 경우 성능 저하를 감내하기 어렵습니다. 그러므로 effection java 저자 조슈아 블로크는 성능에 신경을 써야 한다면 많은 문자열 연결 시 문자열 연결 연산자(+)를 피하라고 말하고 있습니다. Swift에서도 동일하게 적용되는 제안인지 확인해보았습니다.

Swift로 문자열 연결을 할 수 있는 방법에는 몇 가지가 있습니다. 그 중 가장 많이 사용되는 방법들을 골라 아래와 같이 일정 갯수만큼 반복하게 해 걸리는 시간을 측정해보았습니다.

두가지 방법으로 테스트를 해보았는데, 첫번째는 아래처럼 `CFAbsoluteTimeGetCurrent` 를 이용해 시간을 측정했고, 두번째는 하단 이미지처럼 테스트 코드 내 `measure` 메서드를 이용했습니다.

```swift
let lineForItem = "12345678901234567890123456789012345678901234567890123456789012345678901234567890"
let number = 1000

func bench(function: () -> Void, number: Int) {
  let startTime = CFAbsoluteTimeGetCurrent()
  function()
  let processTime = CFAbsoluteTimeGetCurrent() - startTime
  print("process time\(number) = \(processTime)")
}

func test1() {
  var result = ""
  for _ in 1...number {
    result += lineForItem
  }
}

func test2() {
  var result = ""
  for _ in 1...number {
    result.append(lineForItem)
  }
}

func test3() {
  var result = ""
  for _ in 1...number {
    result = "\(result)\(lineForItem)"
  }
}
```

<img src="https://user-images.githubusercontent.com/40784518/110232054-bc6f5b80-7f5e-11eb-8d9e-1faf66c52ea5.png"/>

두가지 방법으로 테스트 해 본 결과 각각 `+ 연산자`, `string interpolation`, `append()` 방법으로 빨랐습니다. 하지만 `test2()` 에서 `append()` 메서드를 이용한 경우 오차범위가 ±185%이어서 신뢰할만한 측정값은 아니라는 결론을 내렸습니다. 

하지만 책에서 말하는 것처럼 많은 양의 문자열을 합칠 때 + 연산자를 지양하지는 않아도 될 것 같습니다.

