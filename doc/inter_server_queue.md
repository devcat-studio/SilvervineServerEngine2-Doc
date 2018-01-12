# 서버간 통신

## EngineAPI.InterServerQueue

### 개요
다른 서버 프로세스에 메시지를 보낼 수 있습니다.

메시지를 수신할 서버 프로세스를 `EngineAPI.CurrentProcessUniqueId` 를 통해 얻는
고유 식별자를 통해 지정하기 때문에,
`EngineAPI.InterServerQueue`를 직접 사용해서 게임 로직을 만들기에는
번거롭고 복잡할 것입니다.

### 사용법
`Post`로 메시지를 보내면, 대상 프로세스에서 `AddHandler`로 설정했던 메시지 핸들러가 호출됩니다.

핸들러 메서드는 `Task`가 아니라 `void`를 리턴합니다. 따라서 핸들러 메서드 안에서 `await`할 수 없는데,
이것은 의도적인 것입니다. 메시지를 수신하는 루프가 프로세스별로 단 하나만 있기 때문에,
메시지 핸들러가 즉시 리턴하지 않고 오래 기다리면 `InterServerQueue`를 사용하는
모든 시스템의 시간당 메시지 처리량이 떨어지기 때문입니다.

핸들러 메서드 안에서 `await`가 필요하면,
메시지를 처리하는 파이버를 `EngineAPI.Fibering.BeginQueueAttachedFiber`를 통해 생성하고,
수신한 메시지를 이 파이버에 큐잉해서 처리하기를 권장합니다.

EngineAPI.InterServerQueue를 확장해서 서버간 통신을 구현한 예제가 
샘플 리파지터리의 `InterServerMessaging`에 있습니다.

### 구현
서버 프로세스별 큐가 하나의 Redis LIST입니다. 새 메시지가 들어왔는지 짧은 시간 간격으로 폴링합니다.