# 컴파일러 경고 무시

## CS1998

CodeGen이 CS1998 경고가 발생하는 코드를 생성하는 경우가 있습니다.

> warning CS1998: This async method lacks 'await' operators and will run synchronously. Consider using the 'await' operator to await non-blocking API calls, or 'await Task.Run(...)' to do CPU-bound work on a background thread.

실버바인 서버 엔진 2를 써서 개발하다 보면 이 워닝을 심심찮게 만나게 됩니다.
주로, '파이버를 대기시킬 수 있는 메서드인데 현재 구현에서는 대기하는 곳이 없는' 곳에서 발생합니다.

CS1998 경고에 대한 정책을 확립하기 전에는 이 경고를 만나면
의미없는 `await EngineAPI.Fibering.Yield();` 를 넣거나,
메서드 시그니처에서 async를 지우고 리턴값에 `Task.FromResult` 를 씌우는 식으로 우회했었습니다.

이제는 로직 코드에서 CS1998 경고를 통째로 무시할 것을 권장합니다.

프로젝트 설정에서 `빌드` > `경고 표시 안함` 에 `CS1998`을 입력해주시면 됩니다.
