# Item 8. finalizer와 cleaner 사용을 피하라



Java의 객체 소멸자에 대해서 간단하게 소개한 후 Swift의 디이니셜라이저를 활용한 예제를 소개합니다.

### Java의 객체 소멸자

[finalize()](https://docs.oracle.com/javase/9/docs/api/java/lang/Object.html#finalize--) 메서드는 클래스의 인스턴스가 더이상 참조되지 않을 때 가비지 컬렉터(Garbage Collector)가 힙에서 객체를 제거하기 전에 자동으로 호출 됩니다.

```java
@Override
public void finalize() {
    try {
        reader.close();
        System.out.println("Closed BufferedReader in the finalizer");
    } catch (IOException e) {
        // ...
    }
}
```

[Cleaner](https://docs.oracle.com/javase/9/docs/api/java/lang/ref/Cleaner.html) 는 해당 객체가 [phantom reachable](https://docs.oracle.com/javase/8/docs/api/java/lang/ref/PhantomReference.html)이 되었을 때를 통지 받은 후 실행되도록 Cleaning actions을 등록합니다. GC에 의해 수거될 객체들은 `register()` 메서드를 사용하여서 cleaner 객체에 등록되어야 합니다.

```java
 public class CleaningExample implements AutoCloseable {
        // A cleaner, preferably one shared within a library
        private static final Cleaner cleaner = <cleaner>;

        static class State implements Runnable {

            State(...) {
                // initialize State needed for cleaning action
            }

            public void run() {
                // cleanup action accessing State, executed at most once
            }
        }

        private final State;
        private final Cleaner.Cleanable cleanable

        public CleaningExample() {
            this.state = new State(...);
            this.cleanable = cleaner.register(this, state); // 등록
        }

        public void close() {
            cleanable.clean();
        }
    }
```



### Java의 finalizer와 cleaner 특징

Java의 객체 소멸자인 finalizer와 cleaner는 다음과 같은 특징을 가지고 있습니다.

* finalizer는 예측할 수 없고, 상황에 따라 위험할 수 있어 일반적으로 불필요해 기본적으로 사용하지 않는 것을 권합니다.
* cleaner는 finalizer보다 덜 위험하지만 finalizer와 마찬가지로 예측할 수 없고 느리며 일반적으로 불필요합니다.
* finalizer와 cleaner가 얼마나 신속히 수행할지는 [가비지 컬렉터(CG)](chapter2/item7.md) 알고리즘에 달려있으며 가비지 컬렉터 구현마다 상이합니다.
* Java 언어 명세는 finalizer와 cleaner의 수행 시점 뿐만 아니라 수행 여부도 보장하지 않습니다. 
  * Finalizer 스레드는 다른 애플리케이션 스레드보다 우선순위가 낮아 실행될 기회를 얻지 못할 수도 있습니다. 자바 언어 명세는 어떤 스레드가 finalizer를 수행할지 명시하지 않습니다.



### Swift에서는 

Swift에서는 클래스의 인스턴스 레퍼런스 카운트가 0이 되면 메모리에서 할당 해제합니다. 그리고 클래스의 인스턴스가 소멸되기 직전에 [deinit](https://docs.swift.org/swift-book/LanguageGuide/Deinitialization.html)이 호출됩니다. 즉, Java에서와 달리 개발자가 인스턴스의 소멸 시점과 deinit이 불릴 시점을 예측할 수 있습니다.

그렇다면 deinit에서는 어떤 일을 할 수 있을까요?

책 본문에 나온 C++의 내용과 같이  Swift도 인스턴스가 소멸될 때 deinit을 통해 비메모리 자원 회수 용도(reclaim other [nonmemory resources](https://stackoverflow.com/a/7037712))로 사용할 수 있습니다.

> * 비메모리 자원(nonmemory resources)
>   : 메모리의 일부를 차지하면서 다른 리소스 일부에도 접근할 수 있는 권한이 있는 데이터베이스, 네트워크, 파일 등 

`deinit` 때 처리해줄 일의 예시()로는 [item9](chapter2/item9.md)(NotificationCenter, FileHandle, DBConnection(SQLite))에서 설명하고 있습니다. 이번 아이템에서는 추가적인 예시로 RxSwift의 `Dispose()` 메서드와 `DisposeBag`에 대해서 설명하겠습니다. 아래에 예시는 다양한 구현방법 중 하나입니다.

### RxSwift

`Dispose()` 메서드와 `DisposeBag`을 소개하기 앞서 두 가지 필요한 내용을 정리하겠습니다.

> Obsevable: 변화의 알림을 보냅니다.
>
> Observer: Observable을 구독하고 Observable이 변화되었을 때 알림을 받습니다.
> 
> 
> 
> 

### RxSwift의 `Dispose()`

RxSwift 에서 [Observable](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Observable.swift) 을 subscribe(구독) 하면 항상 [Disposable](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposable.swift) 을 반환합니다. 이 Disposable 들을 dispose 해주지 않으면 메모리에 계속 남아 메모리 누수가 발생합니다. 따라서 구독한 Disposable 들을 명시적으로 dispose 해줘야합니다. 

```swift
final class MyViewController: UIViewController {
    var subscription: Disposable?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        subscription = theObservable().subscribe(onNext: {
            // handle your subscription
        })
    }
    
    deinit {
        subscription?.dispose()
    }
}
```



**그렇다면 일일이 Dispoable 프로토콜에 구현되어있는 dispose() 를 호출하여 없애줘야 할까요?** [RxSwift 가이드 문서의 Disposing](https://github.com/ReactiveX/RxSwift/blob/master/Documentation/GettingStarted.md#disposing) 에서 권장하는 방법 중 하나는 `dispose()` 를 직접 호출하지 말고 `DisposeBag`을 사용하는 것입니다. `DisposeBag` 이 할당 해제될 때, 각 dispoable에 `dispose` 메서드가 호출됩니다. 

### RxSwift의 `DisposeBag`

> DisposeBag은 메모리 관리와 ARC 관리를 위해 RxSwift에서 제공하는 툴로, 부모 객체의 상위 객체를 할당 해제하면 DeleteBag에서 Observer 객체가 폐기됩니다. 
> DisposeBag을 가지고 있는 객체의 deinit이 호출될 때, 각 disposable Observer는 관찰하고 있던 것에서 자동으로 구독해지됩니다. DisposeBag을 사용하지 않으면 Observer는 retain cycle을 만들 수 있습니다. (무기한으로 옵저빙에 매달리거나, 할당을 취소하여 크러쉬를 유발할 수 있습니다.)

[DisposeBag](https://github.com/ReactiveX/RxSwift/blob/master/RxSwift/Disposables/DisposeBag.swift) 의 내부구현 중 일부를 살펴보면

```swift
public final class DisposeBag: DisposeBase {
  
  private var disposables = [Disposable]()
  
    private func dispose() {
    let oldDisposables = self._dispose()

    for disposable in oldDisposables {
        disposable.dispose()
    }
  }
  
  private func _dispose() -> [Disposable] {
    self.lock.performLocked {
        let disposables = self.disposables
            
        self.disposables.removeAll(keepingCapacity: false)
        self.isDisposed = true
            
        return disposables
        }
    }

  deinit {
    self.dispose()
  }
}
```

이렇게 `DisposeBag`의 `Disposable` 타입의 배열에 담긴 `Disposable` 들을 `dispose` 하는 메서드가 구현되어 있습니다. 그리고 `DisposeBag`이 할당 해제될 때(`deinit` 에서) `dispose()` 메서드를 호출해 `DisposeBag`이 담고있는 `Disposable`들을 `dispose`해 각 disposable Observer는 관찰하고 있던 것에서 자동으로 구독해지시킵니다.

아래 예제 코드는 `dispose()` 메서드를 사용한 예제를 `DisposeBag`을 사용하는 방법으로 바꾼 것입니다.

```swift
final class MyViewController: UIViewController {
    let disposeBag = DisposeBag()

    override func viewDidLoad() {
        super.viewDidLoad()
      
      let parsedObject = theObservable
        .map { [parser] json in
        return parser.parse(json)
       }
        parsedObject.subscribe(onNext: {
          // handle your subscription
        })
        .disposed(by: disposeBag)
    }
}
```

