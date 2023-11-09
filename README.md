# 2023 - Fundumental Machine Learning Project.
# Hot cold separation with logistic regression.

## Plan.
| 계획 | 완료 날짜 |
| - | - |
| YCSB를 통해 training / test data set 출력하기 | 23/11/05 |
| 적용 가능한 input features 생각하기 | ONGOING |
| Python을 통해 training / test data set의 access count와 함께 target value 출력하기 | PLANNING | 
| Python을 통해 logistic regression으로 학습 가능한 model 만들기 | PLANNING |
| Model의 학습을 반복하면서 적절한 weight 및 bias 찾기. | PLANNING |
| Simulator에 test data를 ML를 적용 했을 때와 안했을 때를 구분해서 실행하고 결과를 비교하기. | PLANNING |
| 출력한 그래프를 통해 보고서 작성하기 | PLANNING |


## Motivation.
SSD는 내부에 cache managing을 위한 RAM을 갖는다.  
Flash memory operation이 수행되는 것보다 RAM cache hit이 발생하게 되면 response time이 줄어들 뿐만 아니라 SSD의 수명도 늘어나기 때문에 cache hit ratio를 증가시키는 것이 매우 중요하다.  
하지만 cache hit ratio를 증가시키기 위해서는 cache에 이후 요청받을 data들을 저장해야 하는데, 이는 미래를 예측해야 하므로 불가능하다.  
따라서 나는 machine learning을 통해 request가 다수 발생하는 data들을 학습하고, 해당 data들을 RAM에 상주시키며 성능의 증가를 측정할 계획이다.  


## Detail of input data.
1. Input data는 [MongoDB](https://github.com/mongodb/mongo)를 사용하는 DB server에서 발생하는 block IO를 사용한다.
2. 이를 위해 [YCSB](https://github.com/brianfrankcooper/YCSB)를 통해 DB server에 request를 전달하고 DB server에서 발생하는 block IO를 [blktrace](https://github.com/efarrer/blktrace)를 통해 출력한다.
3. 출력한 trace들은 다음 feature들을 갖는다.  
Request time / Event type(read or write) / LBA in sectors / size in sectors / process ID.

## Detail of input features.
1. LBA of the request.
2. Size of the request.
3. Difference of time with current request and last request.
4. Write frequency in a specific period.


