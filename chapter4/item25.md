# Item 25. 톱레벨 클래스는 한 파일에 하나만 담으라

### Java에서의 톱레벨 클래스 선언

Java에서는 하나의 소스 파일에 톱레벨 클래스를 여러 개 선언할 수 있습니다. 다만, 이는 아무런 득이 없을 뿐더러 심각한 위험을 감수해야 하는 행위입니다. 한 클래스를 여러 가지로 정의할 수 있으며, 그 중 어느 것을 사용할지는 어느 소스 파일을 먼저 컴파일하느냐에 따라 달라지기 때문입니다. 컴파일러에 어떤 소스를 먼저 건네느냐에 따라 동작이 달라지는 문제가 생길 수 있습니다.

```swift
public class Main {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }
}

// 코드 25-1 두 클래스가 한 파일(Utensil.java)에 정의되었다. - 따라 하지 말 것! 
class Utensil {
    static final String NAME = "pan";
}

class Dessert {
    static final String NAME = "cake";
}

// 코드 25-2 두 클래스가 한 파일(Dessert.java)에 정의되었다. - 따라 하지 말 것!
class Utensil {
    static final String NAME = "pot";
}

class Dessert {
    static final String NAME = "pie";
}

// 코드 25-3 톱레벨 클래스들을 정적 멤버 클래스로 바꿔본 모습 
public class Test {
    public static void main(String[] args) {
        System.out.println(Utensil.NAME + Dessert.NAME);
    }

    private static class Utensil {
        static final String NAME = "pan";
    }

    private static class Dessert {
        static final String NAME = "cake";
    }
}
```



### Swift에서의 톱레벨 클래스 선언

Xcode에서 하나의 소스 파일에 톱레벨 클래스를 여러 개 선언할 수 있습니다. 하지만 Java와 달리 다른 파일에 있다고 하더라도 같은 이름의 톱레벨 클래스를 선언할 수 없습니다.  `Invalid redeclaration of 'className'`와 같은 오류 메세지가 보이며 클래스를 생성할 수 없도록 막아놓았기 때문에 같은 이름의 클래스를 선언하려고 할 수 없습니다.

