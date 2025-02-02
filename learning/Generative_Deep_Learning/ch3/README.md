## Chapter 3 오토인코더

---

**오토인코더(AutoEncoder) 란?** 오토인코더는 입력 데이터를 주요 특징으로 효율적으로 압축(인코딩)한 후 이 압축된 표현에서 
원본 입력을 재구성(디코딩)하도록 설계된 신경망 아키텍처로 인코더와 디코더로 구성되어 있습니다.

- **인코더(Encoder):** 인지 네트워크 라고도 하며, 이미지 같은 고차원 입력 데이터를 내부표현 즉, 저차원 임베딩 벡터로 압축합니다.
- **디코더(Decoder):** 생성 네트워크 라고도 하며, 임베딩 벡터를 원본 도메인으로 압축 해제합니다.

오토인코더는 입력 데이터가 인코더로 들어가서 디코더로 나오면서 고차원 데이터를 재구성하도록 훈련됩니다.  

[**keras를 이용한 오토인코더 구현**](https://github.com/ChanghyunRyu/project-gaerchen/blob/main/learning/Generative_Deep_Learning/ch3/autoencoder_example.ipynb)

그림1은 위의 링크에서 MNIST 패션 이미지를 인코딩한 임베딩 공간이다. 고차원 입력 데이터(이미지)를 저차원 벡터(2D)로 압축한 것이다.

[그림 1]
<img src="https://github.com/user-attachments/assets/3808e80b-2e1f-41f8-8feb-50e989879c2b" height="250">

우리는 디코더를 이용하여 새로운 2D벡터로부터 새로운 이미지를 생성할 수 있다. 그러나 이때 다음과 같은 문제점이 발생하여 성능이 저조하게 나온다.

- 임베딩 공간에서 원본 이미지가 존재하지 않은 큰 공백이 존재한다. 위의 이미지에서는 상단 가운제와 하단 좌우가 이에 해당한다. 
오토인코더는 임베딩공간이 연속적인 것을 강요하지 않기 때문이다.


- 위의 그림에서 각 점의 색은 다른 의류의 종류를 나타낸다. 같은 색들은 비교적 비슷한 공간내에 비치되어 있다.
그러나 색의 구분이 애매한 구역들이 보이고 이러한 구역내에서 혼선이 발생하여 실제 원하던 이미지와 다른 형태의 이미지를 생성할 가능성이 높다.

### 변이형 오토인코더

- **인코더 변화:** 이전 오토인코더에선 각 이미지가 잠재 공간의 한 포인트에 2D 벡터로 직접 매핑되었습니다.
변이형 오토인코더에서는 각 이미지가 잠재 공간에 있는 포인트 주변의 다변량 정규 분포에 매핑됩니다. 
    - 이전 인코더는 이미지를 단순한 2D 벡터로 압축했습니다. 그러나 변형된 인코더는 이미지를 다음 2개의 벡터로 인코딩합니다.
  z_mean(이 분포의 평균 벡터), z_log_var(차원별 분산의 로그 값)
    - 이전에는 임베딩 공간이 연속적으로 만들어지지 않았습니다. 이제 샘플링을 할 때, 
  z_mean 주변 영역에서 랜덤한 포인트를 샘플링하기 때문에 같은 영역에 위치한 포인트를 매우 비슷한 이미지로 디코딩하게 됩니다.
    - 이제 임베딩 공간에서 본 적이 없는 포인트를 선택하더라도 제대로 된 이미지로 디코딩할 가능성이 높아집니다.


- **손실 함수 변화:** 이전 오토인코더에서 손실 함수는 원보 이미지와 오토인코더를 통과한 출력 사이의 재구성 손실로만 구성되었습니다.
변이형 오토인코더에서는 추가로 KL 발산(Kullback-Lelibler divergence)를 추가로 사용합니다. 이는 한 확률분포가 다른 분포와 얼마나 다른지를 측정하는 도구입니다.
  - KL 발산 항을 추가할 경우, 샘플이 표준 정규 분포에서 크게 벗어날 경우 인코더에 피드백을 합니다. 
  따라서 이후 임베딩 공간에서 어떤 지점을 선택할 때 표준 정규 분포를 사용할 수 있습니다.
  - 인코딩된 모든 분포가 표준 정규 분포와 가깝게 됩니다. 이에 따라 클러스터 사이에서 큰 간격이 생길 가능성이 적어집니다.
  - 기존 손실 함수(재구성 손실)의 가중치가 너무 크면 KL 손실이 힘을 쓰지 못 해 기존 오토인코더의 문제점을 반복합니다. 
  반대로 KL 손실에 너무 가중치를 두면 재구성 이미지 품질이 나빠집니다. 이를 적절히 튜닝하는 것이 중요합니다.

[**Keras를 사용하여 변형 오코인코더 구현**](https://github.com/ChanghyunRyu/project-gaerchen/blob/main/learning/Generative_Deep_Learning/ch3/vae_example.ipynb)
