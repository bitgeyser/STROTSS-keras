# Style Transfer by Relaxed Optimal Transport and Self-Similarity

#### Nicholas Kolkin, Jason Salavon, Greg Shakhnarovich, CVPR 2019  [[arXiv]](https://arxiv.org/abs/1904.12785) [[PDF]](https://arxiv.org/pdf/1904.12785.pdf)


## 1. Introduction

- Style Transfer 문제에서의 가장 핵심적인 부분은 얼마나 style과 content를 잘 섞어 구성하는가이다. 위 연구에서는 비전 분야에서는 흔하지만 Style Transfer 분야에는 새로운 공식들을 각 요소에 도입했다. 또한 강건한 Style Transfer 인지 시스템보다는 기술의 효용성에 집중했다. 
  - style을 뉴럴넷이 뽑아낸 feature의 분포로 정의하고, 이들 간의 distance를 Earth Movers Distance를 사용해 근사했다. 
  - 인간의 시각 인지가 이미지의 주변을 보고 물체를 인식하는 데에 기반해, content를 self-similarity로 정의해 픽셀의 실제 값이 크게 바뀌어도 구조와 의미가 좀 더 유지되도록 했다. 
  - Style Transfer의 미적 도구로서의 효용성을 높이기 위해 구역별/지점별로 조건을 줄 수 있도록 했다. 
- 위 연구 결과를 기존 연구들과 비교하기 위해 Amazon Mechanical Turk (AMT) 에서 662명을 대상으로 한 인간 평가를 통해 정량적으로 평가했다. 
  - 두 개의 입력 이미지에서 나온 두 method의 결과와 입력 이미지 중 하나가 주어지고, 둘 중 어느 것이 입력 이미지와 비슷한지 style과 content에 대해 각각 평가하도록 했다. 이를 통해 style과 content 두 축 모두에서 성능을 평가할 수 있었다. 
  - hyper-parameter에 따라 각 method가 얼마나 달라지는지를 측정해 각각의 style-content 간 trade-off를 구했을 때 기존 연구들보다 같은 수준의 content 보존에서 더 나은 품질의 style이 반영됨을 확인했다. 


## 2. Methods

- 기존 Style Transfer와 같이 style 이미지 $I_S$, content 이미지 $I_C$ 두 개를 입력받고, 출력 이미지 $X$에 대한 우리의 objective function을 최소화하기 위해 RMSprop을 사용했다. 
  - content loss인 $\alpha \ell_C$를 **2.2**에서 설명하고, style loss인 $\ell_m + \ell_r + {{1 \over \alpha}} \ell_p$을 **2.3**에서 설명한다. 
  - hyper-parameter $\alpha$는 style 적용에 대한 content 보존의 상대적 비율을 의미한다. 
  - 우리의 방법은 반복적이다; $X^{{(t)}}$ 를 타임스탬프 t에서의 결과 이미지라 정의한다. $X^{{(0)}}$의 초기화를 **2.5**에서 설명한다. 
$$
L(X, I_C, I_S = {{ 
    {{ \alpha \ell_C + \ell_m + \ell_r + {{1 \over \alpha}} \ell_p }} 
    \over 
    {{ 2 + \alpha + {{1 \over \alpha}} }} }}) \qquad (1)
$$

### 2.1 Feature Extraction

- style과 content loss term 모두 임의의 위치에서 좋은 feature representation을 추출하는 것에 목적이 있다. 이 연구에서는 ImageNet으로 학습된 VGG16의 레이어 일부에서 hypercolumn을 뽑아 사용했다. 
  - $\Phi(X)_i$를 이미지 $X$가 네트워크 $\Phi$의 i번째 레이어를 통과한 feature라고 정의한다. 레이어 $l_1$, ..., $l_L$에 대해 $\Phi(X)_{{l_1}}$...$\Phi(X)_{{l_L}}$를 이미지 $X$의 크기에 맞게 bilinear upsampling하고 feature 축으로 모두 이어붙인다. 이는 각 픽셀의 단순한 edge들과 색, 질감, 그리고 semantic한 정보를 담은 hypercolumn이 된다. 
  - 모든 실험에서 우리는 메모리 한계 때문에 VGG16의 레이어 9,10,12,13을 제외한 나머지 모든 레이어를 사용했다. 

### 2.2 Style Loss

- ~~$A = \{A_1,...,A_n\}$를 이미지 $X^{{(t)}}$에서 추출한 n개의 벡터, $B = \{B_1,...,B_m\}$를 style 이미지 $I_S$에서 추출한 m개의 벡터라 할 때, style loss는 Earth Movers Distance (EMD)를 통해 구한다. 
  - $T$는 partial pairwise assignment를 정의하는 'transport matrix', $C$는 $A$의 요소가 $B$의 요소와 얼마나 떨어져 있는지 정의하는 'cost matrix'이다. 
  - $EMB(A, B)$는 집합 $A$와 $B$ 간의 거리를 계산하는데, 최적의 $T$를 $O(\max(m, n)^3)$ 시간으로 찾으며, Style Tranfer의 gradient descent에 영향받지 않도록 한다. (따라서 매 update step마다 계산이 필요하다. )
$$
EMD(A, B) = \min_{T \ge 0} \sum_{ij} T_{ij}C_{ij} \qquad (2) \\
\qquad \qquad \quad \ \ s.t. \sum_{j} T_{ij} = 1/m \qquad (3) \\
\qquad \qquad \qquad \quad \sum_{i} T_{ij} = 1/n \qquad \ (4)
$$
- 이 방법 대신 우린 Relaxed EMD를 사용했는데, 기본적으로 (3)과 (4) 중 한 개의 제약만 가지는 EMD인 두 개의 부가적인 거리를 사용한다. 
