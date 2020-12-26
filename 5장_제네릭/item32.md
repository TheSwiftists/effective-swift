# item 32. 제네릭과 가변인수(varargs)를 함께 쓸 때는 신중하라  

얖서 item28 에서는 제네릭과 배열은 함께 사용할 수 없다는 것(컴파일 오류)을 배웠습니다.
하지만 이번 장에서는 **함수의 매개변수**로서 제네릭과 가변인수(= 배열)은 같이 사용할 수 있고, 사용하는 경우 조심해야 하는 점을 말합니다.

이 글에서는
* 먼저 item28의 내용인 제네릭과 배열을 자바에서 왜 함께 사용 못하는지 되짚어보고, 
* 자바에서 제네릭과 가변인수를 함께 사용하는 것을 왜 허용하는지, 함께 쓸 때 조심해야 하는 것은 무엇인지 살펴봅니다. 
* 그리고 스위프트에서는 이 문제가 어떻게 해결되는지 살펴봅니다. 

# Java

## 제네릭과 배열은 함께 사용할 수 없다(item28)

* 제네릭과 배열을 함께 사용하면 불공변(invariant)인 제네릭의 특징이 배열로 인해 덮어지고, 타입이 불안정해져서 런타임 에러 가능성이 생깁니다.

> 코드 28-3 제네릭 배열 생성을 허용하지 않는 이유 - 컴파일 되지 않는다.  
```java
List<String>[] stringLists = new List<String>[1]; // (1)
List<Integer> intList = List.of(42);              // (2)
Object[] objects = stringLists;                   // (3)
objects[0] = intList;                             // (4)
String s = stringLists[0].get(0);                 // (5) ClassCastException
```

=> (1) 에서 컴파일이 된다고 가정해보면 결국 (5) 에서 런타임 오류인 `ClassCastException` 이 발생합니다. 
따라서 `ClassCastException` 과 같은 (타입 불안정으로 인한) 런타임 오류가 발생하는 것을 방지하겠다는 제네릭 타입 시스템과 취지가 맞지 않으므로 자바에서는 배열과 제네릭을 같이 사용하는 것을 금지합니다. 

> Note
<br>(1) 배열: 공변
<br>=> Sub가 Super의 하위 타입이라면, Sub[]도 Super[]의 하위 타입이 된다.
<br>(2) 제네릭: 불공변 
<br>=> Sub가 Super의 하위 타입이어도, List\<Sub>는 List\<Super>의 하위 타입이 아니고 
List\<Super>도 List\<Sub>의 상위 타입이 아니다.

## 제네릭과 varargs를 함께 사용할 수 있지만, 역시 타입 불안정하다.

* 제네릭과 varargs를 함께 사용 가능하지만, 타입 불안정해서 런타임 에러 가능성이 있고 따라서 힙 오염(heap pollution) 경고가 발생합니다.  
  * 힙 오염 경고는 힙 메모리에 잘못된 데이터의 존재 가능성을 경고합니다. Java 언어에서 힙 오염은 매개변수화 된 유형의 변수가 해당 매개변수화 된 유형이 아닌 객체를 가리킬 때 발생하는 상황입니다. 

> 코드 32-1 제네릭과 varargs를 혼용하면 타입 안정성이 깨진다!
```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42); // (1)
    Object[] objects = stringLists;      // (2)
    objects[0] = intList;                // (3)
    String s = stringLists[0].get(0);    // (4) ClassCastException
}
```
=> item28 과 마찬가지로 제네릭의 불공변(invariant) 특성이 가려지게 됩니다. 그래서 위의 코드처럼 제네릭을 사용함에도 불구하고, 타입이 다른 객체를 잘못 참조하게 되면 런타임 오류인 `ClassCastException` 이 발생합니다. 
그럼 item28처럼 **(타입이 다른 이유로 인한) 런타임 오류**가 발생함에도 왜 자바는 경고에서만 그치고 제네릭과 varargs를 함께 선언하는 것을 허용할까요?
 
## 제네릭과 varargs 매개변수를 사용할 수 있는 이유
 
* (책에서만 말하는 것 그대로) 실무에서 매우 유용하기 때문입니다. 자바에서는 이런 메서드를 여럿 제공하는데, `Arrays.asList(T... a), Collections.addAll(Collection<? super T> c, T... elements), EnumSet.of(E first, E... rest)`가 대표적입니다. 물론 이 메서드들은 **타입 안전**합니다.
* 추측해보건데, 메서드들의 타입 안전이 확실하다면 `Object...` 를 사용하는 것보다 컴파일 타임에 **타입 검사 가능한 이점**과 특정 타입의 varargs(ex: `Integer...`)를 사용하는 것보다 메서드의 **타입 사용 범위가 넓어진다는 이점**에서 사용 허용하는 것 같습니다.

## @SafeVarargs 어노테이션을 사용해 경고를 없애서 호출자를 안심시키자. 

* 자바 7 전에는 제네릭 가변인수 메서드의 작성자가 호출자 쪽에서 발생하는 경고에 대해서 해줄 수는 있는 일이 없었습니다.따라서 이런 메서드는 사용하기에 꺼림칙했고, 사용자는 이 경고들을 그냥 두거나 (더 흔하게는) 호출하는 곳마다 @SuppressWarings("unchecked") 에너테이션을 달아 경고를 숨겨야 했습니다. 
* 자바 7에서는 @SafeVarargs 에너테이션이 추가되어 제네릭 가변인수 메서드 작성자가 클라이언트 측에서 발생하는 경고를 숨길 수 있게 되었습니다. 
**@SafeVarargs 에너테이션은 메서드 작성자가 그 메서드가 타입 안전함을 보장하는 장치입니다.** 컴파일러는 이 약속을 믿고 그 메서드가 안전하지 않을 수 있다는 경고를 더 이상 하지 않습니다. 

=> 메서드가 안전한게 확실하지 않다면 절대 @SafeVarargs 에너테이션을 달아서는 안됩니다. 그럼 어떻게 메서드가 타입 안전한지 알 수 있을까요? 

## @SafeVarargs를 써도 되는 간단한 규칙 

@SafeVarargs를 써도 되는 간단한 규칙 두 가지가 있습니다. 
  
### varargs 매개변수 배열에 아무것도 저장하지 않는다. 

따라서 다음과 같은 코드는 **매개변수 배열에 intList를 저장하므로** 위 규칙에 위반됩니다. 
```java
static void dangerous(List<String>... stringLists) {
    List<Integer> intList = List.of(42); // (1)
    Object[] objects = stringLists;      // (2)
    objects[0] = intList;                // (3)
}
```

### 그 배열(혹은 복제본)을 신뢰할 수 없는 코드에 노출하지 않는다. 
신뢰할 수 없는 코드에 노출하는 경우 

따라서 다음과 같은 코드는 신뢰할 수 없는 코드에 노출해서 위 규칙에 위반됩니다. 심지어  varargs 매개변수 배열에 아무것도 저장하지 않고도 타입 안정성을 깰 수 있으니 주의해야 합니다. 

> 코드 32-2 자신의 제네릭 매개변수 배열의 참조를 노출한다 - 안전하지 않다! 
```java
static <T> T[] toArray(T... args) {
  return args;
}
```
=> 이 메서드가 반환하는 배열의 타입은 이 메서드에 인수를 넘기는 컴파일 타입에 결정되는데, 그 시점에는 컴파일러에게 충분한 정보가 주어지지 않아 타입을 **잘못 판단**할 수 있습니다. 따라서 자신의 varargs 매개변수 배열을 그대로 반환하면 힙 오염을 이 메서드를 호출한 쪽의 콜스택으로까지 전이하는 결과를 낳을 수 있습니다.  

```java
static <T> T[] pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
        case 0: return toArray(a, b);
        case 1: return toArray(a, c);
        case 2: return toArray(b, c);
    }
    throw new AssertionError();
}
```
=> pickTwo 메서드는 내부에서 toArray 메서드를 호출합니다. 이 메서드가 toArray()에 넘길 배열의 타입은 Object[]가 되는데, pickTwo에 어떤 타입의 객체를 넘기더라도 담을 수 있는 구체적인 타입이기 때문입니다.
만약 toArray에 String 값이 넘어가면 반환값이 String[] 으로 확실해지지만, 위의 예제처럼 제네릭 T 타입를 넘긴 경우 T가 어떤 타입이어도 대응해야 하기 때문에 Object[]가 반환되야 한다는 것입니다. 

```java
public static void main(String[] args) {
    String[] attributes = pickTwo("좋은", "빠른", "저렴한");
}
```
=> 따라서 위 메서드는 아무 문제가 없는 메서드이니 별다른 경고 없이 컴파일 되지만, 결국 pickTwo의 반환타입이 Object[]라 String[] 으로 자동 형변환되어 ClassCastException 이 발생합니다. 정리하지면 제네릭 varargs(T...) 매개변수를 그대로 반환하고 또 제네릭을 이용해 반환하는 메서드(pickTow)로 인해 ClassCastException 이 발생한 것입니다. 

* 이 예는 제네릭 varargs 매개변수 배열에 다른 메서드가 접근하도록 허용하면 안전하지 않다는 점을 말합니다. 단, 예외가 두 가지 있습니다.
  * 첫번째, @SafeVarargs로 제대로 애노테이트된 또 다른 varargs 메서드에 넘기는 것은 안전합니다.
  * 두번째, 그저 이 배열 내용의 일부 함수를 호출만 하는 (결과적으로 리턴받지 않는) 일반 메서드에 넘기는 것도 안전합니다. 


## 제네릭 varargs 매개변수를 안전하게 사용하는 전형적인 예 

```java
static <T> List<T> flatten(List<? extends T>... lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

또 다른 방법으로는 제네릭 가변인자를 사용하지 않고 List로 대체하는 경우입니다. 

## List로 대체하기 

```java
static <T> List<T> flatten(List<List<? extends T>> lists) {
    List<T> result = new ArrayList<>();
    for (List<? extends T> list : lists) {
        result.addAll(list);
    }
    return result;
}
```

```java
List<String> audience = flatten(List.of(friends, romans, countrymen));
```

```java
static <T> List<T> pickTwo(T a, T b, T c) {
    switch (ThreadLocalRandom.current().nextInt(3)) {
        case 0: return List.of(a, b);
        case 1: return List.of(a, c);
        case 2: return List.of(b, c);
    }
    throw new AssertionError();
}
```

```java
List<String> attributes = pickTwo("좋은", "빠른", "저렴한");
```

# Swift

## Swift에서는 위의 자바와 같은 문제가 모두 해결됩니다. 

* 애초에, Swift에서 가변인수 및 배열은 Value(값) 타입이라 다른 값을 저장함으로써 발생하는 문제가 생기지 않습니다.

```Swift
func dangerous(stringLists: [String]...) {
    let intList: [Int] = [42]
    var objects: [Any] = stringLists
    objects[0] = intList
    let s: String = stringLists[0][0];
}
```
=> 위 코드에서 stringLists를 objects 변수에 할당할 때부터 둘은 같은 객체(같은 주소값)를 바라보는게 아니라 복사가 되는 것이기 때문에 이후에 intList 값을 저장해서 일어나는 문제가 발생하지 않습니다. 

=> 그리고 Swift 의 모든 컬렉션(Array,Set,Dictionary)은 Value(값) 타입이므로 모든 컬렉션에서 위 문제는 발생하지 않습니다. 

* Swift는 타입 안정성 언어입니다. Swift는 타입안정성을 추구하기 때문에 코드를 컴파일할 때 타입 확인 작업을 수행하고 잘못된 타입이 있다면 오류로서 표시합니다. 그 근거로 Swift 는 타입 추론을 해주는데 컴파일 타임시 모든 변수의 타입이 확인 가능합니다.  

```Swift
func toArray<T>(args: T...) -> [T] {
    return args
}
    
func pickTwo<T>(_ a: T,_ b: T,_ c: T) -> [T]? {
    switch Int.random(in: 0 ... 2) {
    case 0: return toArray(args: a, b)
    case 1: return toArray(args: b, c)
    case 2: return toArray(args: c, a)
    default:
        return nil
    }
}

let attributes = pickTwo("a", "b", "c")
// attributes의 타입은 [String]? 이라고 타입 추론됩니다. 
```
=> 제네릭을 이중으로 쓴 위의 예시도 타입 추론이 된 걸 보면 스위프트는 자바와 달리 컴파일 타임에 모든 타입이 정확히 추론됨을 알 수 있습니다.

## 번외:  왜 가변인수를 사용할까? 그냥 배열을 사용하면 되지 않을까?

https://stackoverflow.com/questions/27689220/why-cant-we-just-use-arrays-instead-of-varargs

https://www.baeldung.com/java-varargs

