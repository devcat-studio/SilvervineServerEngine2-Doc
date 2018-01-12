# 배경 작업 만들기

## 개요
서버 프로세스가 떠있는 동안 별다른 이벤트가 발생하지 않아도
일정한 시간 간격으로 어떤 코드가 실행되도록 만들고 싶을 때가 있습니다.
이것을 **배경 작업**이라고 부릅시다.

배경 작업을 구현하려면
`while (true)` 루프를 돌며 중간중간 `EngineAPI.Fibering.Sleep()`하는 파이버를 만들면 되지만,
이렇게만 만들면
코드에서 예외가 새어나가면 파이버가 종료되어 버리기 때문에
어느 순간 배경 작업이 중단되어 버리는 버그를 만들기 쉽습니다.

따라서 서버 프로세스가 떠있는 한 계속 실행되어야 하는 코드를 위해서는
파이버를 직접 만들지 말고 대신 `EngineAPI.BackgroundJob`을 쓰는 것이 좋습니다.

`EngineAPI.BackgroundJob`을 통해 만든 배경 작업은
예외가 밖으로 새어나가더라도 잠시 후 자동으로 다시 시작됩니다.

## 익명 함수로 배경 작업 만들기
`LambdaBackgroundJob`을 사용하시면 됩니다.
```
EngineAPI.BackgroundJob.Start(
    "배경 작업 이름",
    () => new SomeBackgroundJob(() => {
        ...
    }));
```

## 배경 작업 중지하기
배경 작업 내부에서 `BackgroundJobStopException`을 던지면
의도적으로 배경 작업을 종료하려고 하는 거라고 인식합니다.
따라서 배경 작업을 자동 재시작하지 않습니다.

## 프로세스 종료 시점에 배경 작업의 종료를 기다리기
`EngineAPI.BackgroundJob.Start`가 리턴했던 객체에 `WaitForFinished();` 를 호출하세요.
`EngineAPI.Destroy()`가 일단 불리고 나면 `EngineAPI.Fibering.Sleep()`에서 즉시
`SleepInterruptedException`이 발생합니다.

배경 작업을 자동으로 재시작하는 절차가 `EngineAPI.Destroy()` 수행중에는 비활성화되므로
배경 작업 파이버가 잘 종료됩니다.
그러고 나면 `WaitForFinished()`가 리턴됩니다.

배경 작업의 무한루프 어딘가에 `EngineAPI.Fibering.Sleep()`을 넣는 것을 잊지 마세요.
