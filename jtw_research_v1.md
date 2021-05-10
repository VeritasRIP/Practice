

# Similarity b/w SCM and Flow model
시작: 되게 단순한 접근, definition of SCM:
$$
x_j=f_j(pa_j,n_j), j=1,2,...d,  pa_j \perp\!\!\!\perp n_j 
$$
이를 단순화해서 $x_{pa}$ 와 $x_{ch}$만 고려해보면,
$$
x_{pa}=f_{pa}(n_{pa}) \\
x_{ch}=f_{ch}(x_{pa},n_{ch})
$$

SCM은 deterministic function의 input-output형태의 그래프가 되고, 이걸 어디선가 본적이 있음:
Flow model-RealNVP [https://arxiv.org/abs/1605.08803]

RealNVP-Eq.7
$$
y_{1:d}=x_{1:d} \\
y_{d+1:D}=x_{d+1:D} \odot \exp(s(x_{1:d}))+t(x_{1:d})
$$

RealNVP-Eq.8
$$
x_{1:d}=y_{1:d}  \\ x_{d+1:D}=(y_{d+1:D}-t(y_{1:d})) \odot \exp(-s(y_{1:d})) \\ =(y_{d+1:D}-t(x_{1:d}))\odot \exp(-s(x_{1:d}))
$$

맨 마지막 식에서 $x_{d+1:D}$를 $x_{ch}$ // $x_{1:d}$를 $x_{pa}$ //  $y_{d+1:D}$를 $n_{ch}$에 대입하면,  RealNVP와 SCM간의 유사성을  볼 수 있게 됨 (날먹설명 ㅈㅅ)

이러한 유사성을 눈치채서 날먹한 페이퍼가 있음: https://arxiv.org/abs/2011.02268  (Causal Autoregressive Flows) 
https://arxiv.org/abs/2007.09390

둘다 날먹한 페이퍼 답게, 방법론 또한 "완벽히" 동일.
summary 하자면, 두 페이퍼 모두 causal discovery를 목적으로 하며, Flow model하에서 exact likelihood를 구할수 있다는 점에 착안하여 $x_1->x_2$ (causal)과 
$x_2->x_1$(anticausal)에 대한 flow 모델을 training data로 학습 한 뒤, test data에 대해 두 모델의 likelihood를 평가, 더 높은 쪽을 true causal direction으로 정하겠다는 방법임.

# 왜 Flow model이 causal inference에서 이렇게 밖에 소비되지 못했을까?

Flow 모델의 특징을 몇자 적어보자면
1) evaluate the exact likelihood (ELBO ㄴㄴ)
2) Invertible function formula
3) 0 reconstruction, regardless to flow model parameter

요정도로 적을 수가 있을텐데, 내가 첫째로 Flow model과 SCM과의 관계에 대해 관심을 갖게 된 것은 2) property 때문임. https://arxiv.org/abs/2011.02268  (Causal Autoregressive Flows) 의 abstract에 적힌 부분을 따와 보자면

"...Third, we demonstrate that flows naturally allow for direct evaluation of both interventional and counterfactual queries, the latter case being possible due to the invertible nature of flows."

여기서 주목하는 것은 counterfactual queries에 대한 평가가 가능하다는 부분임. flow model의 fucntion들 (i.e. $f_{ch}$)이  invertible하기 때문에, $x_{ch}=f_{ch}(x_{pa},n_{ch})$ 에서 우리는 $n_{ch}$의 (posteior)분포가 아닌, realizing된 값을 자체를 얻을 수 있게됨. 따라서 $do(x_{pa}=x^*)$를 할 시, $x_{ch}$에 대한 counterfactual값도 알 수 있게 됨 ($=f(x^*,n_{ch})$).

물론 이러한 것들이 flow model 학습 이후의 이야기이긴 하지만, 학습 과정에서 이러한 성질을 이용할 수 있진 않을까 하는 생각을 하게됨. 아쉽다랄까?

# 그래서 연구는?

[1] Flow model base로 multiple variable에 대한 causal discovery : 키메라 with "Learning Neural Causal Models From Unknown Interventions"
https://arxiv.org/abs/1910.01075
--> Flow model+ episodic training 
--> 결국 few shot으로 학습한 likelihood 장난질
참조: https://arxiv.org/abs/1901.10912 (A Meta-Transfer Objective for Learning to Disentangle Causal Mechanisms)

[2] Causal variable learning: 
참조: page 9 of "toward causal representation learning"  https://arxiv.org/abs/2102.11107

Eq.10 of toward causal representation:
$$
X=G(S_1,...S_n)
$$
Eq.12 of toward causal representation learning:
$$
S_i=f_i(pa_i,U_i),  i=1,2,...n  \ \ \  pa_i \perp\!\!\!\perp U_i
$$

mind: 우리가 관측하는 데이터가 causal structured 될 리가 없어! (e.g. $X=(x_1,...x_d)$, 이미지 픽셀 데이터). $X$로 부터 Causal structured variable $(S_1,...S_n)$으로extract (혹은 encode) 할 필요가 있다.

여기서 $(S_1,...S_n)$을 잘 추출하였는 가에 대한 평가가 필요하며,  decoder network ($G$, eq.10 of toward causal representation)는 그러한 평가 요소중 하나를 맡게 된다.

이 decoder 또한 학습을 요구하게 되는데, 이 $G$에 대해 flow model 혹은 다른 invertiable fuctnion formula의 network로 대체한다면, flow model의 3) property에 의해 불필요한 학습을 제거 할 수 있지 않을까 하는 생각을 해봄 (문과적 상상주의)

그리고  Flow model의  2) property로 부터 얻게 되는 counterfactual queries가 $S_i$들을 추출 혹은 그 추출법을 학습하는데 있어 도움이 될 수 있지 않을까 생각해봄 (참조: # Structured Representation Learning using Structural Autoencoders and Hybridization https://arxiv.org/abs/2006.07796 || 이 논문에서 $S_i$ 추출 학습을 하는데 있어 "억지" intervention을 가하게 되는데, 이를 counterfactual 로 대체할 수 있을지도)

# 개 날먹 보고서 ㅈㅅ
원래 주말에 논문 떨어지고 그 빡침을 양분삼아 논문 ㅈㄴ 보려했으나 말도안되게 붙어버려서 술쳐먹음 ㅈㅅㅈㅅ. 글구 몬헌이 생각보다 잼있더라. 














 











