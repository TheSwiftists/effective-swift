# Item 6. 불필요한 객체 생성을 피하라

똑같은 기능의 객체를 매번 생성하기보다는 객체 하나를 재사용하는 편이 나을 때가 많습니다. 재사용은 빠르고 세련됩니다. 특히 불변 객체(아이템 17)은 언제든 재사용할 수 있습니다.

불필요한 객체를 생성하는 경우, 즉 객체를 재사용하는게 좋은 경우에 대해 알아봅시다.

<br>

### 자주 사용되는 객체의 재사용

같은 기능을 가진 객체를 새로 생성하는 것보다는 재사용하는 것이 나을 때가 있습니다. 이처럼 싱글톤 패턴으로 인스턴스를 만들고 재사용할 수 있습니다.

```swift
class Device {
    private static let sharedInstance: Device = {
        let instance = Device()
        // 디바이스 설정
        return instance
    }()

    private init() { }

    public static func initMethod() -> Device {
        return instance
    }
}
```

<br>

### 인스턴스 생성 비용이 높은 객체의 재사용

서버에서 받아온 데이터나 디스크에서 읽어온 데이터처럼 큰 용량의 객체나 인스턴스를 생성할 때 비용의 높은 객체의 경우에도 객체를 캐싱하는 등 재사용하는 것이 좋습니다.

```swift
class ArticleLoader {
    typealias Handler = (Result<Article, Error>) -> Void

    private let cache = Cache<Article.ID, Article>()

    func loadArticle(withID id: Article.ID,
                     then handler: @escaping Handler) {
        if let cached = cache[id] {
            return handler(.success(cached))
        }

        performLoading { [weak self] result in
            let article = try? result.get()
            article.map { self?.cache[id] = $0 }
            handler(result)
        }
    }
}
```



<br>

이번 아이템을 "객체 생성은 비싸니 피해야 한다"로 오해하면 안 됩니다. ARC같은 자동 참조 해제 기능이 있으므로, 작은 객체를 생성하고 회수하는 일이 크게 부담되지 않습니다. 프로그램의 명확성, 간결성, 기능을 위해 객체를 추가로 생성하는 것이라면 일반적으로 좋은 일입니다.

 거꾸로, 아주 무거운 객체가 아닌 다음에야 단순히 객체 생성을 피하고자 여러분만의 *객체 풀(pool)을 만들지는 마세요. 물론 객체 풀을 만드는 게 나은 예가 있긴 합니다. 네트워크 연결처럼 접근 비용이 비싸지는 경우 재사용하는 편이 낫습니다. 하지만 일반적으로는 자체 객체 풀은 코드를 헷갈리게 만들고 메모리 사용량을 늘리고 성능을 떨어뜨립니다. 

> 객체 풀(Object pool)?
>
> 객체를 매번 할당, 해제하지 않고 고정 크기 풀에 들어있는 객체를 재사용함으로써 메모리 사용 성능을 개선함.
>
> 객체들의 크기가 비슷하거나 객체를 빈번하게 생성/삭제하는 경우, 객체를 힙에 생성하기가 느리거나 생성 비용이 비싼 객체를 캡슐화하는 경우 사용. 객체를 위한 메모리 크기가 고정되어 가장 큰 자료형에 맞춰야하고, 메모리 낭비 가능성이 있어 사용에 주의해야 함.



### 결론

이번 아이템은 방어적 복사(defensive copy)를 다루는 아이템 50과 대조적입니다. 이번 아이템이 **"기존 객체를 재사용해야 한다면 새로운 객체를 만들지 마라"**라면, 아이템 50은 "새로운 객체를 만들어야 한다면 기존 객체를 재사용하지 마라"입니다. **방어적 복사가필요한 상황에서 객체를 재사용했을 때의 피해가, 필요 없는 객체를 반복 생성했을 때의 피해보다 훨씬 크다는 사실을 기억하세요. **방어적 복사에 실패하면 언제 터져 나올지 모르는 버그와 보안 구멍으로 이어지지만, 불필요한 객체 생성은 그저 코드 형태와 성능에만 영향을 줍니다.



##### References

- [디자인 패턴 - 객체 풀(Object pool)](http://hajeonghyeon.blogspot.com/2017/06/object-pool.html)

- [cashing in swift - swiftbysundell](https://www.swiftbysundell.com/articles/caching-in-swift/)
