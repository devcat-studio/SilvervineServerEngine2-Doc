# 세션 파이버

## 개요
클라이언트가 서버에 접속해서 세션이 만들어지면, 엔진이 자동으로 세션마다 하나씩 파이버를 만들어 줍니다.

게임 로직에서 `EngineAPI.Fibering.Begin`을 직접 호출하는 것이 아니기 때문에
파이버로 동작한다는 것을 의식하기는 어렵지만,
해당 세션과 관련된 모든 사건은 세션 파이버 안에서 순서대로 처리됩니다.
즉, 핸들러의 메서드가 앞선 사건의 처리를 끝내고 리턴해야만 다음 사건의 처리가 시작됩니다.

## 세션 파이버와 관련된 동작 순서
간단히 요약하면 다음과 같습니다.

1. 클라이언트가 서버에 접속하여 세션이 새로 생성되었습니다.

2. 세션 파이버가 만들어집니다.

3. 세션 파이버 안에서 `ILoginProcessor.TryLogin`이 불립니다.
이 때 여러명의 로그인 처리가 하나의 `ILoginProcessor` 객체에서 동시에 실행될 수 있습니다.
오작동을 막기 위해서, `ILoginProcessor`를 구현한 클래스에서는 변할 수 있는(mutable) 필드를 쓰지 않기를 권장합니다.

4. `ILoginProcessor.TryLogin`이 새 세션을 위한 세션 핸들러 객체를 리턴합니다.

5. (여러번 반복) 이 세션에서 발생하는 사건을 세션 핸들러 객체에서 처리합니다.

6. 세션이 파괴되어, `SessionDestroyed` 메서드가 호출됩니다.

7. `SessionDestroyed` 메서드가 리턴하면 드디어 세션 파이버도 종료됩니다.

## 세션 파이버에서 실행될 동작을 바깥에서 등록하기
`EngineAPI.Networking.SessionHandler.QueueToLocalSession` 을 하세요.
