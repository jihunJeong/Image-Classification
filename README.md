# Image-Classification
네이버 부스트캠프 AI Tech P stage 1

### <기술적인 도전>

#### 검증(Validation) 전략
- 제공된 Dataset에 대해 Stratified KFold 5를 적용했습니다. 이것을 적용하기 위해 Dataset Class에서 해당 데이터의 label도 따로 모아 후에 학습할 때 넘겨줬습니다.

#### 사용한 모델 Architecture 및 Hyper Parameter

1. Resnet101 & Resnet 50
    - LB 점수 : 0.7100
    - Data Augmentation
      - RandomCrop((330, 220)), ColorJitter(brightness=0.1, contrast=0.2), Normalize([0.560, 0.524, 0.501], [0.233, 0.243, 0.246])
    - Image Size : 330 X 220
    - 추가 시도
      - Batch Size를 처음에 256으로 시작했다가 최종적으로 32를 적용해 Local Minimum에 빠지는 문제를 해결했습니다.
      - Learning rate를 0.01, 1e-04, 1e-05 등 적용 해 가장 나은 값이 1e-04라는 것을 알게 되어습니다.
      - Optimizer 부분에서 SGD와 Adam을 두고 여러 번 반복 시도를 하였지만 큰 차이가 없다는 결론을 내렸습니다.
     
2.	OpenCV Age Detection
    1.	세가지 분류에 대해 Mask Detection, Gender는 높은 확률로 맞췄지만 Age에 대해 성능이 좋지 않았던 점에서 시도하게 되었습니다.
    1.	OpenCV에 있는 Age Detection을 적용해 Images의 나이를 예측했습니다. 이 과정은 먼저 MTCNN에 있는 Pretrained 모델을 불러와 Face Detection을 통해 얼굴의 위치를 찾게 됩니다.
    1.	찾은 얼굴의 위치를 Box 배열로 전달해 OpenCV는 전달받은 Face Image를 Age_net을 이용해 나이를 예측합니다.
    1.	age_list = ['(0 ~ 2)','(4 ~ 6)','(8 ~ 12)','(15 ~ 20)', '(25 ~ 32)','(38 ~ 43)','(48 ~ 53)','(60 ~ 100)'] 나이 분포 List는 이렇게 되어있으므로 이 Labeling을 적절히 이용해 이번 Classification Model에 적용했습니다.
    1.	하지만 이 Model은 서양 얼굴에 대해 예측을 잘하지만 지금 분류하는 동양 얼굴에 대해서는 실제 나이보다 10~20살 정도 어리게 예측하는 문제가 있으며 정확도가 높지 않았습니다.

3.	앙상블(Ensemble) 방법
    - 각 KFold 모델 마다 나온 예측 Label을 K로 나눠 ans에 저장한 뒤 최종적으로 Test가 끝나면 해당 Image에 대한 예측 분포에서 Argmax를 이용한 가장 큰 값을 정답으로 사용하였습니다.

4.	시도했으나 잘 되지 않았던 것들
    -	Overfitting을 피하기 위해 Label Smoothing을 적용해 보았으나 성능 개선이 좋지 않았습니다.
    -	Random Augmentation을 하고 싶어 적용을 해보았으나 적절한 각 Augmentation에 대한 확률 분포를 알 수 없어 개선이 되지 않았습니다.
    -	Resnet Fully Connect 부분에서 Dropout을 적용해 보았으나 실질적인 성능 개선은 이뤄지지 않았습니다. 
    -	처음에 Pretrained Model을 쓰고 싶지 않아 혼자서 스스로 VGG11을 구현해 학습을 시켜봤으나 Data와 Model 양측 문제로 성능이 나오지 않았습니다.
    -	수평, 수직으로 뒤집기와 Resize후 CenterCrop 등 다양한 방식을 시도하였지만 눈에 띄는 성과는 없었습니다. 

