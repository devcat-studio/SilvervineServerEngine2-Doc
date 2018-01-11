# Queue

## 소개
문자열로 된 큐 이름으로 식별되는 전역 큐들입니다. Enqueue, Dequeue가 제공됩니다.

항목이 없을 경우 블록되는 Dequeue는 아직 제공하고 있지 않습니다.

## 구현
Redis LIST 자료구조를 사용했습니다.