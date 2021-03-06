# (creating draft) Generative Adversarial Nets
### Ian J. Goodfellow et al., 2014
> https://arxiv.org/pdf/1406.2661.pdf


#### 초록
이 논문에서는 적대적 프로세스(adversarial process)를 통해 생성모델을 추정하는 새로운 방법론을 제안하였다. 이 방법론은 G라는 훈련 데이터의 분포를 추정하는 생성모델과 어떤 샘플데이터가 생성모델 G가 아니라 훈련데이터로부터 뽑힐 확률을 계산하는 판별모델(D)를 동시에 학습하는 것이다. G(생성모델)을 학습하는  과정은 판별모델 D가 잘못 판단할 확률을 높이는 것이다. 이 방법은  two player minimax game과 대응된다. 임의의 G와 D 공간에서, 훈련데이터의 분포를 커버하는 G가 존재하며 D가 1/2이 되는 유일한 해가 항상 존재한다. G와 D가 다중퍼셉트론 모형으로 구성될 경우, 전체 과정은 역전파(backpropagation)를 통해 학습할 수 있다. 모델을 훈련하거나 샘플을 생성하는 과정에서 마코프체인이나 unrolled approximate inference는 사용되지 않는다. 이 논문에서 실험을 통해 생성된 샘플데이터의 정량적, 정성적 평가를 시행하였고 이 방법론의 잠재가능성을 설명하였다.

#### 1. 서론
딥러닝은 이미지 인식, 음성인식, 자연어처리 등 인공지능 산업에서 데이터의 확률분포를 표현하는 다양한 모델을 발견하는 성과를 얻었다. 지금까지 딥러닝에서 가장 큰 성공은 고차원의 센서 입력값을 특정 클래스로 맵핑하는 판별모델(discriminative models)를 학습한 것이다. 이러한 성공적인 모델들은 그래디언트가 잘 작동하는 선형조각을 이용해 역전파와 dropout 알고리즘에 기반한다. 반면 깊은 생성모델은 상대적으로 큰 영향력을 미치지 못했는데, 최우추정 및 관련 방법에서 난해한 확률계산을 근사하기 어렵고 분포를 생성하는 목적에서 선형조각들의 장점을 이용하기 어렵기 때문이다. 이 논문에서 이러한 문제점을 피하면서 새로운 생성모델을 추정하는 과정을 제안하였다. 

이 방식에서 생성모델은 적대적인 과정을 통해 학습된다. 어떤 샘플데이터가 훈련데이터에서 뽑힌것인지 생성모델에서 뽑힌것인지 결정하는 것을 학습하는 판별모델과 적대관계를 가진다. 생성모델은 탐지에 걸리지 않고 위조지폐를 만들고 사용하는 위조지폐범과 유사하고, 반면 판별모델은 위조화폐를 발견하려고 하는 경찰에 비유할수 있다. 이 게임의 경쟁구도를 통해 두 팀은 위조화폐가 진짜와 구분되지 않을 때까지 그들의 방법을 개선한다. 

이 방법론을 이용해 다양한 모델을 학습시키는 알고리즘과 최적화 알고리즘을 만들어낼 수 있다. 이 논문에서는 생성모델과 판별모델 모두 다중퍼셉트론 모형을 이용한 케이스를 탐색했다. 이 특별한 케이스를 적대적 네트워크(adversarial nets)라고 부르도록 하겠다. 이 경우에는 forward propagation을 통해 생성모델로부터 샘플데이터과, backpropagation와 dropout알고리즘을 이용해 두 모델을 학습시킬수 있다. 근사적 추정이나 마코프체인은 필요하지 않다. 

#### 2. 관련문헌

#### 3. 적대적 네트워크
적대적 모델 방법론은 두 모델이 모두 다중레이어모델일때 가장 직접적으로 적용된다. Pg 분포를 학습하기 위해 우리는 노이즈변수에 대한 사전분포 pz를 정의한다. 그리고 pz는 theta_g를 파라미터로하는 다중퍼셉트론으로 표현되는 미분가능한 함수 G(z; theta_g)를 통해 데이터와 맵핑된다. 또한 스칼라 값을 아웃풋으로 하는 두번째 다중퍼셉트론 모델 D(x; theta_d)를 정의한다. D(x)는 x가 p_g가 아니라 데이터로부터 추출될 확률을 나타낸다. 우리는 학습데이터의 샘플과 G로 생성된 샘플 모두 정확히 분류할 확률을 최대화하도록 D를 학습힌다. 동시에 log(1-D(G(z)))를 최소화하도록 G를 학습시킨다. 
즉 D와 G는 V(G, D)함수값을 가지는 minmax문제이다. 

![식(1)](eq_1.png)

다음 섹션에서는 대다수 네트워크에 대한 이론적 분석을 알아볼 것이다. 훈련 기준은 G와 D가 충분한 데이터와 학습시간이 주어진다면 데이터의 생성분포를 복수할 수 있게 한다 것이다. 직관적인 이해를 위해 그림1을 참조해라. 실제 구현에서는 반복적인 수치 계산을 통해 게임 모델을 학습해야한다. 학습과정의 내부 루프에서 D를 계속 최적화시키는 것은 한정적인 데이터에서 오버피팅이 발생할 수 있게 때문에 제한된다. 대신에 우리는 D를 k단계의 최적화과정과 G를 최적화하는 1단계를 수행한다. 이를 통해 G가 충분히 천천히 변화하면서, D가 최적 해에 가깝게 유지되도록 한다. 이 전략은 SML/PCD 학습과정에서 마코프체인이 무한루프가 되는 것을 피하기 위해 이전단계에서 다음단계로 넘어갈때 마코프체인에서 뽑힌 샘플을 유지하는 것과 유사하다. 이 과정을 알고리즘1로 표현됩니다. 

실제 적용에서, 목적식 1은 G가 충분히 학습되는데 충분한 그래디언트를 제공하지 못한다. 학습초기에는 G의 성능이 좋지 않으므로 D가 생성된 샘플를 구분할 가능성이 높다. 이경우, log(1-D(G(z))) saturate 된다. log(1-D(G(z))) 를 최소화하도록 G를 학습하는 대신에 logD(G(z))를 최대화하도록 G를 학습시킨다. 이렇게 하면 더 큰 그래디언트를 가지면서 동일한 결과를 얻을 수 있다. 

![그림1](fig_1.png) 
그림1. 생성적 적대 네트워크는 데이터로부터 만들어진 분포(검은색, 점선)와 생성모델에 의한 분포(G, 녹색, 점선)에서 뽑힌 샘플을 구분하기 위해 판별분포(D, 파란색의 점선)을 업데이트하면서 동시에 학습된다. 아래의 수평선은 유니폼 분포의 z가 뽑히는 도메인을 나타낸다. 그 위의 수평선은 x의 도메인을 나타낸다. 위쪽방향의 화살표는 논유니폼 분포 p_g를 통해 x = G(z)를 맵핑하는 방법을 나타낸다. G는 고밀도 영역에서 계약을 하고 p_g의 저밀도 영역에서 확장합니다.(a) 수렴하기 전의 적대적 관계를 생각해보자: p_g는 p_data와 비슷하고 D는 부분적으로 정확한 분류기이다. (b) 알고리즘의 안쪽 루프에서 D는 데이터에서 뽑힌 샘플을 구별하도록 학습되며, D(x)=p_data(x)/(p_data(x) + p_g(x))로 수렴한다. (c) G를 업데이트한 후, D의 기울기는 G(z)가 데이터로 분류될수 있는 영역으로 이동할수 있도록 돕는다. (d) 몇번의 학습과정 후에는(충분한 capacity가 있다면) 두 모델은 p_g=p_data이기때문에 더이상 개선할수 없는 점에 도달하게 된다. 판별모델은 두 분포를 구분할수 없게 된다. (즉, D(x)=1/2)

 
#### 4. 이론적 결과
생성모델 G는 p_z를 따른 z에 대해 G(z)의 샘플의 분포로 확률분포 p_g를 암시적으로 정의한다.  따라서 충분한 데이터와 학습과정을 통해 알고리즘1이 p_data의 좋은 추정치로 수렴해야한다. 이 섹션의 결과는 비모수적 접근이다. 즉 확률분포함수의 공간에서 수렴성을 통해 inifite capacity를 가진 모델을 표현한다.  

섹션 4.1에서는 이 미니맥스게임문제가 p_g = p_data인 글로벌 최적해를 가진다는 것을 보일것이다. 섹션 4.2에서는 알고리즘1이 목적식1을 최적화시키고, 원하는 결과를 얻는 다는 것을 보일것이다. 

-------------
알고리즘1. 미니배치 stochastic gradient descent를 통해 적대적 네트워크를 학습시킨다. 판별모델을 학습시키는 스텝 수, k는 하이퍼파라미터이다. 여기서는 최소실험비용을 위해 k=1이다.

-------------


for number of training iterations do

 for k steps do
  
  * sample minibatch of m noise samples {z_1, ...., z_m} from noise prior p_z(z)
  * sample minibatch of m examples {x_1, ...., x_m} from data generating distributions p_data(x)
  * update the discriminator by ascending its stochastic gradient:

![식_dis](gradient_discriminator.png)

 end for
 
 * sample minibatch of m noise samples {z_1, ..., z_m} form noise prior p_g(z)
 * update the generator by descending its stochastic gradient:

![식_gen](gradient_generator.png)

end for

그래디언트 기반의 업데이트은 그래디언트 기반의 여러 학습 규칙(옵티마이저)를 사용할 수 있다. 이 논문의 실험에서는 모멘텀을 사용하였다.

-------

##### 4.1 p_g = p_data에서의 글로벌 최적해
우리는 먼저 G가 주어졌을때 최적 판별기 D에 대해서 생각해보자

프로포지션1. 주어진 G에 대해서 최적 D는
![식2](eq_2.png)

증명. G가 주어졌을때, D를 학습하는 기준은 V(G,D)를 최대화는 것이다. 
![식3](eq_3.png)

p_data(x)와 p_z(x)는 \[0, 1] 사이 값을 가짐으로 \[0,1]구간의 a와 b에 대해 y->a*log(y) + b*log(1-y) 는 a/(a+b)일때 최대값을 가진다. 그 외 D는 supp(p_data) ∪ supp(p_g) 밖에서 정의될 필요가 없다. 

D를 학습하는 목적은 x가 데이터 분포 p_data(with y = 1)에서 뽑힌것인지, 생성모델의 분포 p_g(with y = 0)에서 뽑인것인지 나타내는 Y 확률변수의 P(Y = y|x) 조건부확률을 추정하기위해 로그-우도값을 최대화하는 것으로 해석할 수 있다. 식(1)의 최소최대문제는 아래처럼 다시 쓸수 있다. 
![식4](eq_4.png)


정리1. 가상의 C(G)는 p_g=p_data일 때 글로벌 최소값을 가진다. 이 최소점에서 C(G)의 값은 -log4이다. 

증명. p_g=p_data이면, D_G(x) = 1/2이다. 위의 식4에 대입하면 E_x~p_data\[log1/2] + E_x~p_g\[log1/2] = - log4이다. 이 값이 C(G)의 최소값이라는 것은 보이기 위해 C(G)=V(D,G)를 빼면 아래와 같다. 

![식5](eq_5.png)

KL은 Kullback–Leibler divergence이다. 식5를  데이터 분포와 생성모델의 분포의 Jensen–Shannon divergence로 변형하면 식6과 같다.

![식6](eq_6.png)

두 분포의 Jensen–Shannon divergence는 항상 0 또는 양수이고, 두 분포가 같을 때만 0값을 가진다. 따라서 우리는 C(G)= -log4 값이 최소값이고 이는 p_g = p_data일때만 가능하다. 즉 생성모델이 완변히 학습데이터를 복제하여 생성할때 이다. 

##### 4.2 알고리즘1의 수렴
정리2. G와 D가 충분한 용량이 있다면, 알고리즘1의 매 스텝에서 주어진 G에 대해서 D는 최적해에 가까워진다. 그리고 p_g는 아래 학습기준값을 개선시키기는 방향으로 업데이트되므로 p_g는 p_data로 수렴한다.

증명. 위 학습기준값을 p_g에 대한 함수로써 V (G, D) = U(p_g, D) 표현하자. U(p_g, D)는 p_g의 컨벡스 함수이다. 
The subderivatives of a supremum of convex functions include the
derivative of the function at the point where the maximum is attained. In other words, if f(x) =
sup_α∈A f_α(x) and f_α(x) is convex in x for every α, then ∂fβ(x) ∈ ∂f if β = arg supα∈A fα(x).
This is equivalent to computing a gradient descent update for pg at the optimal D given the corresponding
G. supD U(pg, D) is convex in pg with a unique global optima as proven in Thm 1,
therefore with sufficiently small updates of pg, pg converges to px, concluding the proof.

실제 적용에서 적대적 네트워크는 G(z; θg) 함수의 제한적 패밀리 분포로 표현되고, p_g보다는 θg를 최적화시킨다. 
다중레이어 퍼셉트론을 이용해 파라미터 공간에서 다양한 기준 점들로 유도된다. 하지만 이론적으로 보장되기보다는 실제에서 좋은 성과를 내기때문에 합리적인 모델로 제안된다. 

#### 5. 실험
MNIST, the Toronto Face Database(TFD), and CIFAR-10 데이터를 이용해 적대적 네트워크 모델을 학습하였다. 생성모델은 ReLu(rectifier linear activation)와 시그모이드 활성함수를 사용하고, 판별모델은 맥스아웃 함수를 사용하였다. 또한 판별모델은 dropout을 적용하였다.  이론 상으로는 생성모델의 중간 레이어에서 노이즈를 추가하거나, dropout을 사용해도 무방하지만, 생성모델의 처음 레이어의 입력값으로만 노이즈를 사용하였다. 

학습된 p_g에서 테스트셋 데이터의 확률을 추정하기 위해 G로 생성된 샘플에 가우시안 파첸 윈도우를 피팅시켰고, 이 분포의 로그-우도를 계산하였다. 가우시안의 σ 파라미터는 검증셋에 대해서 교차검증을 통해 얻었다. 이 방법은 Breuleux et al. 에서 소개되었고, 정확한 우도값을 추적할수 없는 생성모델에 사용되었다. 실험 결과는 표1과 같다. 우도값을 추정하는 방법은 편차가 크고 고차원에서 잘 작동하지 않지만, 생각할수 있는 최선의 대안이다. 샘플 데이터를 생성할수 있지만, 우도값을 계산할수 없는 문제가 있어 생성모델의 발전 가능성은 그러한 모델을 평가하는 방법에 대한 추가 연구로 이어질 수 있다.

그림2와 그림3는 생성모델로 생성한 샘플들이다. 기존의 다른 생성모델보다 더 좋은 샘플을 생성했다고는 할수 없지만, 적어도 이 샘프들이 기존의 것과 동일한 수준이며 더 발전가능성이 있음을 확인하였다. 

![표1](tb_1.png)

표1: 파첸 윈도우 기반의 로그 우도 추정값. MNIST는 테스트셋 샘플들을의 평균 우도값과 교차검증의 표준편차임. TFD는 서로 다른 시그마를 이용한 교차검증을 통해 계산한 표준편차임. MNIST데이터는 다른 모델과 비교평가하였다. 


![그림2](fig_2.png)

![그림3](fig_3.png)

![표2](tb_2.png)



참고 reference

* https://brunch.co.kr/@kakao-it/145
* https://brunch.co.kr/@kakao-it/162
* https://www.slideshare.net/NaverEngineering/1-gangenerative-adversarial-network
* https://www.slideshare.net/carpedm20/pycon-korea-2016

