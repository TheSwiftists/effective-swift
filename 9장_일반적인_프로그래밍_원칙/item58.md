# item58. 전통적인 for문보다는 for-each문을 사용하라

Array나 Set과 같은 컬렉션을 순회 할 때, 전통적인 for-loop을 사용할지, for-each문을 사용할지 고민됩니다. 두 방법 모두 순차적으로 element에 접근이 가능합니다. 메커니즘이 매우 유사해 선호도나 스타일에 따라 달리 사용하기도 하지만 몇가지 뚜렷한 면에서 차이가 있습니다. 이번 아이템에선 그 차이점들에 대해 알아보고 어떨 때 어떤 것을 사용하면 좋을 지 알아봅니다.



### for-loop



### forEach



차이점 중 하나는 for-in문의 반복은 코드 제어 흐름 내에서 직접 수행되어 이러한 반복을 훨씬 더 정확하게 제어 할 수 있다는 것입니다. 예를 들어 continue 키워드를 이용해서 다음 요소로 넘어갈지 혹은 break 키워드를 이용해서 반복을 멈출지 결정할 수 있습니다.

```swift
func categorizeDogs(among dogs: [Dog]) -> [Dog] {
  var result = [Dog]()
  
  fo
}
```



For-in문에서 guard나 if의 조건문을 사용해 필터링의 작업을 하는 경우 where 조건을 추가할 수도 있습니다.

```swift
for dog in dogs where dog.kind == .jino {
  results.append(dog)
  guard results.count < 5 else { break }
}
```

https://kimlh2.tistory.com/entry/Swift-for-in-foreach-의-차이점

https://www.swiftbysundell.com/tips/picking-between-for-and-for-each/