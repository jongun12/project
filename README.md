# 6-DOF Camera Pose Estimation (PoseNet 기반)

## 1. 프로젝트 개요
특정 건물에 대한 영상으로 모델을 학습시킨 후 그 건물에 대한 RGB 이미지를 input 으로 넣으면 실시간으로 6자유도 카메라 포즈인 위치(X, Y, Z) 와 방향(W, P, Q, R)을 예측하는 모델을 개발하는 프로젝트입니다.
자율주행 로봇(특히 실내) 또는 내비게이션 그리고 증강현실에 중요한 기술입니다. 또한 사전에 지도를 만들 필요가 없고 사진을 통해 학습시킬 수 있다는 장점이 있습니다.

## 2. 파일 및 폴더 구조
* **`cambridge.py`**: Dataset root 와 mode (train/test), transform을 매개변수로 받아 데이터셋을 불러오고 전처리하는 커스텀 Dataset 로더입니다.
* **`criterion.py`**: 학습에 사용되는 Loss 함수가 정의되어 있습니다.
* **`utils.py`**: 학습된 모델을 평가(Eval)하기 위해 위치 오차와 회전 오차를 계산하는 유틸리티 모듈입니다.
* **`training.ipynb`**: 패키지 로드, 데이터 로더 설정, 모델 초기화, Optimizer 설정 및 전체적인 Training / Validation 루프가 포함된 학습용 파이프라인 노트북입니다.
* **`models/`**: 실험에 사용된 다양한 신경망 모델들이 구현되어 있습니다.
  * `mobilenet.py`: MobileNetV2 백본 기반 모델.
  * `resnet.py`: ResNet152 백본 기반 모델.
  * `vgg.py`: VGG16 백본 기반 모델.
  * `simplenet.py`: 자체 구성한 커스텀 CNN 모델.
  * `eachnet.py`: Transformation 과 Rotation 은 서로 필요한 정보가 다르다고 생각되어 위치(X,Y,Z)와 방향(W,P,Q,R) 레이어를 각각 분리하여 MobileNet 백본으로 구현한 자체 모델입니다.

## 3. 알고리즘 및 접근 방법
* **손실 함수 (Loss Function)**: Transformation 과 Rotation 의 각각의 예측 값과 참값과 beta 값을 매개변수로 받습니다. 
  * 수식: $loss(I)=||\hat{x}-x||_{2}+\beta||\hat{q}-\frac{q}{||q||}||_{2}$
  * tr_loss 과 rot_loss 의 범위 차이를 줄이기 위해 rot_loss 에 beta 값을 곱해줍니다.
  * 실험에서 beta 값은 주로 300을 사용하였습니다.
* **전처리 (Transform)**: 이미지 크기를 224x224 또는 360x480로 변환하였습니다. 모델이 안정적으로 성능을 발휘할 수 있도록 R, G, B 평균(0.485, 0.456, 0.406)과 표준편차(0.229, 0.224, 0.225)를 사용해 정규분포로 만들었습니다.
* **평가 (Evaluation)**: 위치 오차는 참값과 예측 값의 유클리디안 거리를 사용합니다. 회전 오차는 Rotation 의 쿼터니언 값을 오일러 각도로 변환한 후 참값과 예측 값의 유클리디안 거리를 계산하여 사용합니다.

## 4. 데이터셋 및 개발 환경
* [cite_start]**데이터셋**: Posenet 논문에서 공개한 Cambridge Street 데이터를 사용하였습니다[cite: 39]. [cite_start]Training과 Valuation 데이터는 8:2로 나누어 학습을 진행하였습니다[cite: 25].
* **하드웨어**: NVIDIA GeForce RTX 3080
* [cite_start]**운영체제**: 우분투 (Ubuntu) 20.04 [cite: 42]
* [cite_start]**소프트웨어 및 프레임워크**: Visual Studio Code, Python, PyTorch [cite: 42]

## 5. 실험 결과
[cite_start]여러가지 모델(VggNet, PoseNet, EachNet, ResNet, RamNet 등)을 적용하여 가장 효과적으로 loss를 낮춰주는 모델을 모색하였습니다[cite: 61].
* [cite_start]백본을 서로 분리한 **EachNet** 모델이 실험에서 가장 낮은 `val_loss`를 기록하며 좋은 성과를 보였습니다[cite: 56, 57].
* [cite_start]MobileNet 백본 모델은 Batch 크기를 크게 가져갈 수 있어 좋은 결과가 나왔습니다[cite: 52, 53].
* [cite_start]ResNet 모델의 경우 후반부에서 loss가 20점 대에서 수렴하여 잘 내려가지 않는 모습을 보였습니다[cite: 54, 55].

## 6. 팀원 역할
* [cite_start]**김민 (2019120015)**: 문서작성자로 팀 내 활동지를 매주 제출하는 역할을 진행하였습니다[cite: 2, 73].
* [cite_start]**김종언 (2020120019)**: 리더 역할을 수행하였으며, 개발자로 EachNet 과 ResNet, VggNet, PoseNet 등을 구현하였습니다[cite: 2, 74, 75].
* [cite_start]**김준식 (2019120033)**: 서브 문서 작성자 겸 개발자로서 팀원들의 의견을 수용하여 최대한 적용될 수 있도록 노력하였고, 하드웨어 문제가 있을 시 해결하는 역할을 수행하였습니다[cite: 2, 76, 77].

## 7. 참고 문헌
* [cite_start]PoseNet: A Convolutional Network for Real-Time 6-DOF Camera Relocalization, Alex Kendall, Matthew Grimes, Roberto Cipolla [cite: 80]
