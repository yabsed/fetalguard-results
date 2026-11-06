# 01 · 산모 단위 분할

[00](00-data-preparation.md)에서 구성한 구간을 학습·검증·시험에 배정한다. 예측 대상은 구간의 전문가 판독이며, 평가 대상은 모델 개발에 사용되지 않은 산모이다. 같은 산모의 반복 검사와 쌍태아 기록은 함께 배정한다.

## 1. 산모의 배정과 구간의 배정

00의 구간 인덱스 집합을 $I=\{0,\ldots,n-1\}$라 하자. 산모 집합과 산모별 구간 집합은

$$
M=\{m_i:i\in I\},
\qquad
I_m=\{i\in I:m_i=m\}
$$

이다. 남은 구간이 없는 산모는 $M$에 포함되지 않는다.

산모의 배정은 함수

$$
g:M\longrightarrow\{\mathrm{tr},\mathrm{va},\mathrm{te}\}
$$

로 나타낸다. 아래 첨자는 각각 학습·검증·시험을 뜻한다. 산모의 배정으로부터 구간의 배정이 정해진다.

$$
\boxed{
\begin{aligned}
M_a&=\{m\in M:g(m)=a\},\\
I_a&=\{i:m_i\in M_a\},\\
D_a&=\{(x_i,y_i,m_i,r_i,s_i):i\in I_a\}.
\end{aligned}
}
$$

$g$가 각 산모에 하나의 값을 부여하므로 $M_a\cap M_b=\varnothing$ $(a\ne b)$이다. 구간 수나 기록 수에 관계없이 같은 산모의 관측값은 하나의 집합에 속한다.

## 2. 이상 여부에 따른 층화

산모 $m$에게 남은 이상 구간이 하나라도 있는지를

$$
b_m=\max_{i\in I_m}y_i,
\qquad
M^{(b)}=\{m\in M:b_m=b\}
$$

로 나타낸다. 산모를 $b_m=0$과 $b_m=1$의 두 층으로 나누고, 각 층 안에서 무작위로 배정한다. 목표 비율은

$$
(p_{\mathrm{tr}},p_{\mathrm{va}},p_{\mathrm{te}})
=(0.8,0.1,0.1),
\qquad
|M_a\cap M^{(b)}|\approx p_a|M^{(b)}|
$$

이다. 산모 수가 정수이므로 목표 비율과 실제 비율에는 차이가 생길 수 있다.

$\operatorname{Split}(U,q,s)$를 산모 집합 $U$에서 $b_m$으로 층화하여 비율 $q$를 분리하는 연산이라 하자. 반환값은 남은 집합과 분리한 집합이며, $s$는 난수 시드이다. 실제 배정은 두 단계로 구성한다.

$$
\begin{aligned}
(M_{\mathrm{dev}},M_{\mathrm{te}})
&=\operatorname{Split}(M,0.1,42),\\
(M_{\mathrm{tr}},M_{\mathrm{va}})
&=\operatorname{Split}(M_{\mathrm{dev}},1/9,43).
\end{aligned}
$$

첫 단계에서 시험용 $10\%$를 분리하고, 남은 $90\%$의 $1/9$을 검증에 배정한다. 각 단계에서 산모 ID를 정렬하고 시드를 고정하며, 분할이 실패해도 시드를 바꾸어 재시도하지 않는다.

## 3. 산모 비율과 구간 비율

산모마다 가진 구간 수와 이상 구간의 비율은 다를 수 있다. 이를

$$
n_m=|I_m|,
\qquad
\pi_m=\frac1{n_m}\sum_{i\in I_m}y_i
$$

로 쓰면 $b_m=\mathbf1[\pi_m>0]$이다. 집합 $a$의 산모 단위 이상 비율과 구간 단위 이상 비율은

$$
\boxed{
\begin{aligned}
\beta_a
&=\frac1{|M_a|}\sum_{m\in M_a}\mathbf1[\pi_m>0],\\
\pi_a
&=\frac{\sum_{m\in M_a}n_m\pi_m}
        {\sum_{m\in M_a}n_m}
=\frac1{|I_a|}\sum_{i\in I_a}y_i.
\end{aligned}
}
$$

$\beta_a$는 이상 구간이 있는 산모를 한 명씩 센다. $\pi_a$는 산모별 이상 비율을 구간 수로 가중한 평균이다. 층화는 $\beta_a$가 비슷해지도록 하지만 $\pi_a$까지 같게 만들지는 않는다. 산모 수의 $80:10:10$ 비율도 기록 수나 구간 수에 그대로 적용되지 않는다.

## 4. 배정의 사용 범위

모든 구간이 배정되고 산모가 집합 사이에 겹치지 않아야 한다. 또한 각 집합에는 정상·이상 구간이 모두 있어야 한다.

$$
\{y_i:i\in I_a\}=\{0,1\},
\qquad a\in\{\mathrm{tr},\mathrm{va},\mathrm{te}\}.
$$

이 조건을 만족한 배정에서 각 집합의 역할은 다음과 같다.

$$
\begin{aligned}
D_{\mathrm{tr}}&\longrightarrow
\text{모델 및 자료에서 추정하는 전처리의 적합},\\
D_{\mathrm{va}}&\longrightarrow
\text{후보 선택, 조기중단, 확률 보정, 경보 기준 결정},\\
D_{\mathrm{te}}&\longrightarrow
\text{선택을 고정한 뒤 평가}.
\end{aligned}
$$

시험 결과를 보고 선택을 바꾸면 시험 집합도 개발에 사용한 것이 된다. [02](02-waveforms-and-measurements.md)의 파형 측정은 구간별 $g(m_i)$를 이어받으며, 구간이 추가로 제외되면 남은 각 집합의 클래스 조건을 다시 확인한다.

기본 모델 비교는 이 $g$를 공유한다. [04](04-combined-models.md)의 개발 집합 교차검증과 [07](07-site-transfer.md)의 기관 이동 평가에서는 산모를 묶은 채 배정을 새로 구성한다. [08](08-reading-and-birth-outcomes.md)의 분만 결과 분석은 관측 단위와 표적도 바꾼다.

구현과 저장 형식은 [report 01](../report/01-data-contract.md), 원문은 [01 노트북](../original/01-데이터-계약.ipynb)에 있다.
