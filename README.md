# Biliard_Ball_Detection

YOLOv3을 이용한 당구공 탐지 프로그램입니다. PyTorch를 기반으로 짜여진 YOLOv3를 직접 제작한 데이터셋으로 학습시켜 당구공을 object detection하는 모델을 만들었습니다. <br> <br>

제작자 : 김민규(minkyu4506@gmail.com)

YOLOv3 구현코드 출처 : https://github.com/eriklindernoren/PyTorch-YOLOv3

**Note : 실행파일을 만드는 과정에서 생기는 버그로 인해 실행파일을 공유하지 못했습니다. 버그 해결 후 실행파일을 공유하도록 하겠습니다.** <br><br>

### 레포지토리 구성(2022.2.3 기준)


<br>

|파일명|설명|
|---|---|
|README.md|현재 읽고 계시는 파일입니다. 레포지토리에 대한 설명을 담고 있습니다.|
|util_codes.py|프로그램 작성 중에 유용했던 메소드들을 정리한 파이썬 파일입니다.|


<br>

## 0. 개발환경
<br>

### [1] 컴퓨터 
**OS** : Ubuntu 20.04.3 LTS <br>
**GPU** : RTX 3060 (vram : 12GB) <br>
**PyTorch** : 1.9.0 <br>
**CUDA** : 11.2 <br>

### [2] 그 외 장치
**Camera** : Microsoft® LifeCam Studio(TM)

<br>

### 개발, 실행 환경 구성

<img height="800" alt="스크린샷 2022-01-30 오전 1 02 44" src="https://user-images.githubusercontent.com/50979281/151799427-2e31be67-1df3-43b1-88aa-8b687fce1788.JPG">


위와 같이 당구대 위에 있는 천장을 뜯은 뒤 카메라를 설치하였습니다. 

<br>

## 1. 데이터셋 : 자체 제작 데이터셋
당구대를 천장에서 촬영 후 라벨링 작업을 수행하여 제작했습니다. 라벨링에 사용한 프로그램은 [labelme](https://github.com/wkentaro/labelme)입니다.



<h3></h3>             |  <h3></h3>  
:-------------------------:|:-------------------------:
![](https://user-images.githubusercontent.com/50979281/151668240-07632f54-ee64-4a10-b47c-dc769a622d8e.png) | ![](https://user-images.githubusercontent.com/50979281/151668281-ea971d14-925e-4beb-b1d0-a6bc8941cd4d.png)


<br>

labelme를 이용해 모든 이미지에 대한 라벨링을 수행한 뒤, [416 x 416] 크기에 맞춰 이미지와 라벨 데이터를 변경하였습니다. 

<br>

**[발전 과정]** 

데이터셋은 2번의 개선과정을 거쳐 3번째 데이터셋을 최종 데이터셋으로 사용하였습니다. 

1. [빨간공, 흰공, 노란공] -> 손이랑 큐대를 공으로 인식하는 경우가 많이 발생
2. [손, 큐대, 빨간공, 흰공, 노란공] -> 손이랑 큐대를 공으로 인식하지 않으나 하나로 모인 공들을 제대로 인식하지 못함
3. [손, 큐대, (공 2개가 모인 것), (공 3개가 모인 것), 빨간공, 흰공, 노란공] -> 이전 데이터셋보다 공 탐지 성능이 증가했으나 하나로 모인 공을 제대로 인식하지 못하는 문제는 완전히 고쳐지지 않음. 새로운 데이터셋을 만들어서 다시 학습시킬 계획(22.01.31)

## 2. 학습

학습에 사용한 Hyperparameter들은 다음과 같습니다. 

>**epochs** = 300 <br>
**batch size** = 16 <br>
**momentum** = 0.9 <br>
**weight decay** = 0.0005 <br> 
**burn_in** = 1000 # 학습률 관련 Hyperparameter

여기서 burn_in은 학습률 조정에 사용하는 값입니다. 미니 배치 단위로 학습을 수행할 때마다 학습률을 (지금까지 학습에 사용했던 미니 배치의 개수/ burn_in )으로 수정했습니다. 

즉, 학습을 할 때마다 학습률을 조금씩 상승시킨 것입니다.

<br>

### 학습 후 성능
<br>

검증 데이터셋을 이용해 측정한 모델의 성능은 다음과 같습니다. (mAP = 0.6148, 2021.01.30 기준)

|객체 종류|AP|
|------ |-------|
|큐대|0.05052|
|손|0.55097|
|공 2개 모인 것|0.71080|
|공 3개 모인 것|0.04444|
|**빨간공**|0.97318|
|**흰공**|0.97668|
|**노란공**|0.99698|

제가 학습시킨 YOLOv3의 주목적은 '공을 잘 탐지하는 것'입니다. <br>
빨간공, 흰공, 노란공 외의 다른 객체들은 '공이 아닌 것을 공으로 잘못 판정하는 일'을 최대한 줄이기 위해 탐지 대상으로 추가한 것들이며 실제로 탐지 대상을 늘릴 수록 공 탐지에 robust한 모습이 강해지는 것을 확인할 수 있었습니다.

**Note : 22.02.03에 수정한 데이터셋으로 학습을 다시 진행중입니다. epoch 71에서 

|객체 종류|AP|
|------ |-------|
|큐대|0.00000|
|손|0.60066|
|공 2개 모인 것|0.04264|
|공 3개 모인 것|0.00000|
|**빨간공**|0.92029|
|**흰공**|0.92569|
|**노란공**|0.92270|
|**빠르게 움직이는 빨간공**|0.55229|
|**빠르게 움직이는 흰공**|0.44594|
|**빠르게 움직이는 노란공**|0.53204|

위와 같이 성능이 측정되었습니다. 

<br>

## 3. 실행방법

<br>

현재 실행파일 생성 과정에서 생기는 버그로 인해 실행파일을 만들지 못했습니다. 
실행파일읆 만들면 공유하도록 하겠습니다.

<br><br>


<br>

## 4. 테스트 결과
학습시킨 YOLOv3으로 당구공 탐지를 수행했을 때 결과는 다음과 같습니다. 

<h3></h3>             |  <h3></h3>  
:-------------------------:|:-------------------------:
<img width="600" alt="스크린샷 2022-01-30 오전 1 02 44" src="https://user-images.githubusercontent.com/50979281/151748223-96fdd8ff-80c2-4035-bfcd-23a3f1970d26.jpg"> |  <img width="600" alt="스크린샷 2022-01-30 오전 1 02 44" src="https://user-images.githubusercontent.com/50979281/151748239-414f3867-c2fb-4017-a956-02cd6e9b7c94.jpg">


당구대에 흰공이 없거나 흰공의 confidence가 매우 낮아 nms를 통과하지 못하는 경우가 아니면 손과 당구대를 공으로 인식하는 일이 없었습니다. 

<br> <br>

## 4. 개선해야될 부분
두 공이 붙어있을 경우에 대해 학습했으나 두 공이 붙어있어도 한 공으로 인식하는 경우가 계속 발생했습니다. <br>

<h3></h3>             |  <h3></h3>  
:-------------------------:|:-------------------------:
<img width="600" alt="스크린샷 2022-01-30 오전 1 02 44" src="https://user-images.githubusercontent.com/50979281/151749899-c323de34-7a0a-4f8c-91cb-24b0f7853ca8.jpg"> |  <img width="600" alt="스크린샷 2022-01-30 오전 1 02 44" src="https://user-images.githubusercontent.com/50979281/151749909-fd4fec2c-4566-44c7-8045-ca232aad3fcd.jpg">






이를 해결하기 위해 데이터셋의 라벨링 작업을 다시 하거나 데이터셋의 크기를 키우는 방안을 생각중입니다. 

<br>

22.02.03 : 공을 라벨링한 bbox의 크기가 공 전체가 아닌 내부만 포함한 경우가 많아 '공의 경계'를 제대로 학습하지 못했다는 사실을 확인했습니다. 그래서 bbox의 크기를 늘렸습니다. 
<br>그리고 빠르게 움직이는 공에 대해 moving_ball이라고 따로 라벨링을 하였습니다. 

22.02.04 : bbox의 크기를 일괄적으로 상하좌우 1.5씩 늘리고 학습을 수행한 결과, 노란공이 큐대와 가까운 위치에 있어야 공으로 인식이 된다는 사실을 발견했습니다. <br> 그래서 bbox를 상하좌우 1만큼만 늘리고 공을 표시하는 코드를 수정하기로 했습니다. 2022.02.04 오전 10시 50분부터 수정한 데이터셋으로 학습을 다시 시작했습니다.

