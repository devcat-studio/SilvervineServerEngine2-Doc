# 실버바인 서버엔진 2의 파이버

## 개념
실버바인 서버엔진 2에서는 스택풀 코루틴(stackful coroutine)을 폭넓게 사용합니다.
스택풀 코루틴에는 여러가지 다른 이름이 있는데, 실버바인 서버엔진 2에서는 **파이버**(`Fiber`)라고 부릅니다.

## 용도
실버바인 서버엔진 2에서는
로직 프로그래머에게 멀티스레드나 비동기 I/O 프로그래밍의 부담을 지우지 않으면서
서버 프로세스 내부의 공유 상태를 안전하게 조작하기 위한 목적으로 파이버를 사용합니다.
DB 접근 등 I/O 대기가 필요한 동작은 파이버를 통해서만 실행할 수 있습니다.

## 파이버 로컬 스토리지
스레드 로컬 스토리지와 비슷한 것을 파이버에서도 사용할 수 있습니다.
`Fiber.Current.GetLocalStorage<T>()`, `Fiber.Current.SetLocalStorage<T>()`를 참고하세요.

## 구현
.NET Framework는 스택풀 코루틴을 지원하지 않지만,
[async 및 await를 이용](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/async/)해서, **작업 종료를 기다릴 수 있는 비동기 작업**들을 잘 연결된 하나의 실행 흐름처럼 느껴지게 만들었습니다.
그리고 여러 파이버가 공유 상태를 안전하게 변경할 수 있도록 하기 위해, 모든 파이버는 **메인 스레드**에서 실행됩니다.

파이버로 실행되는 코드가 메인 스레드를 떠나지 않고 안전하게 실행되게 하기 위해서 지켜야 하는 규칙들이 있습니다.
규칙을 어기면 파이버 전환이 비정상적으로 일어나고, 그 이후의 엔진 호출에서 EngineErrorException이 발생합니다.
비정상적인 파이버 전환이 일어난 시점 **이후에** 예외가 발생하기 때문에, 무엇을 잘못했는지 찾아내기가 매우 어렵습니다.
반드시 아래의 주의사항을 이해하시고 코드를 주의깊게 작성하셔야 합니다.

## 실버바인 서버엔진 2의 비동기 작업 반환 형식

실버바인 서버엔진 2는 **파이버에서 실행되는 비동기 작업**으로 쓰일 반환 형식으로 다음과 같은 형식을 제공합니다.

- `FTvoid`: 파이버 안에서 작업 완료를 기다리고 값을 반환하지 않습니다.
- `FT<TResult>`: 파이버 안에서 작업 완료를 기다리고 값을 반환합니다.

C#이 기본적으로 제공하는 [비동기 메서드의 반환 형식](https://docs.microsoft.com/ko-kr/dotnet/csharp/programming-guide/concepts/async/async-return-types)으로는 다음과 같은 것들이 있지만 실버바인 서버엔진 2 안에서는 사용하지 않습니다.

- [Task\<TResult>](https://docs.microsoft.com/ko-kr/dotnet/api/system.threading.tasks.task-1): 작업 완료를 기다리고 값을 반환합니다.
- [Task](https://docs.microsoft.com/ko-kr/dotnet/api/system.threading.tasks.task): 작업 완료를 기다리고 값을 반환하지 않습니다.
- void: 작업의 완료를 기다리지 않고, 기다릴 수 없습니다.

### 반환된 `FTvoid`, `FT<TResult>` 작업은 반드시 `await`

어떤 메서드가 반환한 `FTvoid`, `FT<TResult>` 작업은 완료되기 전까지 일시 중단될 수 있고 `await` 식으로 완료를 기다려야 메서드 호출 전에 실행 중이던 파이버 안에서 계속 실행될 것이 보장됩니다.

```csharp

async FTVoid WaitForOneSecond()
{
    var task = EngineAPI.Fibering.Sleep(1.sec());
    // 메서드가 시작될 때의 파이버가 아닌 곳에서 실행된다.
    await task; // 작업을 기다린다.
    // 메서드가 시작될 때의 파이버에서 실행된다.
}
```

비동기 메서드의 모든 내용이 파이버 안에서 실행되도록

- 반환받은 `FTvoid`, `FT<TResult>` 작업을 바로 `await` 연산자로 기다리거나,
- `return` 문으로 제어를 호출 메서드에게 반환해

호출 메서드가 `FTvoid`, `FT<TResult>` 작업을 기다리도록 하십시오.

모든 `FTvoid`, `FT<TResult>`는 메인 스레드의 정해진 파이버에서 실행되므로 .NET의 `Task`처럼 여러 개의 작업을 병렬로 실행한 뒤 모두 완료되기를 기다리거나 하는 기능은 제공되지 않습니다.

#### `FTvoid`, `FT<TTesult>` 작업을 `await`하지 않았을 때

`await`으로 완료를 기다리지 않은 파이버 작업은 실행 시간에 엔진 경고를 일으킵니다.
`EngineAPI.Config.EnableFiberingStackTrace` 속성을 `true`로 켜면 파이버 작업을 만들 당시의 호출 스택이 경고에 포함돼 `await`을 빠트린 코드를 확인할 수 있습니다.

### `async void`를 쓰면 안 되는 이유

`async void`는 [`await` 연산자](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/await)를 사용할 수 없는 [이벤트 처리기에만 사용](https://docs.microsoft.com/ko-kr/dotnet/csharp/async#important-info-and-advice)해야 하는데 실버바인 서버엔진 2에서는 이벤트 패턴을 사용하지 않습니다.
`async void` 메서드를 호출하면 파이버 안에서 메서드가 계속 실행되는 것을 보장하지 못합니다.
`async void` 메서드에서 일어나는 작업의 완료는 기다릴 수 없고, 이 작업은 [스레드 풀](https://docs.microsoft.com/ko-kr/dotnet/standard/threading/the-managed-thread-pool)에서 작업이 완료될 수 있습니다.
소스 코드에 전체 검색으로 `async void`를 찾았을 때 하나라도 발견됐다면 **메인 스레드에서 실행되지 않아 공유 상태가 망가지는** 심각한 문제가 있다고 생각하셔도 됩니다.

### 파이버 밖에서 실행되는 메서드에서 `FTvoid`, `FT<TResult>` 호출 금지

[`async` 한정자](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/async)를 사용해 `Task`, `Task<TResult>` 작업을 반환하는 비동기 함수 안에서 `FTvoid`, `FT<TResult>`를 호출할 수 없습니다. 모든 파이버 작업은 메인 스레드의 정해진 파이버에서 실행돼야 하는데 `Task`, `Task<TResult>`는 파이버는 물론이고 메인 스레드에서 실행되고 있을 것을 보장하지 않습니다.

메인 스레드 밖에서 실행되는 메서드가 메인 스레드에서 어떤 작업을 실행하려고 할 때에는 `EngineAPI.EnqueueFromOtherThread`를 사용할 수 있고, 메인 스레드에서는 `EngineAPI.Fibering.Begin`으로 새 파이버를 시작할 수 있습니다.

```csharp

EngineAPI.EnqueueFromOtherThread(() => {
    EngineAPI.Fibering.Begin("파이버 이름", async () => {
        …
    });
});
```

### 파이버 안에서 실행되는 메서드 안에서 `Task`, `Task<TResult>`를 사용하는 방법

[`async` 한정자](https://docs.microsoft.com/ko-kr/dotnet/csharp/language-reference/keywords/async)를 사용해 `FTvoid`, `FT<TResult>`를 반환하는 비동기 메서드는 모두 정해진 파이버 안에서 실행돼야 합니다.
메서드 안에서 일시 중단되는 `await` 식이 `Task`, `Task<TResult>`를 기다릴 경우 `await`에 연속된 작업이 다른 스레드에서 실행될 수 있어 메인 스레드의 정해진 파이버에서 실행돼야한다는 규칙이 깨져 정상적으로 실행되지 않게 됩니다.

그런데, `Task`, `Task<TResult>` 작업을 반환하는 다른 .NET 라이브러리들을 실버바인 서버엔진 2와 함께 사용해야할 경우가 있습니다.
이 경우에는 작업이 끝나고 다시 파이버에서 실행되도록 만들어 놓은 ToMainThread 확장 메서드를 사용하면 `Task`를 `FTvoid`로, `Task<TResult>`를 `FT<TResult>`로 변환할 수 있습니다.

#### 주의가 필요한 라이브러리의 예

 * `…Async`:  
   ![AWS S3 SDK](../img/thirdparty_async_library_example.png)  
   이 예는 AWS S3 SDK에서 가져왔습니다.

   ```csharp

   {
       await PutObjectAsync(request, CancellationToken.None);
       // 다른 스레드에서 실행
   }
   ```

   과 같이 반환된 `Task<PutObjectResponse>` 작업을 await하면 기다린 다음 다른 스레드에서 연속 작업이 실행됩니다.
   이와 같은 문제를 해결하기 위해

   ```csharp

   {
       await PutObjectAsync(request, CancellationToken.None).ToMainThread();
       // 메인 스레드·일시 중단 전에 실행 중이던 파이버에서 실행
   }
   ```

   반환된 `Task<PutObjectResponse>`에 `ToMainThread()`확장 메서드를 호출해 변환된 `FT<PutObjectResponse>`를 `await` 하면 메인 스레드의 파이버에서 연속 작업이 실행됩니다.

### 세션핸들러에서는 null을 리턴 가능

`ISessionHandler` 인터페이스의 모든 메서드들은 `FTvoid`, `FT<TResult>` 형식을 반환합니다.
따라서 이 메서드들은 모두 파이버 안에서 실행됩니다.
세션핸들러의 메서드들은 엔진에서 특별하게 처리해 `null`이 반환될 경우 `await`하지 않고 바로 이어서 진행하고 있습니다.

모바일 게임이라면 클라이언트가 보낸 모든 요청에서 DB에 접근해야 하니까 언제나 실행 중지/재개가 필요하겠지만,

```csharp

async FTvoid ReceivedGameLogic(Payload payload)
{
    await …;
}
```

MMORPG라면 DB에 접근하는 요청보다는 메모리에서 처리가 끝나는 요청들이 더 많을 것입니다.

```csharp

FTvoid ReceivedGameLogic(Payload payload)
{
    …;
    return null;
}
```

불필요하게 파이버를 중지했다가 다시 시작하는 맥락 전환하는 비용이 걱정된다면 동기적으로 실행될 때 `async` 한정자를 사용한 비동기 메서드 대신, `null`을 반환하는 메서드로 구현해도 됩니다.
