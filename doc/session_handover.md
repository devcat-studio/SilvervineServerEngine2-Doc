### 세션 재연결
#### 개요
모바일 환경에서는 게임 클라이언트를 사용하는 동안 TCP 연결을 계속 유지하고 있기 어렵습니다. 클라이언트가 WIFI에서 LTE로 갈아타는 경우나 앱이 백그라운드로 내려갔다가 올라오는 경우에 TCP 연결이 끊길 수 있습니다.

EngineAPI.Networking을 사용하면 TCP 연결이 끊기는 경우에 자동으로 재접속합니다. 보낸 메시지를 잠시 보관해 두었다가 재접속 후에 자동으로 다시 보냅니다. 보관했던 메시지를 버리는 것은 상대편에서 메시지를 수신한 것이 확인된 시점입니다.

#### 클라이언트 작성 방법
클라이언트 라이브러리 입장에서는 서버가 완전히 다운된 경우와 네트워크가 일시적으로 불안정한 경우를 구분할 수 없기 때문에, 사용자가 중간에 개입하는 실행 흐름을 만들어야 합니다. IConnectionCallback을 구현한 클래스에서 다음과 같이 처리하기를 권장합니다.
1. SessionRepairStarted()가 호출되었을 때는 현재 통신이 끊어졌으며, 복구를 하고 있다는 것을 사용자에게 알려주면 좋습니다. 사용자의 행동을 요구하고 있는 것은 아니므로 크게 눈에 띌 필요는 없습니다. 심지어 아무런 표시를 하지 않아도 상관없습니다.
2. SessionRepairFailed()가 호출되었을 때는 끊어진 통신을 복구하려고 클라이언트 라이브러리가 여러번 시도했지만 아직 성공하지 못했음을 나타냅니다. 현재 네트워크 상황이 좋지 않으니 재시도하겠느냐는 의사를 묻는 UI를 띄워주십시오. 이 UI에서 사용자가 확인 버튼을 누르면 Connection.RetrySessionRepair()를 호출하십시오.
3. SessionRepairSucceeded()가 호출되었을 때는 끊어진 통신이 완전히 복원되었음을 의미합니다. SessionRepairStarted()에서 표시했던 알림을 제거하는 정도로만 해두면 됩니다.
4. KickedByServer()가 호출되었을 때는 현재 세션(Connection 객체)을 복원하는 것이 확실하게 불가능하다는 뜻입니다. 게임을 재시작하는 것을 권장합니다.

주의할 점: SessionRepairFailed가 호출되었을 때 유저에게 묻지 않고 바로 RetrySessionRepair를 호출하면 절대로 안됩니다! 서버가 다소 느려졌을 때 클라이언트들이 아주 빠르게 세션 복구를 재시도하면서 서버를 DDOS해서 죽이게 됩니다.

#### 로드밸런서 문제
게임 서버를 AWS ELB(Elastic Load Balancer)와 같은 로드밸런서 뒤에 배치할 경우, 클라이언트가 재접속할 때 아까와 같은 서버 프로세스에 접속할 수 있다는 보장이 없습니다. 게임 서버를 로드밸런서 뒤에 놓으면서 세션 재연결을 활용하고 싶은 경우에는, 로드밸런서를 통해서 게임 서버에 접속할 수 있는 동시에 게임 서버마다 공용 IP를 하나씩 할당하기를 권장합니다. 그러면 클라이언트가 서버에 처음으로 접속할 때는 로드밸런서를 통해 접속하고, 끊겼다가 다시 접속할 때는 아까 접속했던 서버의 공용 IP로 접속합니다.

### 세션 핸드오버
#### 개요
클라이언트가 재접속할 때 방금 전 접속했던 바로 그 서버 프로세스가 아니더라도, 새로 접속한 프로세스로 세션을 넘겨받아서 게임을 계속할 수 있습니다. 그 과정에서 클라이언트와 서버가 서로 주고받는 메시지는 손실되지 않으며, 클라이언트 게임 로직은 아무것도 신경쓸 필요가 없습니다. 수 초 정도 통신이 늦어지는 것처럼 보일 뿐입니다.

#### 세션 재연결 대비 장점
- 게임 서버에 공용 IP를 할당하지 않으면서 로드밸런서 뒤에 놓을 수 있습니다.
- 게임 경험에 영향을 주지 않으면서 게임 서버를 끄고 업데이트할 수 있습니다. 게임 서버를 끌 때 그 서버에 연결된 세션들이 모두 직렬화되어 redis에 임시 저장되고, 클라이언트가 스스로 재접속하면서 다른 프로세스에 복원될 것입니다.

#### 사용하는 법
NetworkingListenSetting.HandoverSetting을 설정해주세요. 이것이 null이면 핸드오버 기능이 꺼집니다.

- 세션 핸들러의 `SessionDestroyed()`에서 인자 `swapOut`이 true로 들어오면, 세션 핸들러를 `byte[]`로 직렬화해서 리턴하도록 합니다.
- `byte[]`로부터 세션 핸들러를 복원하는 코드(ISessionHandoverProcessor)를 만들어서 `EngineAPI.Networking.Listen`에 넘기는 `NetworkingListenSetting` 안에 들어있는 `SessionHandoverSetting` 에 설정합니다.

#### 제약
세션 핸들러가 안전하게 다른 서버 프로세스로 이동할 수 있으려면, 세션이 프로세스 안의 다른 부분과 별다른 관계를 맺고 있지 않아야 합니다.
- 세션에서 하는 일의 거의 대부분이 DB 입출력인 퍼즐 게임에 세션 핸드오버를 적용하기 좋습니다.
- 다른 서버 프로세스로 이동하면 현재의 플레이 맥락이 깨지는 MMORPG 게임에는 세션 핸드오버를 적용할 수 없습니다.
- 턴제 대전 게임은 세션 핸드오버를 적용할 수 있도록 만들 수 있습니다. 게임 로직을 VActor 안에서 돌리고, 아래의 '서버간 메시지를 안전하게 처리하기' 를 신경쓰면 됩니다.

#### 서버간 메시지를 안전하게 처리하기
서버에서 세션을 상대로 발송하는 메시지는 핸드오버 과정에서 손실될 수 있습니다. `EngineAPI.Networking.SessionHandler`의 `SendGameLogicMessageToServer`, `QueueToLocalSession` 이 여기 해당됩니다. 손실되면 안되는 메시지는 다음 과정을 거쳐 발송하십시오.
- `EngineAPI.Queue`에 메시지 내용을 먼저 저장합니다.
  - 세션 핸드오버가 일어날 때 세션 id가 바뀌지 않으므로, 큐 이름에는 수신자의 세션 id를 사용하면 됩니다.
  - 세션이 살아있을 수 있는 최대 시간 + a로 expire 시간을 설정합니다.
- `EngineAPI.Networking.SessionHandler.SendGameLogicMessageToServer`로 메시지를 확인하라고만 보냅니다.
- 위 메시지를 수신한 시점, 그리고 세션 핸드오버를 마친 시점에 `EngineAPI.Queue.DequeueAll`로 메시지를 모두 가져와서 하나씩 처리합니다.

#### FAQ
##### SessionDestroyed()의 인자 swapOutSession은 어떤 경우에 false로 들어오나?
- 명시적으로 세션 파괴를 명령했을 때
- 다른 (아마도 새로 만들어진) 세션에서 같은 세션키로 RegisterSession했을 때
- 세션 핸드오버를 사용하지 않는 옵션으로 Listen했을 때

##### DestroySession에서 null을 리턴하면 어떻게 되나?
세션이 저장되지 않고 파괴됩니다. 세션의 파괴를 멈출 수는 없습니다.

##### 세션 키란 무엇인가?
'이 세션을 통해 누가 접속했는가' 를 표현합니다. 세션 id는 매번 세션이 생겨날 때마다 새로 발급되지만, 세션 키는 사용자를 대표합니다. 대체로 사용자 id(사용자의 document id)를 그대로 쓰면 됩니다.

세션 키는 EngineAPI.Networking.SendGameLogicMessageToServer를 통해 다른 서버에 있는 세션을 대상으로 메시지를 보낼 때도 사용됩니다.

##### 세션키가 묶이지 않은 세션도 있을 수 있나?
세션 핸들러가 처음 생성되었을 때는 아직 여기로 접속한 사람이 누구인지를 모르는 상태이므로, 세션 키가 없는 상태로 초기화됩니다. 세션 키가 없는 세션은 스왑아웃되지 않고, 따라서 핸드오버도 되지 않습니다.

RegisterSessionKey했을 때 비로소 그 세션과 세션 키가 연결됩니다.

##### 세션 핸드오버를 거치지 않고 같은 계정으로 다시 로그인하면 이전 세션은 어떻게 파괴되나?
`EngineAPI.Networking.CurrentSession.RegisterSessionKey`를 호출할 때, 엔진이 자동으로, 해당 세션키를 이전에 점유하고 있었던 세션에게 세션 파괴 메시지를 보내서 파괴합니다.

그런데 이 과정은 협력적으로 일어납니다. 즉, 이전 점유 세션이 어딘가에 블록되어 있어서 파괴 명령에 응답을 하지 않거나, 혹은 이전 점유 세션이 있는 프로세스가 죽진 않은 채로 정상동작하지 않고 멈춰 있으면, RegisterSessionKey가 여러번 시도해보고 결국 실패를 리턴합니다.

#### BeginDestroySession하고 나서 클라이언트로부터 도착한 메시지에 대해 어떻게 처리해야 하나?
세션을 파괴하려고 했던 이유가 무엇이었냐에 따라 다르겠지요.

#### SocketDisconnected 이벤트에 어떻게 대응해야 하나?
세션 핸드오버를 사용하는 게임에서는 이 시점에 `EngineAPI.Networking.BeginSwapOutSession(long sessionId)` 을 통해 강제로 스왑아웃을 시켜주는게 좋습니다.
