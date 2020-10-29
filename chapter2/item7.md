# Item 7. 다 쓴 객체 참조를 해제하라

### Java의 Garbage Collection(GC)

<img src="https://t1.daumcdn.net/cfile/tistory/275F4333578E05A021" alt="img" style="zoom:75%;" />



* **동작 방식**
  1. **런타임**시 **백그라운드**에서 사용되지 않는 객체와 객체 그래프들(objects and object graphs)을 감지하는 방식으로 동작합니다.
  2. 1번 동작은 일정 시간 경과한 뒤나 런타임 메모리가 낮아졌을 때 중간 간격(intermediate intervals)으로 발생하며 정확한 순간에 해제되지 않습니다.
  3. Java의 경우 JVM에 의해 힙(heap) 영역에 생성 객체가 할당됩니다.
  4. JVM은 객체들이 더이상 코드에 참조되지 않을 때를 추적합니다.
  5. GC는 Unrechable Objects(Root Set of References(유효한 최초의 참조)가 이루어지지 않는 객체)들을 수거합니다.
  6. GC는 객체를 메모리에서 제거하기 전에 해당 객체의 finalize( ) 메소드 호출합니다.
  7. GC는 사용자가 강제로 수행할 수 없고 언제 일어나는지도 불확실합니다.

* **일반적인 GC의 대상**
  a. 모든 객체 참조가 null 인 경우

  b. 객체가 블럭 안에서 생성되고 블럭이 종료된 경우

  c. 부모 객체가 null이 된 경우, 자식 객체는 자동적으로 GC 대상

  d. 객체가 Weak 참조만 가지고 있을 경우

  e. 객체가 Soft 참조이지만 메모리 부족이 발생한 경우

* **장단점**
  * **Garbage Collection의 장점**
    1. retain cycles을 포함한 전체 객체 그래프(object graphs)를 정리(clean up)할 수 있습니다.
    2. GC는 백그라운드에서 발생(happen)하므로 정기적인 애플리케이션 흐름의 일부로서 메모리 관리 작업이 덜 수행됩니다.
  * **Garbage Collection의 단점**
    1. GC는 백그라운드에서 발생하기 때문에, 객체 릴리즈(objects release)에 대한 정확한 시간이 결정되지 않습니다(undetermined).
    2. GC가 발생하면 애플리케이션의 다른 스레드가 일시적으로 보류될 수 있다(put on hold).

