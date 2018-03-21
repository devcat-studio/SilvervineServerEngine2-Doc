# 실버바인 서버엔진 2

## 네트워킹
 * 네트워킹 개요
 * [세션 파이버](doc/session_fiber.md)
 * 로그인 과정
 * 메시지 송수신
 * 메시지 핸들링
 * [테스트 클라이언트 만들기](doc/test_client.md)
 * [네트워크 지연시간 시뮬레이션](doc/latency_simulation.md)
 * [서버간 통신 - InterServerQueue](doc/inter_server_queue.md)
 * [서버간 통신 - 세션 핸들러에게 메시지 보내기](doc/send_game_logic_message_to_server.md)
 * [네트워킹 설정값 가이드](doc/networking_setting_guide.md)

## 파이버
 * [async/await](doc/async_await.md)
 * [실버바인 서버엔진의 파이버](doc/fiber_in_silvervine_server_engine.md)
 * [배경 작업 만들기](doc/background_job.md)
 * [파이버를 사용하는 유닛 테스트 작성](doc/fiber_unit_test.md)
 * [백그라운드 스레드를 파이버와 어울려 사용하기](doc/background_thread.md)
 * [인터럽트 브릿지](doc/interrupt_bridge.md)
 * [파이버의 성능](doc/fiber_performance.md)

## DB
 * [DB 개요](doc/db_outline.md)
 * [트랜잭션](doc/db_transaction.md)
 * [DB 내부 구현](doc/db_implementation.md)
 * [로깅](doc/db_logging.md)
 * [DB 설계 의도](doc/db_design.md)
 * [DB의 변화와 프로세스 메모리에 있는 객체의 변화를 트랜잭션으로 묶기](doc/db_memory_transaction.md)
 * [자동 생성된 Row 클래스를 공통 인터페이스로 묶기](doc/db_row_adapter.md)

## 보조 저장소
 * [보조 저장소 개요](doc/aux_storage_outline.md)
 * [KeyValueStorage](doc/aux_storage_key_value_storage.md)
 * [Cache](doc/aux_storage_cache.md)
 * [Queue](doc/aux_storage_queue.md)
 * [Ephemeral](doc/aux_storage_ephemeral.md)
 * [Ranking](doc/aux_storage_ranking.md)
 * [StaticIndexDeque](doc/aux_storage_static_index_deque.md)
 * [TimeSeriesData](doc/aux_storage_time_series_data.md)

## 빌드
 * [코드 생성기](doc/codegen.md)
 * [컴파일러 경고 무시](doc/compiler_warning_ignore.md)

## 가상 액터
 * 개념
 * 사용법

## 웹서버
 * Http 서버 열기
 * 웹소켓 핸들링하기

## 스크립팅
 * [서버에서 루아 스크립트 사용하기](doc/lua_scripting.md)

## 런칭 준비
 * 개발서버에 올리기

## 기타
 * [세션 타이머](/doc/session_timer.md)
