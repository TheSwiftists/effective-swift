# Item73. 추상화 수준에 맞는 예외를 던지라 

## Java

수행하려는 일과 관련 없어 보이는 예외가 튀어나오면 당황스러울 것입니다. 
메서드가 저수준 예외를 처리하지 않고 바깥으로 전파해버릴때 종종 일어나는 일입니다.
당황시키는 것 뿐만 아니라 내부 구현방식을 상위에 드러내어 **윗 레벨 API를 오염 시킬 수 있고**, 다음 릴리스에서 구현방식을 바꾸면 **다른 예외가 튀어나와 기존 클라이언트 프로그램을 깨지게 할 수도 있습니다.** 해결책으로 예외 번역, 예외 연쇄가 있습니다.  

### 예외 번역

상위 메서드에서는 저수준 예외를 잡아 자신의 추상화 수준에 맞는 예외로 바꿔 던져야 합니다.
이를 예외 번역(Exception Translation)이라 합니다.

```java
try {
    ... // 저수준 추상화를 이용한다. 
} catch (LowerLevelException e) {
    // 추상화 수준에 맞게 번역한다. 
    throw new HigherLevelException(...);
}
```

```java
    /**
     * Returns the element at the specified position in this list.
     *
     * <p>This implementation first gets a list iterator pointing to the
     * indexed element (with {@code listIterator(index)}).  Then, it gets
     * the element using {@code ListIterator.next} and returns it.
     *
     * @throws IndexOutOfBoundsException {@inheritDoc}
     */
    public E get(int index) {
        try {
            return listIterator(index).next();
        } catch (NoSuchElementException exc) {
            throw new IndexOutOfBoundsException("Index: "+index);
        }
    }
```

### 예외 연쇄 

예외 연쇄(Exception chaining)이란 문제의 근본 원인(cause)인 저수준 예외를 고수준 예외에 실어 보내는 방식입니다.
별도의 접근자 메서드(Throwable의 getCause메서드)를 통해 필요하면 언제든 저수준 예외를 꺼내 볼 수 있습니다. 

```java
try {
    ... // 저수준 추상화를 이용한다.
} catch (LowerLevelException e) {
    // 저수준 예외를 고수준 예외에 실어 보낸다.
    throw new HigherLevelException(cause);
}
```

대부분의 표준 예외는 예외 연쇄용 생성자를 갖추고 있습니다.
그렇지 않은 예외라도 Throwable의 initCause 메서드를 이용해 원인 을 직접 못박을 수 있습니다
예외 연쇄는 문제의 원인을 프로그램에서 접근할 수 있게 해주며 원인과 고수준 예외의 Stack trace를 잘 통합해줍니다.

```java
public class Exception extends Throwable {
    public Exception() {
        super();
    }    

    /// 생략

    // 예외 연쇄용 생성자 
    public Exception(Throwable cause) {
        super(cause);
    }
}    

**가능하다면 저수준 메서드가 반드시 성공하도록 하여 아래 계층에서는 예외가 발생하지 않도록 하는 것이 최선입니다.때로는 상위 계층 메서드의 매개변수 값을 아래 계층 메서드로 건네기 전에 미리 검사하는 방법으로 달성할 수 있습니다.** 
차선책도 있습니다. 아래 계층에서의 예외를 피할 수 없다면, 상위 계층에서 그 예외를 조용히 처리하여 문제를 API 호출자에 전파하지 않는 방법이 있습니다.
이 경우 발생한 예외는 로깅를 활용하여 개발자가 분석할 수 있도록 조치를 취하게 해줍니다. 

## Swift

* Alamofire의 `AFError` 를 참고하였습니다. 

### Enum의 associated value 사용하여 에러 연쇄 나타내기 

* 상위 에러는 AFError.parameterEncodingFailed 이고 하위 에러는 ParameterEncodingFailureReason 의 연관값인 `error` 입니다.  

```swift
public enum AFError: Error {
        /// The underlying reason the `.parameterEncodingFailed` error occurred.
    public enum ParameterEncodingFailureReason {
        /// The `URLRequest` did not have a `URL` to encode.
        case missingURL
        /// JSON serialization failed with an underlying system error during the encoding process.
        case jsonEncodingFailed(error: Error)
        /// Custom parameter encoding failed due to the associated `Error`.
        case customEncodingFailed(error: Error)
    }

    case parameterEncodingFailed(reason: ParameterEncodingFailureReason)
    // 나머지 생략
}
```

```swift
open class JSONParameterEncoder: ParameterEncoder {
    // 생략

    /// `JSONEncoder` used to encode parameters.
    public let encoder: JSONEncoder

    /// Creates an instance with the provided `JSONEncoder`.
    ///
    /// - Parameter encoder: The `JSONEncoder`. `JSONEncoder()` by default.
    public init(encoder: JSONEncoder = JSONEncoder()) {
        self.encoder = encoder
    }

    open func encode<Parameters: Encodable>(_ parameters: Parameters?,
                                            into request: URLRequest) throws -> URLRequest {
        guard let parameters = parameters else { return request }

        var request = request

        do {
            let data = try encoder.encode(parameters)  
            request.httpBody = data
            if request.headers["Content-Type"] == nil {
                request.headers.update(.contentType("application/json"))
            }
        } catch {
            throw AFError.parameterEncodingFailed(reason: .jsonEncodingFailed(error: error)) // 이 에러가 하위 에러인 EncodingError.invalidValue 이다. 
        }

        return request
    }

    // 생략
}

```

### Enum의 연산 프로퍼티 사용하여 LowerLevel 에러 원인 나타내기 

* 연산 프로퍼티를 이용해 AFError을 사용하는 외부에서 쉽게 하위 에러가 무엇인지 파악하고 에러 처리할 수 있습니다.  

```swift
extension AFError.ParameterEncodingFailureReason {
    var underlyingError: Error? {
        switch self {
        case let .jsonEncodingFailed(error),
             let .customEncodingFailed(error):
            return error
        case .missingURL:
            return nil
        }
    }
}

extension AFError {
    public var underlyingError: Error? {
        switch self {
        case let .parameterEncodingFailed(reason):
            return reason.underlyingError
        // 이하 생략
        }
    }
}
```
