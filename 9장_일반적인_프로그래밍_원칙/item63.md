# item 63. 문자열 연결은 느리니 주의하라

문자열 연결 연산자(+)는 여러 문자열을 하나로 합쳐주는 편리한 수단입니다. 그런데 한 줄짜리 출력값이나 작고 크기가 고정된 객체의 문자열 표현을 만들 때라면 괜찮지만, 많은 양의 반복적인 사용을 하는 경우 성능 저하를 감내하기 어렵습니다. **문자열 연결 연산자로 문자열 n개를 잇는 시간은 n^2에 비례합니다.** 두 문자열을 연결할 경우 양쪽의 내용을 모두 복사해야 하므로 성능 저하는 피할 수 없는 결과입니다.



```swift
  func bench(function: () -> Void, number: Int) {
        let startTime = CFAbsoluteTimeGetCurrent()
        function()
        let processTime = CFAbsoluteTimeGetCurrent() - startTime
        print("process time\(number) = \(processTime)")
    }

    func test1() {
        var gonni = "gonni"
        let hi = "hi"
        for _ in 1...1000000 {
            gonni += hi
        }
    }

    func test2() {
        let gonni = "gonni"
        let hi = "hi"
        for _ in 1...1000000 {
            _ = gonni + hi
        }
    }

    func test3() {
        var gonni = "gonni"
        for _ in 1...1000000 {
            gonni.append("hi")
        }
    }

    func test4() {
        let gonni = "gonni"
        let hi = "hi"
        for _ in 1...1000000 {
            _ = "\(gonni)\(hi)"
        }
    }
```

1000만번 일 때

```text
2-3-4-1 순으로 빠름
process time1 = 3.922340989112854
process time2 = 3.3447110652923584
process time3 = 3.719417929649353
process time4 = 3.779484987258911
```

100만번일 때

```text
2-3-4-1 순으로 빠름
process time1 = 0.4163990020751953
process time2 = 0.34728097915649414
process time3 = 0.3788869380950928
process time4 = 0.39645493030548096
```



10만번 이하로 테스트 한 경우에는 빠르고 느린 순위가 일관되지 않음

