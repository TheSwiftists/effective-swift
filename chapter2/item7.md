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

### Swift의 Automatic Reference Counting(ARC)

: 객체의 레퍼런스 카운팅(reference counting, 참조 수)을 제공하는 메모리 관리 기법입니다.

<img src="https://developer.apple.com/library/archive/releasenotes/ObjectiveC/RN-TransitioningToARC/Art/ARC_Illustration.jpg" alt="../Art/ARC_Illustration.jpg" style="zoom:75%;" />



* **동작 방식**

  1. 레퍼런스 카운팅(reference counting)은 각 객체의 레퍼런스(= 참조)의 수를 계산하는 방식으로 동작합니다.
  2. 레퍼런스 카운트가 0이 되면 그 객체는 확실히 접근할 수 없습니다(unreachable).
  3. **컴파일** 타임 동안, 런타임에 레퍼런스 카운트를 증가시키거나 감소시키는 retain이나 release와 같은 메시지를 삽입하고, 객체의 레퍼런스 카운트가 0에 도달했을 때 객체의 할당 해제를 표시합니다.
  4. 컴파일러는 일정한 규칙에 의해 메모리 참조 및 해제 코드를 생성합니다. 개발자는 ARC가 언제 참조 카운트를 올리고 내리는지 규칙을 알고, 적절한 시점에 코드가 생성되도록 조절할 수 있습니다.
  5. GC와 달리 백그라운드에서 이뤄지는 과정이 아니며 **런타임에 비동기적**으로 객체를 제거합니다. 

* **레퍼런스 카운트가 오르는 경우**
  레퍼런스 카운트는 클래스 인스턴스(Class instance) 등의 레퍼런스 타입 값을 변수에 할당할 때 증가합니다. 클래스 인스턴스를 변수에 할당할 때 메모리에서는 heap 영역에 저장된 instance meta data의 주소값을 변수에 할당하는 작업이 일어납니다. 즉, 메모리의 **heap 영역에 저장되어 있는 instance meta data의 주소값이 변수에 할당되는 시점이 참조 카운트가 증가하는 시점**입니다.

  <img src="https://images.velog.io/images/cskim/post/8572c476-6691-4f93-b72e-4f43cce35dac/image.png" alt="img" style="zoom: 25%;" />

* **레퍼런스 카운트가 내려가는 경우**
  레퍼런스 카운트가 감소하는 시점은 **인스턴스를 참조하던 stack 영역의 변수들이 메모리에서 해제될 때** 입니다.

  <img src="https://images.velog.io/images/cskim/post/cf077539-43c2-4c24-b22f-5c03ada0874c/image.png" alt="img" style="zoom:25%;" />

* **장단점**

  * **Automatic Reference Counting의 장점**
    1. 실시간으로, 객체가 사용되지 않을 때 결정론적 파괴([deterministic destruction](https://docs.elementscompiler.com/Concepts/ARCvsGC/))가 일어납니다.
       참고로, [결정론적 알고리즘(*deterministic* algorithm](https://ko.wikipedia.org/wiki/%EA%B2%B0%EC%A0%95%EB%A1%A0%EC%A0%81_%EC%95%8C%EA%B3%A0%EB%A6%AC%EC%A6%98))은 '예측한 그대로 동작하는' 알고리즘을 뜻하는데 이곳에서도 이와 같은 맥락으로 **예측 그대로 파괴된다**는 의미라고 해석할 수 있을 것 같습니다.
    2. 백그라운드 프로세싱이 없으므로 모바일 장치와 같은 저전력 시스템에서 더 효율적입니다.
  * **Automatic Reference Counting의 단점**
    1. retain cycles에 대처할 수 없습니다.

