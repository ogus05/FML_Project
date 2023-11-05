# 2023 - Fundumental Machine Learning Project.

## Study plan.
| 계획 | 완료 날짜 |
| - | - |
| Python module을 cpp program에 import하는 법 익히기| 23/11/05 |
| YCSB 통해 training / test data set 뽑기 | 23/11/05 |
| Ridge와 Lasso의 특성 비교하면서 적절한 ML algorithm 찾기 | ONGOING |
| ML algorithm으로 구성된 python module 만들기 | PLANNING |
| 만든 python module을 MQSim에 import하기 | PLANNING |
| Output을 분석하면서 error 및 parameters related to ML algorithm과 적절한 time unit of classification 찾기 | PLANNING |
| 출력한 그래프를 통해 보고서 작성하기 | PLANNING |



## Motivation.
1. Flash memory의 free block을 만들기 위해서는 garbage collection processing이 필요하다.
2. Garbage collection processing 시 victim block에는 valid한 page가 포함되어 있을 수도 있다.
3. Victim block의 valid page data는 새로운 page에 writing을 필요로 한다.
4. 따라서 validity period를 예측하고, 비슷한 page들을 같은 block에 두어 valid page re-write overhead를 최소한으로 한다.

## Coding plan.
1. [MQSim simulator](https://github.com/CMU-SAFARI/MQSim)를 사용하여 SSD의 성능 개선을 파악한다.
2. Simulator는 c++로 구성되어 있기 때문에 machine learning 계산에 유용한 python module을 import하여 사용한다.
3. Input data는 [MongoDB](https://github.com/mongodb/mongo)를 활용하는 DB 서버에서 클라이언트의 요청을 처리할 때 발생하는 block IO를 [blktrace](https://github.com/efarrer/blktrace)를 통해 출력해서 사용한다.

## Detail of input data / features.
1. 이 계획은 workload dependent한 점이 hot-cold separation과 궤를 같이한다. 따라서 특정 workload를 지정해야한다.
	Read performance 변화의 가능성을 위해 50% read / 50% update request workload를 사용한다.  
	(training - 1M client requests / test - 1M client requests).
2. Page unit transaction 별로 validity period를 예측해야한다.  
	h(x, f(x)) - {x = logical page address(LPA), f(x) = validity period}

## Design.
### 1. Measurement method of Validity Period.
Flash memory physical page에 실제 write된 시점부터 update에 의해 invalid check된 시점까지를 측정한다.
Valid status 도중 erase로 인해 physical page가 변경이 되어도 validity period에는 영향을 미치지 않는다.

### 2. Applying technology.
2-1. Block의 수는 제한적이기 때문에 validity period를 측정한 이후 일정 기간(time unit of classification)을 기준으로 LPA별로 classification한다.  

2-2. 새로운 update transaction이 flash memory operation을 필요로 할 때, 어떤 block의 어떤 page에 새로 write해야할지를 결정해야 한다.  

2-3. 이 때 classification information을 사용해서 block을 결정해야 한다.  

2-4. 만약 이 때 ML을 위해 classification algorithm을 사용하게 되면 time unit of classification을 지정해야 한다. 만약 이 time unit이 매우 짧다면 hypothesis function에서 target region이 매우 많아진다.  

2-5. Target region이 매우 많아지게 되면 learning time이 매우 길어질 수 있다.  

2-6. 따라서 regression algorithm을 사용하여 learning을 수행한 다음 time unit을 지정하여 비슷한 validity period를 가진 LPA들을 한 class로 둔다.  

2-7. 한 class는 같은 block에 write되며, block에는 담고있는 page들의 classification information이 저장되어 있다.  

2-8. 각 LPA를 갖는 transaction들이 physical page out-of-place update를 필요로 할 때, 본인과 같은 classification information을 가진 block에 write된다.

### 3. Machine learning algorithm.
