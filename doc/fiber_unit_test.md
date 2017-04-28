# 파이버를 사용하는 유닛 테스트 작성

테스트케이스에서 파이버를 만들었으면, **파이버가 모두 종료된 다음에 다음 테스트케이스로 넘어가도록 해야** 합니다.
그러지 않으면 테스트케이스끼리 간섭이 생겨서, 테스트 실행 순서가 바뀌거나 했을 때 테스트가 실패하게 되는 경우가 생길 수 있습니다.
 
테스트케이스에서 파이버를 사용하려면 FiberTest 클래스를 사용하면 됩니다.
 
사용 예제는 아래와 같습니다:
```
var ft = new FiberTest(false);
ft.Begin("ReadMinMaxLogId_1", ReadMinMaxLogId_1);
ft.Begin("ReadMinMaxLogId_2", ReadMinMaxLogId_2);
ft.Begin("ReadLogsBefore", ReadLogsBefore);
ft.Begin("ReadLogsAfter", ReadLogsAfter);
ft.Begin("ReadByStartTime", ReadByStartTime);
ft.Begin("ReadByEndTime", ReadByEndTime);
ft.Loop();
```

여기에서 Begin으로 실행한 파이버들은 모두 동시에 실행됩니다.  
이 파이버들이 모두 종료되어야 Loop 메서드가 리턴합니다.