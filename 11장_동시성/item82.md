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
  
  
