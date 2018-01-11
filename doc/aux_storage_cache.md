# Cache

## 소개
[KeyValueStorage](doc_aux_storage_key_value_storage.md)와 비슷한 키-값 저장소인데,
캐시 용도로만 사용하도록 의도했습니다.

항목을 작성할 때 반드시 만료 시간을 설정해야 합니다.

## 구현
Redis SET/GET/DEL/EXPIRE 명령어를 사용했습니다.