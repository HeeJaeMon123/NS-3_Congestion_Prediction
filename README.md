# NS-3_Congestion_Prediction
### .cc 파일은 data 생성을 위한 ns3 관련 파일입니다. 해당 파일은 ns3가 설치되어있어야 실행되니 데이터 파일을 올려두었습니다.
+ 프로젝트 요약

  + ns3를 이용한 네트워크 혼잡 예측

+ 프로젝트 개요

  + 네트워크 환경에서는 혼잡(congestion) 현상으로 인한 packet drop이 빈번하게 발생합니다. ns3(network simulator)는 말 그대로 실제환경은 아니고 네트워크를 시뮬레이션하는 시뮬레이터입니다. 이러한 ns3를 이용하여 혼잡을 일부러 발생시켜 packet drop을 발생시키고 관련 feature들을 모아서 머신러닝프로젝트를 진행하려고 합니다. 

  + 정상, 혼잡 이진분류입니다.

  + 관련 feature들로는 tcp reno에서의 congestion window size, RTT(Round Trip Time)가 있습니다. 

   ![3](https://user-images.githubusercontent.com/66413753/146819320-0fd70887-5b33-4145-ad6f-f1f815e01b12.PNG)


  + 저는 네트워크와 기계학습(딥러닝 포함)을 엮어서 연구를 하려고 하고 있기 때문에 저만의 프로젝트를 시작해보고 싶어서 이 주제를 선정하게 되었습니다.

    

+ 데이터 생성

  + ns3 오픈소스를 활용하여 데이터를 생성했습니다.  
  + 아래 사진은 시간에 따른 congestion window의 크기입니다. congestion window가 2개(오른쪽 2개)출력 된 것을 볼 수 있는데 이는 전 시간대의 congestion window와 현재 시간대의 congestion window 크기를 나타내고 모델에 사용한 데이터는 현재 시간대의 congestion window 크기만을 사용했습니다.
  + RxDrop이라고 써있는 곳은 Packet drop이 일어났다는 것입니다. RxDrop at 1.13696 밑의 2줄을 보면 congestion window가 14472에서 1040으로 변한 것을 볼 수 있습니다. 이는 TCP Reno의 특징입니다. 이때가 혼잡 상태이므로 제가 일일이 labeling을 진행했습니다. 데이터는 약 1000개입니다. 또한 train과 test를 나눌 때는 ** stratify split **을 사용했습니다.
  + label 0은 정상, label 1은 혼잡상태 입니다.
  ![2](https://user-images.githubusercontent.com/66413753/146819359-2440e359-6eb0-449a-a518-409be794da22.PNG)

  + 아래 사진은 RTT입니다. ns 단위로 출력되기 때문에 apply를 이용하여 전처리 해주었습니다.

  
  ![5](https://user-images.githubusercontent.com/66413753/146819380-70d1afff-8334-4c91-9400-f87f123ed093.PNG)

  ![6](https://user-images.githubusercontent.com/66413753/146819395-8cfdbdf1-2637-4421-9c90-bb5773809580.PNG)


 
​								

+ 모델 생성

  + 데이터 생성 후 모델 여러 개를 만들어서 각각 예측을 진행하고 아래 사진과 같이 metric들과 결정 경계를 출력하도록 했습니다. (metric을 출력할 때는 test data를 이용했습니다.)
  + 분류문제이기 때문에 정확도 외에 f1-score나 confusion matric도 출력했습니다. 

  ![4](https://user-images.githubusercontent.com/66413753/146819436-6288de3c-19a8-4e46-9236-cfa86bd3e654.PNG)


+ 모델 결과

  + tree나 linear kernel을 이용한 SVC모델을 제외하곤 만족스러운 결과가 도출되지 않았습니다. 예를 들어 LinearSVC 모델은 정확도는 굉장히 좋아보이지만 f1-score는 0으로 거의 예측을 하지 못하는 수준이었습니다. 이는 다시한번 정확도는 분류에서 좋은 metric이 아니라는 것을 말해줍니다.

    ![7](https://user-images.githubusercontent.com/66413753/146819456-a91cdf3a-1f61-40d7-8da8-f2834795e485.PNG)


  + 반면 linear kernel을 사용하면 굉장히 예측을 잘했습니다. (사진에서 edgecolor가 다른 데이터가 있는 이유는 train_data이기 때문입니다.)

    
    ![4](https://user-images.githubusercontent.com/66413753/146819466-7c47db66-5bb0-4cea-a6d7-144ff2909d8e.PNG)

    

  + 프로젝트를 시작하기 전에 저는 트리 계열은 거의 예측 정확도가 100%라고 예측했습니다. (cross entropy로 계산하기 때문에) 결과는 매우 좋았습니다. 하지만 결정 경계를 그려보면 굉장히 overfit이 심한 것을 볼 수 있습니다. 이는 트리 계열 알고리즘이 overfit하기 굉장히 쉽다는 것을 상기시켜줍니다.

     ![8](https://user-images.githubusercontent.com/66413753/146819477-2eb6378e-be22-4534-a5a7-4aaf187c2b92.PNG)


  + precision, recall값 바꿔보기

    + 혼잡 상황이 아닌데 혼잡이라고 예측하는 것은 그렇게 큰 문제가 되지 않습니다. 하지만 혼잡 상황인데 혼잡이 아니라고 예측해버리면 더 많은 packet들이 drop되는 결과를 초래할 것입니다.

    + 이 상황을 가정하여 임계값을 바꿔가며 confusion matrix를 확인해보려고 합니다. 조금 극단적으로 가정했습니다.

    + 모델은 polynomial kernel을 사용하는 SVC입니다.

      ![9](https://user-images.githubusercontent.com/66413753/146819486-593f636b-af9f-4f64-ae0e-b11145a91eeb.PNG)


    + polynomial kernel을 사용하는 SVC 모델은 test 결과가 매우 좋지 않은 것으로 보입니다. confusion matrix를 보면 모두 정상이라고 예측했습니다. 이는 혼잡 상황인데 혼잡이 아니라고 예측해버리면 더 많은 packet들이 drop되는 결과를 초래할 것입니다. 

    + 하지만 아래 사진과 같이 (상황을 가정하기 위해 극단적으로) 임계값을 0으로 바꾸면 confusion matrix가 모두 혼잡 상황이라고 예측하게 됩니다. 실용적으로는 이 상황에서 혼잡 상태를 정상으로 바꾸기 위해 network resource를 많이 쓰게 되기 때문에 좋지 않습니다.

      ![10](https://user-images.githubusercontent.com/66413753/146819494-dcd40e59-b614-40d1-a1a6-d52ae2354673.PNG)


    + 따라서 만약 자기가 생성한 network의 resource가 차고 넘쳐서 상관없으면 임계값을 0에 가깝게 설정하면 되고 resource가 부족하다면 임계값을 1에 가깝게 설정하면 됩니다.

  

  + 프로젝트를 진행하면서 느낀점
    + 이번 프로젝트는 저에게 조금 특별합니다. 왜냐하면 제가 연구하고자 하는 통신분야와 머신러닝 분야를 엮어서 저 혼자 진행한 첫 프로젝트이기 때문입니다.  또한 머신러닝관련 프로젝트를 하면 kaggle에 있는 정형화된 데이터를 사용하거나 이미 주어진 데이터를 많이 사용했습니다. 하지만 이번에 처음으로 데이터 생성부터 완전 처음부터 시도를 해봤기 때문에 뜻깊은 것 같습니다.

  
