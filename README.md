# 6-DOF Camera Pose Estimation (PoseNet 기반)

## 1. 프로젝트 개요
특정 건물에 대한 영상으로 모델을 학습시킨 후, 해당 건물의 RGB 이미지를 입력하면 실시간으로 6자유도 카메라 포즈인 위치(X, Y, Z)와 방향(W, P, Q, R)을 예측하는 모델을 개발하는 프로젝트입니다. 
자율주행 로봇(특히 실내), 내비게이션, 증강현실 등에 중요한 기술이며, 사전에 지도를 만들 필요 없이 사진을 통해 학습시킬 수 있다는 장점이 있습니다.

## 2. 파일 및 폴더 구조
* **`cambridge.py`**: 데이터셋 경로와 모드(train/test), transform을 매개변수로 받아 데이터를 불러오고 전처리하는 커스텀 Dataset 로더입니다.
* **`criterion.py`**: 학습에 사용되는 Loss 함수가 정의되어 있습니다.
* **`utils.py`**: 학습된 모델을 평가(Eval)하기 위해 위치 오차와 회전 오차를 계산하는 유틸리티 모듈입니다.
* **`training.ipynb`**: 패키지 로드, 데이터 로더 설정, 모델 초기화, Optimizer 설정 및 전체적인 Training / Validation 루프가 포함된 학습용 파이프라인 노트북입니다.
* **`models/`**: 실험에 사용된 다양한 신경망 모델들이 구현되어 있습니다.
  * `mobilenet.py`: MobileNetV2 백본 기반 모델
  * `resnet.py`: ResNet152 백본 기반 모델
  * `vgg.py`: VGG16 백본 기반 모델
  * `simplenet.py`: 자체 구성한 커스텀 CNN 모델
  * `eachnet.py`: Transformation과 Rotation은 서로 필요한 정보가 다르다고 판단하여 위치와 방향 레이어를 분리하고, MobileNet 백본으로 구현한 자체 모델입니다.

## 3. 알고리즘 및 접근 방법
* **손실 함수 (Loss Function)**: 위치(Transformation)와 방향(Rotation)의 예측값과 참값을 매개변수로 받아 오차를 계산합니다.
  * **수식**: `Loss = (위치 오차) + beta * (회전 오차)`
  * **상세 수식**: `loss(I) = ||x_pred - x_true||_2 + beta * ||q_pred - (q_true / ||q_true||)||_2`
  * 두 오차 값의 범위 차이를 줄이기 위해 회전 오차에 `beta` 값을 곱해주며, 실험에서는 주로 `beta = 300`을 사용했습니다.
* **전처리 (Transform)**: 이미지 크기를 224x224 또는 360x480으로 변환하고, 모델의 안정적인 학습을 위해 RGB 평균(0.485, 0.456, 0.406)과 표준편차(0.229, 0.224, 0.225)를 사용하여 정규화(Normalization)를 진행했습니다.
* **평가 (Evaluation)**: 위치 오차는 참값과 예측값의 유클리디안 거리를 사용하며, 회전 오차는 쿼터니언(Quaternion) 값을 오일러 각도(Euler angle)로 변환한 뒤 참값과 예측값의 유클리디안 거리를 계산해 평가합니다.

## 4. 데이터셋 및 개발 환경
* **데이터셋**: PoseNet 논문에서 공개한 Cambridge Street 데이터를 사용하였으며, Train/Validation 비율은 8:2로 분할했습니다.
* **하드웨어**: NVIDIA GeForce RTX 3080
* **운영체제**: Ubuntu 20.04
* **소프트웨어 및 프레임워크**: Visual Studio Code, Python, PyTorch

## 5. 실험 결과
VggNet, PoseNet, EachNet, ResNet, RamNet 등 여러 모델을 적용하여 Train/Validation loss를 효과적으로 낮추는 모델을 탐색했습니다.
* 백본을 서로 분리한 **EachNet** 모델이 가장 낮은 `val_loss`를 기록하며 우수한 성능을 보였습니다.
* MobileNet 백본 모델은 배치(Batch) 사이즈를 크게 설정할 수 있어 비교적 좋은 결과를 얻었습니다.
* ResNet 모델은 학습 후반부에서 loss가 20점대 부근에서 더 이상 수렴하지 않는 한계를 보였습니다.

## 6. 팀원 역할
* **김민 (2019120015)**: 문서 작성 및 주간 팀 활동지 제출
* **김종언 (2020120019)**: 팀 리더 및 핵심 개발자 (EachNet, ResNet, VggNet, PoseNet 등 모델 구현)
* **김준식 (2019120033)**: 서브 문서 작성 및 개발, 팀원 의견 취합 및 하드웨어 트러블슈팅

## 7. 참고 문헌
* PoseNet: A Convolutional Network for Real-Time 6-DOF Camera Relocalization, Alex Kendall, Matthew Grimes, Roberto Cipolla
