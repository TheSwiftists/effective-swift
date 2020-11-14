
# Item 17. 변경 가능성을 최소화하라

인스턴스의 내부 값을 수정할 수 없는 불변 클래스는 설계하고 구현하고 사용하기 쉬우며, 오류가 생길 여지도 적고 안전합니다.

클래스를 불변으로 만들려면 다음 규칙들을 따르면 됩니다.

- 객체의 상태를 변경하는 메서드를 제공하지 않는다.
- 클래스의 서브클래싱을 막아서 하위 클래스에서 객체의 상태를 변경하지 못하도록 막는다.
- 모든 필드를 final로 선언하여 변경되지 않도록 만든다.
- 모든 필드를 private으로 선언하여 필드가 참조하는 가변 객체까지 변경하지 못하도록 한다.
- 자신 외에는 내부의 가변 컴포넌트에 접근할 수 없도록 한다.

### 불변 객체의 장점

불변 객체의 장점은 다음과 같습니다.

- 생성된 시점의 상태를 파괴될 때까지 그대로 간직하고 있어서 단순하다. 가변 객체는 임의의 복잡한 상태에 놓일 수 있다.
- Thread-Safe하여 안심하고 공유할 수 있다. 이를 활용하여 자주 쓰이는 값들을 여러 클라이언트가 재활용할 수 있어 메모리 사용량과 GC 비용이 줄어든다.
- 불변 객체들끼리 내부 데이터를 공유할 수 있다.
- 객체를 만들 때 다른 불변 객체들을 구성요소로 사용하면 이점이 많다.
- 불변 객체는 그 자체로 실패 원자성을 제공한다. 일시적으로 inconsistency한 상태에 빠질 가능성이 없다.

### 불변 객체의 단점

반면, 불변 객체는 값이 다르면 반드시 독립된 객체로 만들어야 해서 값의 가짓수가 많다면 객체 생성 비용이 많이 들 수 있다는 단점도 있습니다.

### Immutable 클래스 예시

```java
public final class Complex {
		private final double re;
		private final double im;

		public Complex(double re, double im) {
				this.re = re;
				this.im = im;
		}

		public Complex plus(Complex c) {
				return new Complex(re + c.re, im + c.im);
		}
}
```

연산 메서드에서는 자신을 수정하지 않고 새로운 인스턴스를 만들어 반환합니다. 이 때 메서드의 이름으로 객체의 값을 변경하지 않는다는 점을 강조해 전치사 등을 사용했습니다. 이런 방식은 코드에서 불변이 되는 영역의 비율이 높아지도록 해 줍니다.

```swift
// 생성자를 접근하지 못하게 하고 정적 팩터리(아이템 1)를 제공하여 상속을 제한할 수도 있습니다.
public class Complex {
		private final double re;
		private final double im;

		private Complex(double re, double im) {
				this.re = re;
				this.im = im;
		}

		public static Complex valueOf(double re, double im) {
				return new Complex(re, im);
		}
}
```

이 방법을 사용할 경우 바깥에서 볼 수 없는 여러 구현 클래스들을 만들어 활용할 수 있고, 다음 릴리즈에서 API 변경 없이 기능 추가도 가능하여 유연합니다.

이런 식으로 단순한 값 객체는 반드시 불변으로 만드는 것이 좋고, 불변으로 만들 수 없는 클래스라도 변경할 수 있는 부분을 최소한으로 줄여서 예측하기 쉽게 만들고 오류 가능성을 낮춰야 합니다.

### Immutable in Swift

불변 객체의 예시를 구조체를 사용하여 작성해 보겠습니다. 구조체를 사용한 이유는 클래스보다 객체 생성 비용이 훨씬 적기 때문입니다. 앞서 언급한 불변 객체의 단점인 객체 생성 비용이 많이 들 수 있다는 점을 어느 정도 완화해 줍니다.

또한, 스위프트에서는 구조체 안에서 프로퍼티의 값을 변경하려 할 때 해당 프로퍼티가 `var`로 선언되어 있더라도 디폴트로 자기 자신을 변경하지 못하게 하기 때문에, 내부 프로퍼티의 변경을 지양하려는 저자의 구현 방식과 부합한다고 생각했습니다. 또한 구조체는 상속이 제한되어 있어 이전에 언급한 규칙들 중 서브클래싱을 막아야 한다는 규칙은 자동으로 만족합니다.

```swift
struct Complex {
		private var real: Double
		private var imaginary: Double

    // 자기 자신의 프로퍼티를 변경하려 할 경우 mutating 키워드를 추가해야 합니다.
    mutating func add(_ complex: Complex) {
        real += complex.real
        imaginary += complex.imaginary
    }

		func plus(_ complex: Complex) -> Complex {
				return Complex(real: real + complex.real, imaginary: imaginary + complex.imaginary)
    }
}
```

자기 자신을 수정하지 않고 새로운 인스턴스를 만들어 반환하도록 구현하였습니다.

### 스위프트에서의 메서드 네이밍 규칙

앞서 언급한 바와 같이 스위프트에서도 메서드의 동작에 따른 네이밍 규칙이 있습니다. Swift API Design Guidelines에 따르면, side-effect를 고려하여 메서드의 이름을 지어야 합니다.

- side-effect가 없으면 x.distance(to: y), i.successor()와 같이 명사형으로 짓습니다.
- side-effect가 있으면 반드시 동사형으로 읽혀야 합니다. 예를 들어, x.sort(), x.append()와 같습니다.
- Mutating/nonmutating pair들을 일관적으로 짓습니다. mutating 메서드가 비슷한 동작을 하지만 인스턴스의 값을 업데이트하는 대신 새로운 값을 반환하는 nonmutating variant를 가질 수 있습니다.
    - x.sort() ←→ z = x.sorted()
    - x.append(y) ←→ x.append(y)
    - y.formUnion(z) ←→ x = y.union(z)
