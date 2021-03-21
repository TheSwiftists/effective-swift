# item74 메서드가 던지는 모든 예외를 문서화하라

이번 아이템에서는 각 메서드가 던지는 예외를 문서화하는데 충분한 시간을 쏟아야 한다고 설명하고 있습니다.

item56을 참고하면 Swift의 경우 `Parameters & Retrun Values` 항목에서 주석 중`Throws`를 작성해 발생할 수 있는 에러에 대해 문서화를 할 수 있도록 기본 제공하고 있습니다.

```swift
/**
 과일을 받아 해당 과일로 쥬스를 만듭니다.
 
 - Parameter fruit: 쥬스로 만들 과일
 
 - Throws: 쥬스를 만들 수 없는 과일이 들어올 경우 'BlendedError.unusableFruit'
 */
func makeJuice(using fruit: Fruit) throws -> Juice {
    // 만들 수 없는 재료일 경우
    if fruit.name == "Durian" {
        throw BlendedError.unusableFruit
    }
    
    // make juice code
    
    return Juice(fruit: fruit)
}
```

<br>

이렇게 주석으로 문서화를 진행해놓을 경우, `makJuice()`를 호출하는 사용자는 해당 내용을 보고 어떤 경우에 어떤 에러가 발생되는지 쉽게 파악할 수 있게 될 것입니다.

<img width= "100%" src="https://i.imgur.com/1sNI8yt.png"></img>

<br>

또한 `public` 메서드의 경우 필요한 전제조건을 문서화해서 남겨 놓아야 하고,
한 클래스에 정의된 많은 메서드가 같은 이유로 같은 예외를 던진다면 그 **예외를 클래스 설명에 추가하는 방법**도 고려해보라고 이야기하고 있습니다.

클래스에 대한 주석 또한 위와 같은 방법으로 작성할 수 있으므로 사용자가 더욱 효과적으로 클래스를 사용할 수 있겠습니다.
