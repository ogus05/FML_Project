# 2023 - Fundumental Machine Learning Project.
# With Logistic Regression & Validity Period.

## Study plan.
| 계획 | 완료 날짜 |
| - | - |
| Python module을 cpp program에 import하는 법 익히기| 23/11/05 |
| YCSB 통해 training / test data set 뽑기 | 23/11/05 |
| 적절한 ML algorithm 찾기 | 23/11/06 |
| ML을 적용하지 않은 validity period를 출력하여 target class 생각하기 | ONGOING |
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
1. 위 프로젝트는 workload dependent한 점이 hot-cold separation과 궤를 같이한다. 따라서 특정 workload를 지정해야한다.  
	위 실험은 garbage collection의 성능을 향상시키는 것을 목적으로 한다. 따라서 garbage collection에 직접적인 영향이 있는 100% update request workload만을 사용한다.  
	(training - 10M client requests / test - 5M client reuqests).    
2. [YCSB](https://github.com/brianfrankcooper/YCSB)가 mongoDB에 수행하는 process는 data를 DB에 loading 하는 load phase / DB에 loading한 data에 목적을 갖는 request를 전달하는 run phase로 나뉜다. 
3. Load phase의 block IO trace는 사용하지 않고, 각각 10M개, 5M개의 update를 갖는 run phase를 수행했을 때 mongoDB server에서 발생하는 block IO를 출력하여 사용한다.  
4. Input feature는 다음과 같이 사용한다. (process / request size / logical page address(LPA))

## Design.
### 1. Measurement method of Validity Period.
Flash memory physical page에 실제 write된 시점부터 update에 의해 invalid check된 시점까지를 측정한다.  

Valid status 도중 erase로 인해 physical page가 변경이 되어도 validity period에는 영향을 미치지 않는다.

### 2. Applying plan.
2-1. Block의 수는 제한적이기 때문에 validity period를 측정한 이후 일정 기간(time unit of classification)을 기준으로 input feature별로 classification한다.  

2-2. 새로운 update transaction이 flash memory operation을 필요로 할 때, 어떤 block의 어떤 page에 새로 write해야할지를 결정해야 한다.  

2-3. 이 때 classification information을 사용해서 block을 결정한다.  

2-4. 먼저, ML을 적용하지 않았을 때 simulator를 실행한 다음 validity period를 출력하고 이를 분석하고 target class들을 만든다.  

2-5. Target class 별로 y값을 구분하여 multi-class classification을 통해 logistic regression을 수행한다.  

2-6. epoch와 learning rate를 변경해가며 학습해 본다.  

### 3. Core properties in script sources.
3-1. Process는 기존 ASCII block IO format에 존재하지 않는다. 나는 이를 읽은 다음 host interface layer의 request segmentation to transaction에서 각 LPA - process mapping infromation을 따로 저장한다.  
이후 transaction의 validty period를 측정할 때나 LPA를 PPA로 변환할 때 LPA를 사용해서 process를 찾아 이를 parameter로 반영한다.  
3-2. Call python method script는 class로 따로 생성하여 사용한다. Process와 LPA의 mapping information또한 해당 class에 저장한다.
