# Item 82. 스레드 안정성 수준 문서화하라

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



### iOS에서의 Multi Thread Programming

GDC( Grand Central Dispatch )는 동시성 프로그래밍을 더 쉽고 간편하게 하기 위해 iOS에서 제공하는 도구로 Thread를 직접적으로 사용하는 것보다 GCD 사용을 권장하고 있습니다. 

> In the past, if an asynchronous function did not exist for what you want to do, you would have to write your own asynchronous function and create your own threads. But now, OS X and iOS provide technologies to allow you to perform any task asynchronously **without having to manage the threads yourself.**
>
> One of the technologies for starting tasks asynchronously is ***Grand Central Dispatch (GCD)***. This technology **takes the thread management code** you would normally write in your own applications and moves that code down to the system level. All you have to do is define the tasks you want to execute and add them to an appropriate dispatch queue. GCD takes care of creating the needed threads and of scheduling your tasks to run on those threads. **Because the thread management is now part of the system, GCD provides a holistic approach to task management and execution, providing better efficiency than traditional threads.**
>
> 출처: [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html#//apple_ref/doc/uid/TP40008091-CH100-SW8)

* **단, 아래와 같은 경우는 Threads를 사용하는 것이 좋다고 명시하고 있습니다**

  > ### When to Use Threads
  >
  > ...
  >
  > **Threads are still a good way to implement code that must run in real time.** (쓰레드는 실시간으로 실행해야하는 코드를 구현하는 좋은 방법입니다.) Dispatch queues make every attempt to run their tasks as fast as possible but they do not address real time constraints. **If you need more predictable behavior from code running in the background, threads may still offer a better alternative.** (백그라운드에서 실행되는 코드에서보다 예측가능한 동작이 필요한 경우 쓰레드가 여전히 더 나은 대안을 제공할 수 있습니다)
  >
  > As with any threaded programming, you should always use threads judiciously and only when absolutely necessary. For more information about thread packages and how you use them, see *[Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)*.
  >
  > **스레드는 실시간으로 실행해야 하는 코드를 구현하는 좋은 방법입니다.** **디스패치 큐는 작업을 최대한 빨리 실행하려고 하지만 실시간 제약 조건을 해결하지는 못합니다. 백그라운드에서 실행되는 코드에서 더 예측 가능한 동작이 필요한 경우에도 스레드가 더 나은 대안을 제공할 수 있습니다.**
  >
  > 모든 스레드 프로그래밍과 마찬가지로 항상 신중하게 스레드를 사용해야 하며 반드시 필요할 때만 스레드를 사용해야 합니다. 스레드 패키지와 이 패키지의 사용 방법에 대한 자세한 내용은  *[Threading Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/Introduction/Introduction.html#//apple_ref/doc/uid/10000057i)*를 참조하십시오.
  >
  >Thread-Safe Designs에 대해 더 알고 싶으면 [Tips for Thread-Safe Designs](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/ThreadSafety/ThreadSafety.html#//apple_ref/doc/uid/10000057i-CH8-SW6)를 읽어보시면 좋습니다.
  
  

### iOS에서의 Multi Thread Programming 체크 리스트 ✅

 - [ ] ✅
     프로세스간 '공유하는 자원'을 서로 다른 '여러 스레드'에서 '동시에' 접근하는 경우가 없는지 확인합니다.
     서로 다른 스레드가 데이터와 Heap 영역을 공유하고 있기 때문에 한 스레드에서 사용 중인 공유 자원에 접근하여 엉뚱한 값을 읽어올 수 있습니다. 이와 관련된 문제를 Readers-Writers problem 라고 합니다.
     
     > The readers-writers problem is a concurrency problem faced when **a shared resource that is not thread-safe can be read and written by multiple threads at the same time.** **The problem happens when one thread is reading or writing and another thread writes at the same time.** In contrast, when the resource is only being read by more than one thread the problem doesn't appear since the resource is up to date and it won't provide outdated/wrong data.
     > 출처: [The Readers-Writers Problem (Swift Edition)](https://medium.com/swlh/the-readers-writers-problem-swift-edition-dcba94c3f02d#:~:text=The%20readers%2Dwriters%20problem%20is,writes%20at%20the%20same%20time.)

- [ ] ✅ 
  위와 같은 Readers-Writers problem을 방지하기 위해 그리고

  Thread-Safe하게 만들기 위해서는 Immutable 한 변수를 고려합니다. Immutable한 인스턴스는 서로 다른 여러 스레드에서 한 번에 접근해도 문제가 되지 않습니다. 즉, Thread-Safe 합니다. 
  -> Structure 사용을 우선하여 고려하면 좋습니다. (이와 관련하여 클래스와 구조체에 다룬 내용은 [iOS Developer - Value and Reference Types](https://developer.apple.com/swift/blog/?id=10) 를 참고해주세요)
  반면 Mutable한 인스턴스는 서로 다른 스레드에서 동시에 변경이 이뤄진다면 문제가 발생합니다. 때문에 Immutable 한 변수를 고려해보는 것을 권장합니다. 
  (단, Mutable한 인스턴스를 사용해야할 경우, 읽기 전용으로 만든다면 문제가 되지 않게 할 수 있습니다.)
  
  
  
 - [ ] ✅ 
    스레드 관련 Property Attirbute 는 atomic, nonatomic 이 있는데 atomic 키워드는 해당 프로퍼티가 동시에 접근 할 수 없게하므로 mutable 멤버 변수는 atomic 으로 하는 것을 권장합니다.
(단, atomic 은 속도면에서 불리함 이있으니 mutable 인스턴스라도 변경 중에 동시 접근 할 일이 없다면 nonatomic 으로 해도 될 것입니다.)
    
    
    Swift 언어의 anomic에 대한 내용은 [atomic/non atomic](https://hcn1519.github.io/articles/2019-03/atomic) 을 참고하면 좋습니다.
    
    > - atomic - 중단되지 않는
    >   : an operation appears to occur at a single instant between its invocation and its response
    >
    >   `atomic`하다는 것은 프로그래밍에서 **데이터의 변경이 한 번에 일어난 것처럼 보이게 하는 것**을 의미합니다. 데이터의 값을 변경하는 작업에는 항상 값 변경의 시간이 필요합니다. 그런데 `atomic`한 데이터는 데이터의 값에 접근하는 여러 데이터 소비자(프로세스, 쓰레드 등)의 관점에서 데이터 값 변경에 걸리는 시간이 0초인 것처럼 느끼게 합니다.
    >
    >   --------------------------------
    >
    >   한편, Swift는 Thread-Safe를 고려하고 디자인된 언어가 아니기 때문에 모든 property는 `non atomic`입니다. 그리고 별도로 `atomic` 옵션을 지정할 수도 없습니다. 그래서 Swift의 property가 `atomic`을 지원하기 위해서는 GCD를 통해 이를 구현해주어야 합니다. 다음 [글](https://www.objc.io/blog/2018/12/18/atomic-variables/)에서 자세한 내용을 확인할 수 있습니다.
    
- [ ] ✅ 
  함수를 실행 할 때 특정 구간에서 특정 자원을 동시에 접근 하지 못하게 하기 위해 Lock을 사용한 경우 Deadlock을 확인하는 것이 좋습니다.
  이 경우 한 스레드에서 그 구간을 끝낼 때 까지 다른 스레드에서 접근 할 수 없게 될 수 있으므로 Deadlock 이 발생할 수 있습니다.

- [ ] ✅ 
  UI 업데이트 관련 작업들을 메인 스레드에서 구현하고 있는지 확인해야 합니다.

- [ ] ✅ viewDidLoad() 나 UI스레드로부터 직접적으로(directly) 실행되는 어떤 코드 안에  DispatchQueue.main.sync { }  코드가 있는지 확인합니다. 또한 중첩된 Sync block 이 있는지 확인합니다.
  중첩된 *sync* block은 deadlock과 코드 크러쉬를 발생시키는데, UI에 대한 일을 처리하는 메인큐가 **직렬(Serial)** 이므로 Deadlock이 발생하기 때문에 이런 코드가 있다면 당장 수정해야 합니다.
  
  > ```swift
  > var concurrentQueue = DispatchQueue(label: "com.snobinsights.ConcurrentQueue", attributes: .concurrent)
  > concurrentQueue.sync {
  >     concurrentQueue.sync {
  >         for _ in 1...5 { print("🔴") }
  >     }
  > }
  > ```
  >
  > As the queue is serial, means it can **only execute one task at a time**. So when a sync function is used inside sync(or even async) function on a serial queue and the inner sync uses the same serial queue for executing its task. Then the **inner sync block won’t be able to start its execution and won’t return the control to the caller (serial queue in this case) as that serial queue is already busy executing tasks assigned to it.** **Hence, they both keep waiting for the completion of each other, and a deadlock is created.**
  > 출처: [Multithreading in iOS with GCD: sync and async operations made easy](https://medium.com/@snobinsights/multithreading-in-ios-with-gcd-sync-and-async-operations-made-easy-9af0fe06c7b3)


### 참고

1. [Concurrency Programming Guide](https://developer.apple.com/library/archive/documentation/General/Conceptual/ConcurrencyProgrammingGuide/ConcurrencyandApplicationDesign/ConcurrencyandApplicationDesign.html#//apple_ref/doc/uid/TP40008091-CH100-SW8)
2. [Thread Programming Guide](https://developer.apple.com/library/archive/documentation/Cocoa/Conceptual/Multithreading/ThreadSafety/ThreadSafety.html#//apple_ref/doc/uid/10000057i-CH8-SW1)
3. [atomic/non atomic](https://hcn1519.github.io/articles/2019-03/atomic)
4. [Swift Tip: Atomic Variables](https://www.objc.io/blog/2018/12/18/atomic-variables/)
5. [iOS Developer - Value and Reference Types](https://developer.apple.com/swift/blog/?id=10)
6. [The Readers-Writers Problem (Swift Edition)](https://medium.com/swlh/the-readers-writers-problem-swift-edition-dcba94c3f02d#:~:text=The%20readers%2Dwriters%20problem%20is,writes%20at%20the%20same%20time.)
7. [iOS ) Concurrency Programming Guide - Concurrency and Application Design](https://zeddios.tistory.com/509)
8. [Swift의 메모리 안정성](https://jcsoohwancho.github.io/2019-08-25-Swift%EC%9D%98-%EB%A9%94%EB%AA%A8%EB%A6%AC-%EC%95%88%EC%A0%95%EC%84%B1/)
9. [[iOS] 멀티 스레드(Multi Thread) 구현 시 고려해야될 것들](https://gwangyonglee.tistory.com/47)
10. [Multithreading in iOS with GCD: sync and async operations made easy](https://medium.com/@snobinsights/multithreading-in-ios-with-gcd-sync-and-async-operations-made-easy-9af0fe06c7b3)