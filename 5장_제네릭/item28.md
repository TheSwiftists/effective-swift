# item28. 배열보다는 리스트를 사용하라

> Swift에는 Array만 있는데요?



## Java

배열과 제네릭 타입에는 중요한 차이가 두 가지 있습니다. 첫 번째, 배열은 공변(covariant)입니다. 어려워 보이는 단어지만 뜻은 간단한데요, Sub가 Super의 하위 타입이라면 배열 `Sub[]`는 배열 `Super[]`의 하위 타입이 됩니다.(공변, 즉 함께 변한다는 뜻) 반면, 제네릭은 불공변(invariant)입니다. 즉, 서로 다른 Type1과 Type2가 있을 때, List<Type1>은 List<Type2>의 하위 타입도 아니고 상위 타입도 아닙니다. 이것만 보면 제네릭에 문제가 있다고 생각할 수도 있지만, 사실 문제가 있는 건 배열 쪽입니다. 다음은 문법상 허용되는 코드입니다.

```java
//런타임에 실패하는 코드
Object[] objectArray = new Long[1];
objectArray[0] = "타입이 달라 넣을 수 없다";	//ArrayStoreExtension을 던짐
```

하지만 다음 코드는 문법에 맞지 않습니다.

```java
//컴파일이 되지 않는 코드
List<Object> objectList = new ArrayList<Long>();	//호환되지 않는 타입
objectList.add("타입이 달라 넣을 수 없다");
```

어느 쪽이든 Long용 저장소에 String을 넣을 수 없습니다. **다만 배열에서는 그 실수를 런타임에야 알게 되지만, 리스트를 사용하면 컴파일할 때 바로 알 수 있습니다.**

두 번째 주요 차이로는 배열은 실체화(reify)됩니다. **배열은 런타임에도 자신이 담기로 한 원소의 타입을 인지하고 확인합니다.** 그래서 위 코드에서 보듯 Long 배열에 String을 넣으려 하면 `ArrayStoreException`이 발생합니다. 반면, 앞서 이야기했듯 **제네릭은 타입 정보가 런타임에는 소거(erasure)됩니다.** 원소 타입을 컴파일타임에만 검사하며 런타임에는 알 수조차 없다는 뜻입니다. 소거는 제네릭이 지원되기 전의 레거시 코드와 제네릭 타입을 함께 사용할 수 있게 해주는 메커니즘입니다.

위의 차이들로 배열과 제네릭은 잘 어우러지지 못합니다. 예컨대 배열은 제네릭 타입, 매개변수화 타입, 타입 매개변수로 사용할 수 없습니다. 즉 코드를 new List\<E>[], new List\<String>[], new E[] 식으로 작성하면 컴파일할 때 제네릭 배열 생성 오류를 일으킵니다. 

제네릭 배열을 만들지 못하게 막은 이유는 무엇일까요? **타입 안전하지 않기 때문**입니다. 이를 허용한다면 컴파일러가 자동 생성한 형변환 코드에서 런타임에 `ClassCastException`이 발생할 수 있습니다. 런타임에 `ClassCastException`이 발생하는 일을 막아주겠다는 제네릭 타입 시스템의 취지에 어긋나는 것입니다.



<br>



## Swift

Array와 ArrayList가 별개로 있는 Java에 반해, Swift는 Array 뿐입니다.  Java의 Array와 ArrayList의 차이는 다음과 같습니다.

```Java
//Array vs ArrayList - Java
  
1. Array는 크기가 고정되어 있지만, ArrayList는 사이즈가 동적인 배열입니다.
2. Array는 Primitive Type(int, byte, char 등)과 Object 모두를 담을 수 있지만,
   ArrayList는 Object Element만 담을 수 있습니다.
3. Array는 제네릭을 사용할 수 없지만, ArrayList는 타입 안정성을 보장해주는 제네릭을 사용할 수 있습니다.
```



Swift의 Array는 Java의 그것과는 다릅니다. 동적 할당 기능을 가지고 있고, 정수부터 문자열, 클래스에 이르기까지 모든 데이터 타입을 저장할 수 있습니다. 주된 차이점 중 하나인 동적 할당 기능에 대해 조금 더 자세히 알아봅시다.

 

### Array의 크기 늘리기

Array가 내부적으로 작동하는 방식에 대해 알아보기 전에, Array의 속성인 `count`와 `capacity`에 대해 알아봅시다. `count`는 배열에 있는 요소의 수를 나타냅니다. `capacity`는 새 메모리를 초과하여 할당하기 전에, 배열에 포함될 수 있는 요소의 총량을 나타냅니다.

모든 Array는 내용을 보관하기 위해 특정 양의 메모리를 예약합니다. 배열에 요소를 추가하고, 해당 배열이 예약 된 `capacity`를 초과하기 시작하면 배열은 더 큰 메모리 영역을 할당하고 해당 요소를 새 스토리지에 복사합니다. 

**새 저장소는 이전 저장소 크기의 배수**입니다. 이 기하 급수적 성장 전략은 요소 추가가 일정한 시간에 발생하여 많은 추가 작업의 성능을 평균화한다는 것을 의미합니다. 재할당을 발생시키는 추가 작업에는 성능 비용이 발생하지만 Array가 커질수록 발생 빈도가 점점 줄어 듭니다.

<img width="90%" src="https://user-images.githubusercontent.com/40784518/103150205-3217b800-47b5-11eb-9951-f6a39da95fa6.png">

위의 그림을 봅시다. 용량을 꽉 찬 배열에 "Peach" 를 추가하면 항목이 즉시 추가되지는 않습니다. 다른 곳에 새 메모리가 생성되고 모든 항목이 복사된 후 마지막으로 "Peach"가 배열에 추가됩니다. 메모리의 다른 영역에 새 스토리지를 할당하는 이 방법을  **재할당(Reallocation)** 이라고 합니다. 

저장해야하는 요소 수를 대략적으로 알고있는 경우  `reserveCapacity(_:)` 메서드를 사용하면 중간에 일어나는 재할당을 방지할 수 있습니다. 하지만 예약된 용량을 초과하도록 요소가 추가된다면 다시 재할당이 일어나게 됩니다. 



<br>



### 참고

- [Array | Apple Developer Documentation](https://developer.apple.com/documentation/swift/array)


- [Array vs ArrayList](https://zorba91.tistory.com/287)
