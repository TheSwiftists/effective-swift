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

