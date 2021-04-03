# Item 82. 스레드 안정성 수준 문서화

책에 소개된 내용과 iOS 관련 내용을 합하여 작성하였습니다.

### 📚 스레드 안전성 문서화

어떤 순서로 호출할 때 외부 동기화가 필요한지, 그리고 호출 순서와 관련 lock에 대한 내용을 명시해야합니다. 또한 클래스의 스레드 안전성은 보통 클래스의 문서화 주석에 기재하지만, 독특한 특성의 메서드라면 해당 메서드의 주석에 기재하도록 하는 것을 책에서 권장하고 있습니다.

* 스레드 안정성 수준
  * 불변 immutable
  * 무조건적 스레드 안전 unconditionally thread-safe
  * 조건부 스레드 안전 conditionally thread-safe
  * 스레드 안전하지 않음 not thread-safe
  * 스레드 적대적 thread-hostile
  
  

### iOS 에서 'Thread - Safe' 의 의미

> Thread safe code can be **safely called from multiple threads or concurrent tasks without causing any problems** such as data corruption or app crashes. Code that is not thread safe can only run in one context at a time.
> 출처: [raywenderlich - Grand Central Dispatch Tutorial for Swift 4: Part 1/2](https://www.raywenderlich.com/5370-grand-central-dispatch-tutorial-for-swift-4-part-1-2#toc-anchor-009)

1. https://medium.com/@snobinsights/multithreading-in-ios-with-gcd-sync-and-async-operations-made-easy-9af0fe06c7b3)