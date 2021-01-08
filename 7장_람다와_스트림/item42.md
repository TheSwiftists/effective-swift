# item 42. 익명 클래스보다는 람다를 사용하라


### 왜 그럴까요?
익명 클래스 방식은 코드가 너무 길어집니다.
람다를 사용하게 되면 익명 클래스와 개념은 비슷하지만 코드는 훨씬 간결해지는 장점을 누릴 수 있습니다.

```java
// 익명 클래스를 사용했을때.
Collections.sort(words, new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
});

// 익명 클래스만 떼서 보면?
new Comparator<String>() {
    public int compare(String s1, String s2) {
        return Integer.compare(s1.length(), s2.length());
    }
}
```

람다식(람다)는 함수형 인터페이스(추상 메서드 하나짜리 인터페이스)의 인스턴스를 만들 때 사용합니다.
밑의 코드는 위의 익명 클래스 부분을 람다 방식으로 바꾼 모습입니다.

```java
// 자바코드
Collections.sort(words,
        (s1, s2) -> Integer.compare(s1.length(), s2.length()));
```

이처럼 동일한 동작을 하는 코드를 3줄에서 1줄로 줄이고 어떤 동작을 하는지에 대한 가독성 또한 좋아지는 효과가 있습니다.

익명 클래스는 Swift에서 대치되는 개념이 없지만 람다식은 Swift의 closure와 몇몇 부분은 비교할 수 있습니다.

<br>

### 람다와 클로저(Closure)의 비교

#### 표현식
- 람다 : `(매개변수, ...) -> { 실행구문 };`
- 클로저 : `{ (매개 변수) -> (반환 타입) in (실행구문) }`

#### 개념
- 람다 : 함수형 인터페이스를 구현하는 표현식입니다.
    ```java
    // 익명클래스 혹은 구현체를 만들어 calc 메소드를 Override 하는 방법도 있다.
    interface Calculator {
        int calc(int n);
    }
    
    // 람다로 구현하면?
    public static void main(String[] args) {
        int n = 2;
        Calculator cal = (n) -> {return n + 1;};

        System.out.println(cal.calc(n)); // 3
    }
    ```
- 클로저 : 코드 내부 혹은 전달도 가능한 '독립된 기능 블럭'입니다.
    ```swift
    let cal = { (n: Int) -> Int in
        return n+1
    }
    
    cal(2) // 3
    
    // 전달
    func plusUsingClosure(number: Int, closure: (Int) -> Int) -> Int {
        return closure(number)
    }
    
    plusUsingClosure(number: 2, closure: ci)
    ```

#### 둘 모두 일급객체
- 모든 일급 객체는 함수의 실질적인 매개변수가 될 수 있습니다.
- 모든 일급 객체는 함수의 반환값이 될 수 있습니다.
- 모든 일급 객체는 할당의 대상이 될 수 있습니다.
- 모든 일급 객체는 비교 연산(==, equal)을 적용할 수 있습니다. <br> (다만, Swift에서는 비교연산을 권장하지 않는다고 합니다. [관련 링크](https://stackoverflow.com/questions/24111984/how-do-you-test-functions-and-closures-for-equality))


#### 타입추론
- 람다 : 컴파일러가 타입 추론을 해줍니다. <br> 타입을 명시해야 코드가 더 명확할 때 / 컴파일러가 타입을 알 수 없다는 오류를 낼때를 제외하고, 람다의 모든 매개변수 타입은 생략하는 것을 권장했습니다.
- 클로저 : 클로저 또한 컴파일러가 타입 추론을 해줍니다.
    ```swift
    let names = ["Chris", "Alex", "Ewa", "Barry", "Daniella"]
    
    reversedNames = names.sorted(by: { (s1: String, s2: String) -> Bool in
        return s1 > s2
    })
    
    // 타입 추론이 가능 하므로 매개변수 타입 및 반환 타입 생략
    reversedNames = names.sorted(by: { s1, s2 in return s1 > s2 } )
    ```
    이처럼 메소드는 문자열 배열에서 호출되므로, 인자는 (String, String) -> Bool 타입의 함수일 수 밖에 없을 것입니다. 이 말은 모든 타입을 추론할 수 있으므로, ‘반환 표시 화살표 (->)’ 와 매개 변수 이름 주위의 괄호도 생략이 가능하게 됩니다. <br>
공식 문서에서는 람다처럼 매개변수 타입을 생략하는것을 권장하진 않고, 필요하거나 코드를 읽을때 모호함을 피하기 위함이면 언제든 타입을 표현해도 된다고 설명하고 있습니다.
    
<br>

### 람다 사용 시 권장사항
해당사항은 클로저를 만들때도 적용하면 좋을 것 같아서 가져와보았습니다.
> 람다는 이름이 없고 문서화도 못 한다. 따라서 코드 자체로 동작이 명확히 설명되지 않거나 코드 줄 수가 많아지면 람다를 쓰지 말아야한다.

<br>

### 참고한 곳
- https://juyoung-1008.tistory.com/48
- https://galid1.tistory.com/509
- https://jeong-pro.tistory.com/208
- https://ko.wikipedia.org/wiki/%EC%9D%BC%EA%B8%89_%EA%B0%9D%EC%B2%B4
- https://docs.swift.org/swift-book/LanguageGuide/Closures.html
- http://xho95.github.io/swift/language/grammar/closure/2020/03/03/Closures.html
