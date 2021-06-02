# Effective Swift

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fswiftyus%2Feffective-swift&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=true)](https://hits.seeyoufarm.com)

Effective Java 3/E을 읽고 프로그래밍에서의 관례적이고 효과적인 용법을 배우고, 스위프트에서의 활용 방안을 제안합니다.

<p align="center"><img width=400 src="http://image.yes24.com/goods/65551284/800x0"></p>

## Contents

- [Items](#items)
- [References](#References)

## Items

- 각 아이템 별로 요약 및 정리합니다.
- 예제 코드들을 스위프트로 다시 작성해 보고 스위프트에서의 활용 방안을 고민해 봅니다.

### 2장 객체 생성과 파괴

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 1 | [생성자 대신 정적 팩터리 메서드를 고려하라](2장_객체_생성과_파괴/item1.md) | | [delma][delma] |
| 아이템 2 | [생성자에 매개변수가 많다면 빌더를 고려하라](2장_객체_생성과_파괴/item2.md) | | [하이디][heidi] |
| 아이템 3 | [private 생성자나 열거 타입으로 싱글턴임을 보증하라](2장_객체_생성과_파괴/item3.md) | | [jinie][jinie] |
| 아이템 4 | [인스턴스화를 막으려거든 private 생성자를 사용하라](2장_객체_생성과_파괴/item4.md) | | [Lin][lin] |
| 아이템 5 | [자원을 직접 명시하지 말고 의존 객체 주입을 사용하라](2장_객체_생성과_파괴/item5.md) | 의존성 주입(Dependency Injection) ( + iOS 활용 예시) | [Lena][lena] |
| 아이템 6 | [불필요한 객체 생성을 피하라](2장_객체_생성과_파괴/item6.md) | | [delma][delma] |
| 아이템 7 | [다 쓴 객체 참조를 해제하라](2장_객체_생성과_파괴/item7.md) | Garbage Collection(GC) vs Automatic Reference Counting(ARC) (+ 메모리 누수(Memory Leak)와 해결 방법) | [Lena][lena] |
| 아이템 8 | [finalizer와 cleaner 사용을 피하라](2장_객체_생성과_파괴/item8.md) | Swift deinit 활용 예시 (feat. RxSwift의 Dispose()와 DisposeBag) | [Lena][lena] |
| 아이템 9 | [try-finally보다는 try-with-resources를 사용하라](2장_객체_생성과_파괴/item9.md) | | [Lin][lin] |

### 3장 모든 객체의 공통 메서드

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 10 | [equals는 일반 규약을 지켜 재정의하라](3장_모든_객체의_공통_메서드/item10.md) | | [하이디][heidi] |
| 아이템 11 | [equals를 재정의하려거든 hashCode도 재정의하라](3장_모든_객체의_공통_메서드/item11.md) | Hashable 프로토콜과 Equatable 프로토콜 | [Lena][lena] |
| 아이템 12 | [toString을 항상 재정의하라](3장_모든_객체의_공통_메서드/item12.md) | | [Lin][lin] |
| 아이템 13 | [clone 재정의는 주의해서 진행하라](3장_모든_객체의_공통_메서드/item13.md) | | [하이디][heidi] |
| 아이템 14 | [Comparable을 구현할지 고려하라](3장_모든_객체의_공통_메서드/item14.md) | | [delma][delma] |

### 4장 클래스와 인터페이스

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 15 | [클래스와 멤버의 접근 권한을 최소화하라](4장_클래스와_인터페이스/item15.md) | | [Lin][lin] |
| 아이템 16 | [public 클래스에서는 public 필드가 아닌 접근자 메서드를 사용하라](4장_클래스와_인터페이스/item16.md) | | [delma][delma] |
| 아이템 17 | [변경 가능성을 최소화하라](4장_클래스와_인터페이스/item17.md) | | [하이디][heidi] |
| 아이템 18 | [상속보다는 컴포지션을 사용하라](4장_클래스와_인터페이스/item18.md) | | [하이디][heidi] |
| 아이템 19 | [상속을 고려해 설계하고 문서화하라. 그러지 않았다면 상속을 금지하라](4장_클래스와_인터페이스/item19.md) | | [Lin][lin] |
| 아이템 20 | [추상 클래스보다는 인터페이스를 우선하라](4장_클래스와_인터페이스/item20.md) | | [delma][delma] |
| 아이템 21 | [인터페이스는 구현하는 쪽을 생각해 설계하라](4장_클래스와_인터페이스/item21.md) | 프로토콜 기본 구현(Protocol Default Implementation) | [Lena][lena] |
| 아이템 22 | [인터페이스는 타입을 정의하는 용도로만 사용하라](4장_클래스와_인터페이스/item22.md) | | [delma][delma] |
| 아이템 23 | [태그 달린 클래스보다는 클래스 계층구조를 활용하라](4장_클래스와_인터페이스/item23.md) | | [하이디][heidi] |
| 아이템 24 | [멤버 클래스는 되도록 static으로 만들라](4장_클래스와_인터페이스/item24.md) | | [Lin][lin] |
| 아이템 25 | [톱레벨 클래스는 한 파일에 하나만 담으라](4장_클래스와_인터페이스/item25.md) | Swift에서의 톱레벨 클래스 선언에 대해 공부하다 알아본 개체를 구분하는 방식 '네임스페이스' 그리고 Swift에서의 '모듈'과 '프레임워크' | [Lena][lena] |

### 5장 제네릭

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 26 | ~~로 타입은 사용하지 말라~~ | | |
| 아이템 27 | [비검사 경고를 제거하라](5장_제네릭/item27.md) | Swift의 타입 캐스팅과 컴파일 오류 | [Lena][lena] |
| 아이템 28 | [배열보다는 리스트를 사용하라](5장_제네릭/item28.md) | Swift에는 Array만 있는데요? | [delma][delma] |
| 아이템 29 | [이왕이면 제네릭 타입으로 만들라](5장_제네릭/item29.md) | Swift Generic의 거의 모든 것 (+ Any vs Generic) | [Lena][lena] |
| 아이템 30 | [이왕이면 제네릭 메서드로 만들라](5장_제네릭/item30.md) | | [Lin][lin] |
| 아이템 31 | [한정적 와일드카드를 사용해 API 유연성을 높이라](5장_제네릭/item31.md) | | [하이디][heidi] |
| 아이템 32 | [제네릭과 가변인수를 함께 쓸 때는 신중하라](5장_제네릭/item32.md) | | [Jason][jason] |
| 아이템 33 | [타입 안전 이종 컨테이너를 고려하라](5장_제네릭/item33.md) | | [하이디][heidi] |

### 6장 열거 타입과 애너테이션

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 34 | [int 상수 대신 열거 타입을 사용하라](6장_열거_타입과_애너테이션/item34.md) | | [Lena][lena] |
| 아이템 35 | [ordinal 메서드 대신 인스턴스 필드를 사용하라](6장_열거_타입과_애너테이션/item35.md) | | [Jason][jason] |
| 아이템 36 | [비트 필드 대신 EnumSet을 사용하라](6장_열거_타입과_애너테이션/item36.md) | | [gwonii][gwonii] |
| 아이템 37 | [ordinal 인덱싱 대신 EnumMap을 사용하라](6장_열거_타입과_애너테이션/item37.md) | | [Lena][lena] |
| 아이템 38 | [확장할 수 있는 열거 타입이 필요하면 인터페이스를 사용하라](6장_열거_타입과_애너테이션/item38.md) | | [Lin][lin] |
| 아이템 39 | [명명 패턴보다 애너테이션을 사용하라](6장_열거_타입과_애너테이션/item39.md) | Swift의 Attributes에 대해 알아봅시다 | [delma][delma] |
| 아이템 40 | [@Override 애너테이션을 일관되게 사용하라](6장_열거_타입과_애너테이션/item40.md) | override 키워드를 붙여야 하는 이유 | [delma][delma] |
| 아이템 41 | [정의하려는 것이 타입이라면 마커 인터페이스를 사용하라](6장_열거_타입과_애너테이션/item41.md) | | [gwonii][gwonii] |

### 7장 람다와 스트림

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 42 | [익명 클래스보다는 람다를 사용하라](7장_람다와_스트림/item42.md) | | [Lin][lin] |
| 아이템 43 | [람다보다는 메서드 참조를 사용하라](7장_람다와_스트림/item43.md) | | [Jason][jason] |
| 아이템 44 | [표준 함수형 인터페이스를 사용하라](7장_람다와_스트림/item44.md) | | [Jason][jason] |
| 아이템 45 | [스트림은 주의해서 사용하라]() | | |
| 아이템 46 | [스트림에서는 부작용 없는 함수를 사용하라](7장_람다와_스트림/item46.md) | | [Jason][jason] |
| 아이템 47 | [반환 타입으로는 스트림보다 컬렉션이 낫다]() | | |
| 아이템 48 | [스트림 병렬화는 주의해서 적용하라]() | | |

### 8장 메서드

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 49 | [매개변수가 유효한지 검사하라]() | | [delma][delma] |
| 아이템 50 | [적시에 방어적 복사본을 만들라]() | | |
| 아이템 51 | [메서드 시그니처를 신중히 설계하라](8장_메서드/item51.md) | | [Jason][jason] |
| 아이템 52 | [다중정의는 신중히 사용하라]() |  |  |
| 아이템 53 | [가변인수는 신중히 사용하라]() | Swift의 Variadic Parameters에 대해 소개합니다! | [Lena][lena] |
| 아이템 54 | [null이 아닌, 빈 컬렉션이나 배열을 반환하라]() | | |
| 아이템 55 | [옵셔널 반환은 신중히 하라](8장_메서드/item55.md) | Optional + nil | [Jason][jason] |
| 아이템 56 | [공개된 API 요소에는 항상 문서화 주석을 작성하라]() | Markup Formatting References in Xcode | [delma][delma] |

### 9장 일반적인 프로그래밍 원칙

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 57 | [지역변수의 범위를 최소화하라]() | | |
| 아이템 58 | [전통적인 for 문보다는 for-each 문을 사용하라]() | | |
| 아이템 59 | [라이브러리를 익히고 사용하라]() | 표준 라이브러리 사용 우선 고려하라 | [Lena][lena] |
| 아이템 60 | [정확한 답이 필요하다면 float와 double은 피하라](9장_일반적인_프로그래밍_원칙/item60.md) | | [Jason][jason] |
| 아이템 61 | [박싱된 기본 타입보다는 기본 타입을 사용하라]() | | |
| 아이템 62 | [다른 타입이 적절하다면 문자열 사용을 피하라]() | | |
| 아이템 63 | [문자열 연결은 느리니 주의하라]() | | [delma][delma] |
| 아이템 64 | [객체는 인터페이스를 사용해 참조하라]() | 객체는 상위 타입의 객체를 사용해 참조하라 | [delma][delma] |
| 아이템 65 | [리플렉션보다는 인터페이스를 사용하라]() | | |
| 아이템 66 | [네이티브 메서드는 신중히 사용하라]() | | |
| 아이템 67 | [최적화는 신중히 하라]() | | |
| 아이템 68 | [일반적으로 통용되는 명명 규칙을 따르라]() | | |

### 10장 예외

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 69 | [예외는 진짜 예외 상황에만 사용하라]() | | |
| 아이템 70 | [복구할 수 있는 상황에는 검사 예외를, 프로그래밍 오류에는 런타임 예외를 사용하라](10장_예외/item70.md) | |[Jason][jason] |
| 아이템 71 | [필요 없는 검사 예외 사용은 피하라](10장_예외/item71.md) | | [Jason][jason] |
| 아이템 72 | [표준 예외를 사용하라]() | | |
| 아이템 73 | [추상화 수준에 맞는 예외를 던지라](10장_예외/item73.md) | | [Jason][jason] |
| 아이템 74 | [메서드가 던지는 모든 예외를 문서화하라]() | | |
| 아이템 75 | [예외의 상세 메시지에 실패 관련 정보를 담으라]() | | |
| 아이템 76 | [가능한 한 실패 원자적으로 만들라]() | | |
| 아이템 77 | [예외를 무시하지 말라]() | | |

### 11장 동시성

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 78 | [공유 중인 가변 데이터는 동기화해 사용하라]() | | |
| 아이템 79 | [과도한 동기화는 피하라]() | | |
| 아이템 80 | [스레드보다는 실행자, 태스크, 스트림을 애용하라]() | | |
| 아이템 81 | [wait와 notify보다는 동시성 유틸리티를 애용하라]() | | |
| 아이템 82 | [스레드 안전성 수준을 문서화하라]() | | |
| 아이템 83 | [지연 초기화는 신중히 사용하라]() | | |
| 아이템 84 | [프로그램의 동작을 스레드 스케줄러에 기대지 말라]() | | |

### 12장 직렬화

| 아이템 번호 | <center>타이틀</center> | <center>서브 타이틀</center> | 담당자 |
|:-----:|-------|-------|:------:|
| 아이템 85 | [자바 직렬화의 대안을 찾으라]() | | |
| 아이템 86 | [Serializable을 구현할지는 신중히 결정하라]() | | |
| 아이템 87 | [커스텀 직렬화 형태를 고려해보라]() | | |
| 아이템 88 | [readObject 메서드는 방어적으로 작성하라]() | | |
| 아이템 89 | [인스턴스 수를 통제해야 한다면 readResolve보다는 열거 타입을 사용하라]() | | |
| 아이템 90 | [직렬화된 인스턴스 대신 직렬화 프록시 사용을 검토하라]() | | |

## References

- [이펙티브 자바 3판 깃허브 저장소][effective-java-3e-source-code]
- [이펙티브 자바 3판 번역 용어 해설][terms]

[effective-java-3e-source-code]: https://github.com/WegraLee/effective-java-3e-source-code
[terms]: https://docs.google.com/document/d/1Nw-_FJKre9x7Uy6DZ0NuAFyYUCjBPCpINxqrP0JFuXk/edit
[delma]: https://github.com/delmaSong
[gwonii]: https://github.com/gwonii
[heidi]: https://github.com/seizze
[jason]: https://github.com/ehgud0670
[jinie]: https://github.com/idevjinie
[lena]: https://github.com/dev-lena
[lin]: https://github.com/Limwin94
