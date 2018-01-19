# 서버간 통신 - 세션 핸들러에게 메시지 보내기

### 개요
세션 핸들러를 지정해서 메시지를 송신할 수 있습니다.
메시지를 수신할 세션 핸들러가 다른 프로세스에 있어도 똑같이 동작합니다.

### 사용법
#### 수신자 직접 지정
메시지를 수신할 프로세스 id와 세션 id를 직접 지정해서 보낼 수 있습니다.
그런데, 프로세스 id와 세션 id는 모두 엔진이 발급하는 값이므로 이 API를 직접 쓰기에는 번거로울 것입니다.
수신자 간접 지정 방식을 사용하실 것을 권장합니다.

#### 수신자 간접 지정
세션이 자신의 세션 키를 등록하고, 송신 측에서 세션 키를 상대로 메시지를 보낼 수 있습니다.

여러 세션이 같은 세션 키를 등록할 수 없으므로, 이것을 중복 로그인을 막는 장치로 활용하셔도 좋습니다. 새로 접속한 세션이 이전 세션을 파괴하게 하려면 대략 다음과 같은 절차를 거치시면 됩니다.

```csharp
string mySessionKey = "blahblah";
bool registered = false;
for (int i = 0; i < 10; ++i)
{
    try
    {
        await EngineAPI.Networking.SessionHandler.RegisterSessionKey(mySessionKey);
        registered = true;
        break;
    }
    catch (SessionKeyIsAlreadyRegisteredException e)
    {
        // 중복 로그인이니 이전 세션에게 자신을 파괴해달라고 요청한다.
        // 세션 핸들러에서는 여기서 보내는 요청을 수신하면 스스로를 파괴하도록 만들어야 한다.
        byte[] apoptosisRequest = ... // 각 프로젝트가 사용하는 메시징 포맷에 따라 다름
        await EngineAPI.Networking.SessionHandler.SendGameLogicMessageToServer(
            mySessionKey, apoptosisRequest);
        await EngineAPI.Fibering.Sleep(TimeSpan.FromSeconds(1));
    }
}
if (!registered)
{
    // 어째서인지 모르겠지만 이전 세션이 정리되지 않는 상태.
    // 프로세스에 너무 심한 부하가 걸려서 요청이 매우 늦게 처리되는 상황일 수 있다.
    // 에러를 보고하고, 클라이언트에게 로그인 불가를 통지하고, 이 세션을 파괴한다.
    ER.Warning("세션 파괴 요청이 실행되지 않았습니다.", new { sessionKey = mySessionKey });
}
```

### 구현
세션 키 등록에 `EngineAPI.Ephemeral`, 메시지 송신에 `EngineAPI.InterServerQueue`를 사용합니다.