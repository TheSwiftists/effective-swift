# 인스턴스화를 막으려거든 private 생성자를 사용하라

### 이따금 단순히 정적 메서드와 정적 필드만을 담은 클래스를 만들고 싶을 때가 있을 것이다.
전역적으로 사용하기 위함. 클래스를 인스턴스화 하지 않고도 직접 호출할 수 있다.
보통 안티패턴이라고 이야기하는데, 그럼 언제 사용해야 할까?

여기서도 예제를 들어주고 있는데 
1. **관련된 메서드들을 한데 모아놓고 사용할때**
ex) java.lang.Math, java.util.Arrays
(실제로 정적프로퍼티와 정적 메소드로 이루어져 있음)
Swift에서 기본 제공하는 Class에서는 찾을 수가 없었고 예시를 가져와봤습니다.
```switf
enum AppStyles {
  enum Colors {
    static let mainColor = UIColor(red: 1, green: 0.2, blue: 0.2, alpha: 1)
    static let darkAccent = UIColor(red: 0.2, green: 0.2, blue: 0.2, alpha: 1)
  }

  enum FontSizes {
    static let small: CGFloat = 12
    static let medium: CGFloat = 14
    static let large: CGFloat = 18
    static let xlarge: CGFloat = 21
  }
}
```
색상이나 글꼴 크기를 인스턴스화 시켜 앱 곳곳에 흩어져있게 하는것 보다 위와같이 한곳에 정의하는 것이 훨씬 관리하기 편하다. <br>
enum을 사용한 이유는 이니셜라이저가 없어 인스턴스화가 불가능하고, static property를 사용해 인스턴스화 하지 않고 바로 접근하기 위함이다. <br>
(전역적으로 설정 + 애초에 enum은 stored property를 가지고 있지 못함.)

2. **특정 인터페이스를 구현하는 객체를 생성해주는 정적 메서드나 팩토리를 구현할때**
ex) java.util.Collections
사용자 대신해서 객체의 인스턴스를 만들어 낸다던가, 간단하게 사용해서 복잡한 객체를 만들어 낼때 사용한다.
```switf
enum BlogpostFactory {
  static func create(withTitle title: String, body: String) -> Blogpost {
    let metadata = Metadata(/* metadata properties */)
    return Blogpost(title: title, body: body, createdAt: Date(), metadata: metadata)
  }
}
```
3. **final 클래스와 관련한 메서드들을 모아놓을때**

### 생성자를 명시하지 않으면 컴파일러가 자동으로 기본 생성자를 만들어준다. 
매개변수를 받지 않는 public 생성자가 만들어지고, 사용자는 구분할 수 없음.<br>
직접 선언한 생성자가 없는 경우에 한해서 컴파일러가 자동으로 기본 생성자를 만들어준다고 한다. <br>
-> 해당부분은 Swift와 동일하다.

### 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다. 
Swift는 추상 클래스라는 것이 존재하지 않는다. <br>
비슷한 개념이 있다면 Protocol이라 할 수 있겠다.

#### 그럼 Java에서 추상 클래스란?
Protocol과 비슷하다. <br>
(재)사용할 프로퍼티와 메소드 이름을 통일하여 객체의 유지보수성을 높이고 통일성을 유지할 수 있는 장점을 얻기 위해 사용한다.

다른 클래스가 추상 클래스를 '상속'받아 실제 내용(프로퍼티, 메소드)를 구현한다. <br>
Protocol은 클래스가 '채택'하여 사용이 가능하다.

### 추상 클래스로 만드는 것으로는 인스턴스화를 막을 수 없다.
추상 클래스 관련이야기라 Swift와 관계 없다고 판단된다. <br>
(Protocol로는 인스턴스화를 막을 수 없다..와 같은 맥락은 아닌것 같다.)

### private 생성자를 추가하면 클래스의 인스턴스화를 막을 수 있다.
java에서 컴파일러가 기본 생성자를 만드는 경우는 오직 명시된 생성자가 없을때 뿐이라고 한다. <br>
-> Swift또한 private한 생성자를 만들 수 있다. <br>
-> 여기서도 언급을 하지만, 생성자가 존재하는데 호출할 수 없는 코드는 직관적이지 않아 주석을 달아주는 것을 권장하고 있다.

해당 방식은 상속을 불가능하게 하는 효과가 있다. 
: 하위 클래스를 만들더라도, 상위 클래스의 생성자에 접근하지 못하므로 상속이 이루어지지 않는 효과도 동시에 누린다. <br>
-> 인스턴스화 + 상속의 문제를 동시에 해결하려면 Enum을 써야하는게 맞는것 같다. <br>
-> 상속의 문제는 final class를 만들면 되고, 파편화된 인스턴스화가 신경이 쓰인다면 공유인스턴스 혹은 싱글톤을 만들어 사용하면 될것 같다.

```swift
// 코드 4-1 인스턴스를 만들 수 없는 유틸리티 클래스 (26~27쪽)
class UtilityClass {
    // 기본 생성자가 만들어지는 것을 막는다(인스턴스화 방지용).
    
    private init() {
        print("private init!")
    }

    // 나머지 코드는 생략
}
```

- 인스턴스화 시도한 결과
![](https://i.imgur.com/EIt5PgR.png)

<br>

참고한 페이지
1. https://www.donnywals.com/effectively-using-static-and-class-methods-and-properties/
2. https://limkydev.tistory.com/188
