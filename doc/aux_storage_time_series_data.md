# TimeSeriesData

## 소개
- 시계열 데이터는 시간이 흐름에 따라 변하는 데이터를 요약해서 보여주는 것입니다.

- 요약은 어떤 시간 범위 안에서 일어난 사건들의 개수, 최소값, 최대값, 총합으로 구성됩니다.

- 데이터를 요약하는 시간 간격을 정의할 수 있습니다. 이것을 해상도라고 부릅니다.

- 하나의 시계열 데이터를 여러 해상도로 동시에 요약해서 저장할 수 있습니다.
  예를 들면, 같은 데이터에 대해 1분 요약, 1시간 요약, 1일 요약을 동시에 제공할 수 있습니다.

- 해상도별로 저장 기간을 다르게 설정할 수 있습니다.

- 대단히 많은 양의 데이터를 여러 서버에서 빠르게 써넣어도 안정적으로 동작합니다.

- 서버가 죽으면 데이터의 일부는 손실될 수 있습니다.

- 데이터를 (해상도, 시간 범위, 시계열 이름) 으로 얻어올 수 있습니다.

- 시계열 데이터는 하나의 버킷에 속합니다.

- 어떤 해상도가 있는지, 해상도별 저장 기간이 얼마인지는 버킷에 정의되어 있습니다.

- (특정 시점, 특정 해상도)에 대해서, 한 버킷에 들어있는 모든 시계열 데이터의 값을 한번에 얻을 수 있습니다.

## 용도
시간에 따라 변하는 대량의 데이터를 실시간으로 요약하는 데 씁니다.
엔진 내부에서는 프로파일링 데이터와 성능 지표를 저장하는 데 사용합니다.

## DB에 아카이빙
데이터의 양이 많기 때문에 Redis에 저장하기에 다소 부담스러울 수 있습니다.
Redis에서는 실시간 요약만 수행하고, 데이터를 DB에 아카이빙하도록 구현할 계획이 있습니다.

## 구현
Redis HASH와 SET 자료구조를 사용했습니다.
