# 00 · 관측 집단의 구성

하나의 관측값은 한 태아의 5분 파형과 그 구간에 대한 전문가 판독이다. 같은 산모의 반복 검사와 쌍둥이 기록을 구별하기 위해, 각 관측값에 산모·기록·기관 식별자를 함께 둔다.

00–09는 관측 집단, 산모 분할, 파형 측정, 예측과 평가를 차례로 다룬다. 정의와 표기는 장 사이에 이어서 사용한다. 먼저 분석 자료를 다음과 같이 구성한다.

$$
\begin{aligned}
\text{원자료}&\longrightarrow D=\{(x_i,y_i,m_i,r_i,s_i)\},\\
D&\xrightarrow{\ g\ }(D_a)_a
\xrightarrow{\ P,\Phi\ }(D_a^\Phi)_a,
\qquad a\in\{\mathrm{tr},\mathrm{va},\mathrm{te}\}.
\end{aligned}
$$

이 장은 분석 자료 $D$를 정의한다. [01](01-data-contract.md)은 산모 배정 $g$를, [02](02-waveforms-and-measurements.md)는 전처리 $P$와 측정 $\Phi$를 정의한다. $D_a^\Phi$는 분할 $a$의 파형을 측정값으로 바꾼 자료이며, 판독과 식별자를 보존한다.

이하 결과 수치는 `original/` 노트북에 저장된 실행에서 가져온다. 문서 작성 중 모델을 다시 학습한 결과는 아니다. 파일 처리와 오류 복구는 `report/`와 원문 코드에 둔다.

## 1. 기록에서 구간으로

### 1.1 원자료의 결합

확장자를 제외한 파일명을 기록 식별자 $r$라 하자. 같은 $r$의 주석·레이블·EMR 객체를 각각 $A_r,L_r,E_r$로 둔다. 이들은 `annotation_person`, `labels`, `emr`에서 가져온다.

세 객체가 모두 있는 기록만 결합하며, 레이블과 EMR의 내부 ID는 파일명과 같아야 한다.

$$
\operatorname{str}(L_r[\texttt{ID}])
=r
=\operatorname{str}(E_r[\texttt{ID}]).
$$

주석은 파일명으로 연결하며 `A_r["ID"]`는 별도로 검증하지 않는다. 같은 종류·같은 $r$의 중복 파일은 해석한 내용이 같으면 하나로 합친다. 레이블의 나머지 내용이 같고 `Bbox`만 한쪽에 있으면 상자가 있는 쪽을 택한다. 내용이 충돌하거나 세 객체가 완비되지 않은 기록은 제외한다.

산모 $m_r$는 `E_r["Mother.de-identification_ID"]`를 문자열로 바꾸고 앞뒤 공백을 제거한 값이다. 선행 0은 보존한다. 소문자로 바꾼 값이 `""`, `"9999"`, `"9999.0"`, `"none"`, `"nan"`, `"null"`, `"<na>"`이면 제외한다. 기관 $s_r$는 `A_r["code"]`의 첫 밑줄 앞부분에서 공백을 제거한 값이며, 빈 값은 허용하지 않는다.

### 1.2 태아와 판독의 대응

`A_r["twins"]`를 소문자 문자열로 바꾸어 `"false"`이면 `A_r["data"]`의 모든 구간을 선택한다. `"true"`이면 $r$의 마지막 밑줄 뒤 태아 번호 $f\in\{1,2\}$에 따라, 구간 객체의 `partial_image`에 독립된 `twin`$f$ 표지가 있는 항목만 선택한다. 그 밖의 쌍둥이 표기는 허용하지 않으며, 태아 표지는 대소문자를 구별하지 않는다.

선택한 객체를 원래 순서대로 $U_{r0},\ldots,U_{r,N_r-1}$라 하자. $N_r>0$이어야 하며, `L_r["Abnormality"]`를 밑줄로 나눈 이진값의 수는 $N_r$와 같아야 한다. 같은 위치의 객체와 판독을 연결한다.

$$
\begin{aligned}
F_{rj}&=\operatorname{parse}(U_{rj}[\texttt{fhr}]),
&T_{rj}&=\operatorname{parse}(U_{rj}[\texttt{toco}]),\\
x_{rj}&=\begin{bmatrix}F_{rj}\\T_{rj}\end{bmatrix}
\in\mathbb R^{2\times150},
&y_{rj}&=\operatorname{int}\!\left(
\operatorname{split}_{\texttt{\_}}(L_r[\texttt{Abnormality}])_j
\right)\in\{0,1\}.
\end{aligned}
$$

`parse`는 쉼표로 구분된 문자열 또는 수치 목록을 배열로 읽는다. $y_{rj}=1$은 이상 판독이다. 이미지 파일명으로 다시 정렬하거나, 출생 결과로 이 표적을 바꾸지 않는다.

각 채널은 2초 간격의 값 150개이다. 이하 측정값 위치는 $t=0,\ldots,149$로 센다. FHR의 단위는 bpm이고 TOCO는 원본 척도를 유지한다.

## 2. 분석에 남길 관측값

### 2.1 기록의 유효성

기록의 결합·식별자·태아 선택·판독 대응이 유효하고, 선택한 모든 구간이 다음 조건을 만족해야 한다.

$$
\operatorname{shape}(x_{rj})=(2,150),
\qquad
F_{rjt},T_{rjt}\in\mathbb R,
\qquad
F_{rjt}\ge0,
\qquad
U_{rj}[\texttt{interval\_sec}]=2\text{ s}.
$$

여기서 신호값은 모두 유한하고 측정 간격은 수치로 변환해 비교한다. 한 구간의 구조가 잘못되어도 기록 전체를 제외한다. TOCO의 음수나 추가적인 생리적 범위는 검사하지 않는다.

`L_r["Bbox"]`는 이상 판독 위치를 확인하는 보조 정보이다. 제공된 상자 $B=(x_1,y_1,x_2,y_2)$와 그 항목의 이미지 크기 $(W,H)$에 대해

$$
w=\frac{W}{N_r},
\qquad
k=\operatorname{round}(x_1/w),
\qquad
\left\|B-(kw,0,(k+1)w,H)\right\|_\infty\le0.5
$$

를 요구한다. 좌표는 유한하고 $W,H>0$이어야 하며, 상자에서 얻은 $k$의 정렬된 목록은 $\{j:y_{rj}=1\}$의 오름차순 목록과 같아야 한다. 따라서 상자의 중복·누락도 허용하지 않는다.

상자가 제공되었는데 이 조건을 어기면 기록 전체를 제외한다. 상자가 없거나 비어 있으면 이상 판독이 있어도 기록을 유지할 수 있다. 크기는 JSON에 적힌 값을 사용하며 실제 이미지를 읽지 않는다.

### 2.2 구간의 결측률

FHR의 0은 결측값으로 센다. $\mathbf1[Q]$는 조건 $Q$의 지시함수이다.

$$
q_{rj}=\frac1{150}\sum_{t=0}^{149}\mathbf1[F_{rjt}=0],
\qquad
K_r=\{j:0\le j<N_r,\ q_{rj}\le0.30\}.
$$

이 조건을 넘는 구간만 제외한다. 보간과 평활화는 뒤에서 수행하므로, 남긴 구간 안의 0은 보존한다.

기록 검증을 통과하고 $K_r\ne\varnothing$인 기록들의 집합을 $\mathcal R$라 하자. 분석 데이터는

$$
\boxed{
D=
\bigl\{
(x_{rj},y_{rj},m_r,r,s_r)
\bigr\}_{r\in\mathcal R,\ j\in K_r}.
}
$$

이다. $D$는 구간 식별자 $(r,j)$를 보존하는 관측값들의 모음이다. 파형과 판독이 같아도 서로 다른 구간은 별개의 관측값이다. 기록의 구조 검증은 기록을 채택할지 결정하고, $K_r$는 그 기록에서 사용할 구간을 결정한다.

남은 구간을 기록 ID 순서와 기록 내 선택 순서대로 열거해 전역 번호 $i=0,\ldots,n-1$을 붙인다.

$$
i\longleftrightarrow(r,j),
\qquad
(x_i,y_i,m_i,r_i,s_i)
=(x_{rj},y_{rj},m_r,r,s_r).
$$

파일명의 `_p` 뒤 번호는 해당 접두사 안의 이미지 구간 번호이다. $j$는 태아 선택 후·결측 제외 전의 순서(`seg_idx`), $i$는 남긴 신호 전체의 행 번호(`signal_index`)이다. 구간을 제외해도 $j$를 다시 매기지 않는다.

## 3. 원래 판독의 기록 요약

기록의 이상 여부 $v_r$와 이상 비율 $a_r$는 결측률로 제외하기 전의 판독 전체에서 계산한다.

$$
v_r=\max_{0\le j<N_r}y_{rj},
\qquad
a_r=\frac1{N_r}\sum_{j=0}^{N_r-1}y_{rj}.
$$

$1-v_r$는 원래 판독이 모두 정상인 기록의 표시이다. 남긴 구간의 레이블이 모두 0이더라도 $v_r=1$일 수 있다.

최장 연속 이상 길이는 구간 객체의 `partial_image`에서 정한다. 파일명 전체가 `(.+)_p(\d+)(?:\.png)?`에 대응할 때 접두사와 뒤의 정수를 각각 $\eta_{rj},p_{rj}$라 하자. 패턴은 대소문자를 구별하지 않고 읽되, 추출된 접두사는 원문 그대로 비교한다.

$$
\begin{aligned}
d_{r0}&=0,\\
d_{rj}&=\mathbf1[
\eta_{rj}=\eta_{r,j-1}
\ \land\ p_{rj}=p_{r,j-1}+1],\qquad j\ge1,\\[2pt]
\lambda_{r,-1}&=0,\\
\lambda_{rj}&=y_{rj}\bigl(1+d_{rj}\lambda_{r,j-1}\bigr),
\qquad
\ell_r=\max_{0\le j<N_r}\lambda_{rj}.
\end{aligned}
$$

정상 판독이면 길이를 0으로 되돌리고, 이상 판독이면 파일명이 인접한 경우에만 이전 길이를 이어받는다. $\ell_r$의 단위는 구간 수이다. 이 계산도 $K_r$로 제한하지 않는다. 파일명을 하나라도 해석할 수 없으면 $\ell_r$는 결측이며, 파일명 인접성만으로 실제 시간의 연속성이 보장되지는 않는다.

## 4. 기록에 부속된 변수

구간 파형과 별개로, 각 기록에 임상변수 $c_r$, 전문가 판독 지표 $e_r$, 출생 결과 $o_r$를 둔다. 공통 수치 변환 $\nu$는 수치로 읽을 수 있는 유한값을 유지하고, 누락·비수치·비유한값과 결측 표지 9999를 결측으로 둔다. 일반적인 임상값 결측은 구간을 제외하지 않는다.

### 4.1 임상변수

EMR의 다음 11개 값에 $\nu$를 적용하고, 산모 연령과 재태 연령을 덧붙여 $c_r\in(\mathbb R\cup\{\mathrm{NA}\})^{13}$을 만든다.

| 변수 | $E_r$의 필드 |
| --- | --- |
| 신장·체중 | `Mother.Height`, `Mother.Weight` |
| 임신 횟수·출산 횟수 | `Mother.Gravida`, `Mother.Para` |
| 수축기·이완기 혈압 | `Mother.SBP`, `Mother.DBP` |
| 임신성 고혈압·고혈압 | `Mother.GHTN`, `Mother.Hypertension` |
| 임신성 당뇨·당뇨 | `Mother.GDM`, `Mother.DM` |
| 전자간증 | `Mother.pre-eclampsia` |

원자료의 수치·범주 코드와 단위는 유지한다. 아이와 산모의 출생 연월은 각각 `E_r["Birth Date"]`, `E_r["Mother.Birth Date"]`의 `YYYYMM` 값이다.

$$
\begin{aligned}
\operatorname{age}_r
&=
\left\lfloor\frac{\nu(E_r[\texttt{Birth Date}])}{100}\right\rfloor
-\left\lfloor\frac{\nu(E_r[\texttt{Mother.Birth Date}])}{100}\right\rfloor+1,\\
\operatorname{ga}_r
&=\nu(E_r[\texttt{GA.wks}])+\frac{\delta_r}{7},\\
\delta_r
&=
\begin{cases}
\nu(E_r[\texttt{GA.day}]),&\text{해당 키가 있으면},\\
0,&\text{해당 키가 없으면}.
\end{cases}
\end{aligned}
$$

계산에 필요한 값이 결측이면 파생값도 결측으로 둔다. 산모 연령은 연도 차이에 1을 더한 값이다. 재태 연령은 주 단위이며, `GA.day`가 존재하면서 결측이면 재태 연령도 결측이다.

### 4.2 전문가 판독과 출생 결과

$e_r$는 $L_r$의 다음 일곱 필드에 $\nu$를 적용한 벡터이다.

| 지표 | $L_r$의 필드 |
| --- | --- |
| 기저선·기저선 변이도 | `BaseLine`, `Baseline_Variability` |
| 가속 | `Acceleration` |
| 조기·후기 감속 | `Early_deceleration`, `Late_deceleration` |
| 변이·지속성 감속 | `Variable_deceleration`, `Prolonged_deceleration` |

수치와 범주 코드는 그대로 옮긴다.

출생 결과는 EMR에서 얻는다.

$$
o_r=
\bigl(
\nu(E_r[\texttt{UA.pH}]),
\nu(E_r[\texttt{APGAR.1min}]),
\nu(E_r[\texttt{APGAR.5min}])
\bigr).
$$

따라서 구간 자료와 함께 다음 기록 자료를 정의할 수 있다.

$$
h_r=(a_r,\ell_r),
\qquad
\boxed{
D_{\mathrm{record}}
=\{(r,c_r,e_r,h_r,o_r,m_r,s_r):r\in\mathcal R\}.
}
$$

두 자료는 기록 식별자 $r$로 연결된다. $h_r$는 실제 구간 판독의 요약이고, $o_r$는 관측된 출생 결과이다. [08](08-reading-and-birth-outcomes.md)은 임상정보를 더한 구간 예측에 $(z_i,c_{r_i})$를, 출생 결과 예측에 기록 단위 $(c_r,h_r)$를 사용한다. $z_i$는 02에서 정의할 파형 특징이다.

원자료 결합과 검증의 구현 세부사항은 [report 10](../report/10-input-organization.md)과 [report 00](../report/00-data-preparation.md)에 둔다. 다음 [01](01-data-contract.md)은 여기서 남긴 $D$의 산모들을 나눈다.
