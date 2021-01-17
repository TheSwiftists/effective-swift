# item40. @Override 애너테이션을 일관되게 사용하라

> override 키워드를 붙여야 하는 이유

<br>

## Java

`@Override` 애너테이션은 가장 자주 사용되는 애너테이션 중 하나일 것입니다. `@Override` 애너테이션은 메서드 선언에만 달 수 있으며, 이 애너테이션이 달렸다는 것은 상위 타입의 메서드를 재정의했음을 뜻합니다. 이 애너테이션을 일관되게 사용하면 여러가지 버그들을 예방해줍니다. 다음의 Bigram 프로그램을 함께 살펴봅시다. 이 클래스는 바이그램, 즉 영어 알파벳 2개로 구성된 문자열을 표현합니다.

```java
// 영어 알파벳 2개로 구성된 문자열을 표현하는 클래스 - 버그를 찾아봅시다
public class Bigram {
  private final char first;
  private final char second;
  
  pulbic Bigram(char first, char second) {
    this.first = first;
    this.second = second;
  }
  public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
  }
  public int hashCode() {
    return 31 * first + second;
  }
  
  public static void main(String[] args) {
    Set<Bigram> s = new HashSet<>();
    for(int i = 0; i < 10; i++) {
      for(char ch = 'a'; ch <= 'z'; ch++) 
        s. add(new Bigram(ch, ch));
      System.out.println(n.size());
    }
  }
}
```



`main` 메서드를 보면 똑같은 소문자 2개로 구성된 바이그램 26개를 10번 반복해 집합에 추가한 다음, 그 집합의 크기를 출력합니다. Set은 중복을 허용하지 않으니 26이 출력될 것 같지만, 실제로는 260이 출력됩니다. 무엇인가 잘못되었군요.

확실히 Bigram 작성자는 `equals` 메서드를 재정의하려 한 것으로 보이고 `hashCode`를 재정의해야 한다는 사실도 잊지 않았습니다. 그러나 안타깝게도 `equals`를 재정의(overriding) 한 게 아니라 다중정의(overloading) 해버렸습니다. `Object`의 `equals`를 재정의하려면 매개변수 타입을 `Object`로 해야만 하는데, 그렇게 하지 않은 것입니다. 그래서 `Object`에서 상속한 `equals`와는 별개인 `equals`를 새로 정의한 꼴이 되었습니다. 

이러한 오류를 컴파일러가 찾아주도록 하기 위해선 `@Override` 애너테이션을 붙여 재정의한다는 의도를 명시해야 합니다.

```java
@Override public boolean equals(Bigram b) {
    return b.first == first && b.second == second;
}
```

그럼 컴파일 오류가 발생해 수정이 필요한 부분들을 지적해 개발자로 하여금 수정할 수 있게 합니다. 그러므로 상위 클래스의 메서드를 재정의하려는 모든 메서드에 `@Override` 애너테이션을 달아야 합니다. 



<br>

## Swift

Swift에서도 비슷한 상황이 발생할 수 있습니다.

```swift
class SuperClass {
    func method(param: String) {
        print(param)
    }
}

class SubClass: SuperClass {
    func method(param: Int) {
        print(param)
    }
}

let sub = SubClass()
sub.method(param: "str")	//str
```

SubClass가 SuperClass를 상속받고 있고 타입만 다른 메서드를 재정의하려고 합니다. 하지만 다른 타입의 파라미터를 지정함으로 재정의(overriding)된 것이 아니라 다중정의(overloading)되어버렸습니다. 



이를 방지하기 위해서는 재정의하려는 메소드 앞에 `override` 키워드를 추가합니다. 이렇게 하면 재정의하려는 의도를 명확히 할 수 있으며 예기치 않은 오동작을 방지할 수 있습니다. `override` 키워드가 없는 재정의는 코드가 컴파일 될 때 오류로 진단됩니다. 또한 `override` 키워드를 붙임으로 재정의하는 클래스의 수퍼클래스 중 재정의를 위한 선언과 일치하는 선언이 있는지 Swift 컴파일러가 확인하도록 합니다.



<br>



### 참고

- [Swift Override](https://docs.swift.org/swift-book/LanguageGuide/Inheritance.html#ID196)
