# 파이버를 사용하는 유닛 테스트 작성

## BetterConsoleRunner 사용

테스트케이스에서 파이버를 만들었으면, **파이버가 모두 종료된 다음에 다음 테스트케이스로 넘어가도록 해야** 합니다.
그러지 않으면 테스트케이스끼리 간섭이 생겨서, 테스트 실행 순서가 바뀌거나 했을 때 테스트가 실패하게 되는 경우가 생길 수 있습니다.

실버바인 서버엔진 2는 `BetterConsoleRunner`라는 테스트 러너를 제공합니다.
이 테스트 러너는 테스트 메서드가 `FTvoid` 형식을 반환하는 경우 파이버를 만들고 파이버에서 테스트 케이스를 실행한 다음 파이버가 종료하도록 기다립니다.

```csharp

[Test]
public async FTvoid ExampleTest()
{
    …
}
```

## 테스트 시나리오에서 여러 파이버를 실행하기

하나의 테스트 시나리오에서 여러 파이버를 한 번에 실행하고 그 결과를 대기하는 경우가 있습니다.
파이버 성능 테스트를 위한 경우이거나 복잡한 행동을 유발하는 테스트 케이스가 그렇습니다.

이런 경우에는 `FiberTest` 클래스를 이용해서 여러 파이버를 동시에 실행할 수 있습니다.

```csharp

[Test]
public void PerformanceTest()
{
    var ft = new FiberTest(false);

    for (var i = 0; i < FIBER_COUNT; ++i)
    {
        ft.Begin($"#{i}", async () =>
        {
            …
        });
    }

    try
    {
        ft.Loop();
    }
    catch (AggregateException)
    {
        …
    }
}
```

`FiberTest.Begin` 메서드로 실행한 파이버들은 모두 동시에 실행됩니다.
또한, `FiberTest.Loop` 메서드는 해당 `FiberTest` 개체에서 시작된 파이버들이 모두 종료될 때까지 엔진 이벤트 처리 루프를 실행합니다.
이벤트 처리 루프가 메인 스레드에서만 실행 가능하고 파이버 안에서는 실행할 수 없기 때문에 `FiberTest`를 사용하는 테스트는 `FTvoid` 형식을 반환할 수 없습니다.
