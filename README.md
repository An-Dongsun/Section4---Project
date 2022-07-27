# Section4-Project

# 🎯 해결하고자 하는 문제
## 🎯 Main : X-ray 이미지를 CNN 분류 모델을 통해 정상/세균성 폐렴/바이러스성 폐렴/COVID19으로 분류한다.


## ☑️ 주제 선정 이유

* 요즘 COVID19가 재유행 하고 있다. → 코로나의 증상은 폐렴의 증상과 매우 유사하다.

* 응급실에서 폐렴을 판단하기 위해서 X-ray를 찍고 CT와 배양검사, 혈액검사등 추가 검사를 통해 판단한다.

* 하지만 긴급한 환자의 경우 다양한 검사를 할 여유가 없어서 X-ray 사진과 항원검사를 통해 판단한다.

*  X-ray 사진만으로는 폐렴이 있다, 없다 정도만 알 수 있고 정확도도 30%정도밖에 안된다.

## 가설 설정
1. CNN 기반 이미지 분류 모델이 정상, 세균간염 폐렴, 바이러스 감염, COVID-19를 분류할 수 있을 것이다.
2. 전이학습을 한 모델의 분류 성능은 좋을 것이다.
3. 분류층을 복잡하게 설계하면 분류 성능은 좋을 것이다.
4. 모델이 분류한 근거를 시각화 할 수 있을 것이다.

## 💾 Data collection
* 3 kinds of Pneumonia
* [데이터 출처 : kaggle](https://www.kaggle.com/datasets/artyomkolas/3-kinds-of-pneumonia)

## 💾 Data set 설명
- 정상 → 3,279개
- 폐렴-세균 → 3,011개
- 폐렴-바이러스 → 1,661개
- COVID-19 → 1285개


## 사용한 모델
* 이미지 분류문제 → CNN 모델
  * ResNet-50
  * InceptionResNetV2
  * EfficientNetB7
  
이미지를 분류하는 CNN모델중 제가 배운  모델들중 가장 성능이 좋았던 ResNet-50을 먼저 사용해 보고    
파라미터 개수가 너무 많이 증가하지 않는 선에서 좋은 성능을 보이는 InceptionResNetV2와 EfficientNetB7을 이용해서 전이학습을 진행했다.

* Base 모델의 파라미터는 학습하지 않게 설정

* 동일한 학습 데이터와 동일한 분류기를 설정하고 결과를 확인   
  → ErriciectNetB7이 가장 좋은 성능을 보여주어 최종적으로 EfficientNetB7을 선택했다.
  
## 과적합 방지
* 배치 정규화
* 학습률 계획법 → 학습 속도가 너무 느려져서 제외
* Dropout

##최종 모델
* EfficientNetB7 + Dropout + 배치 정규화
  * base model 부분의 Dropout 30%
![image](https://user-images.githubusercontent.com/102213564/181206448-c74afa13-198c-4ed6-80d1-dbb365709d85.png)

  * 분류기 부분의 배치 정규화 → 분류기의 각 층마다 적용
![image](https://user-images.githubusercontent.com/102213564/181206765-dd163324-0521-4cf5-adb6-23e3633c4ac5.png)

* Input shape = (256, 256, 3)
* Batch size = 8
* Epoch = 50
* Early stop : 10
* Total params = 67,394,715
* Trainable params : 67,075,284 개
![image](https://user-images.githubusercontent.com/102213564/181207114-3a9af3a9-eabf-4b2a-9aee-c9886f7785e3.png)

## 최종모델 학습 결과
<img width="318" alt="image" src="https://user-images.githubusercontent.com/102213564/181207265-2ea96fb0-049f-4da2-865b-e9f0670553b4.png">
* Early stopping : 17 Epoch

  * Train
    * loss : 0.1336
    * accuracy : 0.9570

  * Test
    * loss : 0.3187
    * accuracy : 0.8957
    
과적합이 일어난것을 확인할 수 있다.

## XAI
### CAM(Class Activation Map)
* CNN 이 무엇을 보고 특정 class로 분류 했는지 알 수 없었다. → 이미지의 Heat Map을 생성을 통해 시각화

### Grad-CAM
* 기존 CNN 모델 구조의 변화 없음
* 기존 CNN 모델의 재학습이 필요 없음

* 정상 → 정상의 경우 폐 부분은 잘 안보고 턱과 횡경막의 위치에 주목하는 모습을 보여주었다.
![image](https://user-images.githubusercontent.com/102213564/181207992-f5386d88-eeb6-43e0-a29a-cca7a1798e8a.png)
<img width="318" alt="image" src="https://user-images.githubusercontent.com/102213564/181208026-74c526ad-fa33-477d-8703-7aba60e361a8.png">

* 코로나 → 코로나 같은 경우 폐에서 염증이 일어난 부분을 보고 판단을 내린것을 볼 수 있었다.
![image](https://user-images.githubusercontent.com/102213564/181208194-971957c4-3f7a-44c9-96cb-800632a2b501.png)
<img width="318" alt="image" src="https://user-images.githubusercontent.com/102213564/181208214-24a8e596-64f6-478f-93ee-d8dfb6419dcd.png">

* 세균성 폐렴 + 이물질 → 이미지에 물체가 존재하면 그 물체를 무시하고 다른 부분들을 보고 파악하는 것을 알 수 있었다.
![image](https://user-images.githubusercontent.com/102213564/181208399-69be8fa4-13b3-4d12-972f-00ca242df087.png)
<img width="318" alt="image" src="https://user-images.githubusercontent.com/102213564/181208447-8c2d727d-3306-4a8a-8307-c88e755b30db.png">

* 바이러스성 폐렴 + 예측 실패 → 다른사진들과 다르게 이미지의 오른쪽에 손 부분이 찍혀서 이 부분을 보고 판단해서 예측이 실패했다는 것을 알 수 있었다.
![image](https://user-images.githubusercontent.com/102213564/181208617-cc4cfab8-dd33-43e6-a45e-c4bcd2ddd799.png)
<img width="318" alt="image" src="https://user-images.githubusercontent.com/102213564/181208654-a6e21e26-0497-4d51-9234-acbb7622738b.png">

## 한계점 및 개선 방향
1. 예측이 이상하게 되는 것을 Grad-CAM을 통해서 확인할 수 있지만 분류 결과는 다른 검사들의 결과가 나올때 까지 기다려야 한다.
2. 모델의 예측 결과에 대한 최종적인 판단은 의사선생님이 해야 한다.   
→ 모델의 예측 정확도 향상 & Grad-CAM외의 방법을 통한 시각화 방법 탐색
3. 해상도가 낮은 이미지에 대한 분류 능력이 떨어진다.   
→ AutoEncoder와 같이 이미지 전처리 과정을 추가해 입력 이미지의 해상도를 높이는 방법을 찾아본다.
4. 모델의 과적합   
→ 학습 데이터를 이미지 증강 기법을 통해 다양한 상황에 대한 판단을 잘하는 모델로 제작   
→ 과적합 방지 기법 추가 (다양한 모델 앙상블 등)
