이 코드의 목적은 감염병(measles)의 혈청 유병률(seroprevalence) 데이터를 분석하여 감염력(force of infection, FOI)을 추정하는 것이다. FOI는 질병이 특정 인구 내에서 전파되는 속도를 나타내는 매개변수이다. 이를 통해 감염병의 전파 특성을 이해하고, 감염 확산을 예측하고 예방하는 데 도움이 된다.

모티베이션

FOI는 감염병의 전파를 이해하고 예측하는 것에 핵심적인 역할을 하며, 특정 연령대에서 감염이 발생하는 비율을 파악하는 데 사용된다. 
혈청 유병률 데이터는 특정 연령대에서 얼마나 많은 사람들이 항체를 보유하고 있는지를 나타내며, 이는 과거 감염 여부를 간접적으로 알 수 있다.
이 코드의 목적은 혈청 유병률 데이터를 기반으로 FOI를 추정, 그로부터 전염율(transmission rate)을 계산하여 감염병의 전파 동향을 이해하고 예측하는 데에 도움을 주는 것이다.



데이터

이 코드에서는 연령에 따른 혈청유병률의 데이터인 두 개의 CSV 파일(seroprevalence_uk.csv 및 seroprevalence_china.csv)과 measles의 시간 경과에 따른 감염자수를 나타내는 'incidence_measles.csv' 을 사용한다. 
이 파일들은 각기 다른 지역(영국과 중국)에서 수집된 혈청 유병률 데이터를 포함하고 있다. 
영국은 중국에 비해 더 많은 표본을 가지고 있다.

간단히 설명하자면
time_stamp: 특정 연령대
Pa: 특정 연령대에서 양성으로 판정된 사람 수
Na: 특정 연령대에서 조사된 전체 사람 수
이 데이터를 통해 각 연령대에서 얼마만큼의 비율이 감염된 적이 있는지를 계산할 수 있다.



모델

모델은 주어진 연령대(time_stamp)에서 FOI (λ)를 추정하는 것을 목표로 다음과 같은 방식을 사용하였다.

expected = 1 - exp(-initial_guess * time_stamp): 초기 추정치(initial_guess)를 기반으로 계산된 혈청 유병률
error = sum((expected - data) ** 2): 초기 추정치와 실제 데이터 간의 오차 제곱합
Susceptible과 ever infected 사이의 관계를 나타내는 Catalytic Model을 이용해 오차를 최소화하는 λ 값을 찾기 위해 scipy.optimize 모듈의 minimize 함수를 사용한다.
Susceptible(감염가능성있음) 에서 Ever Infected 로 가는 비율을 λ로 계산하여 실제 데이터의 그래프와 가장 적은 오차를 가지는 foi 값을 구할 수 있다.
이 과정에서 주어진 초기 추정치(initial_guess)를 사용하여 최적화를 한다.

SEIR 모델은 감염병의 전파를 시뮬레이션하기 위해 사용되는 수학적 모델로 네 가지 상태로 구성된다.

S: 감염 가능성이 있는 사람들 (Susceptible)
E: 잠복기 상태의 사람들 (Exposed)
I: 감염된 사람들 (Infectious)
R: 회복된 사람들 (Recovered)
dSdt = -beta * S * I 
dEdt = beta * S * I  - sigma * E
dIdt = sigma * E - gamma * I
dRdt = gamma * I

여기서 β는 전염율, σ는 잠복기 상태에서 감염 상태로의 전환율,  γ는 회복율을 나타낸다.
홍역의 잠복기는 8일 평균 기간은 7일이므로  σ,γ는 각각 역수인 1/8, 1/7 이다.
유병률을 통해 구한 람다의 추정값으로 베타를 구할 수가 있다. 이 베타는 중요한 수인 R0로 이어진다다

기본 재생산지수 R0: 이 값은 전염병이 얼마나 점염률이 높은지 즉 1명의 환자가 몇명의 감염자를 만들어내는지를 의미한다. 값이 1보다 크면 전염병이 확산되고, 1보다 작으면 전염병이 확산하지 않는다.
집단 면역 (Hred Immunity):  특정 인구가 집단 면역을 달성하기 위해 얼마나 많은 사람이 면역이 되어야 하는지 결정하는 데 도움을 준다. 
H = 1 - 1/R0 d의 값을 가진다. 예를 들어 H = 0.8이라면 전체의 80퍼센트만 접종을 통해 면역력을 갖추면 더 이상의 전염이 일어나지 않는다는 뜻이다. 이는 예방 접종 보건 정책 수립에 유용한 정보를 제공한다. 



코드 설명

우선 클래스에 연령대(time_stamp), 혈청 유병률 데이터(data), 초기 추정치(initial_guess)를 input으로 받아서 infectious를 추정할 수 있는 메소드인 get_seroprev_from_foi 메소드를 입력한다.
그리고 주어진 자료와 최소한의 오차를 가질수 있게 최적화할 수 있는 objective_function 메소드를 입력한다.
이 두 메소드 는 초기 추정치를 기반으로 예상 혈청 유병률(expected)을 계산하고, 실제 데이터와의 오차(error)를 계산, 최적화된 FOI(est)를 찾기 위해 least square method를 통해 오차를 최소화한다.
seroprevalence_uk.csv 및 seroprevalence_china.csv의 데이터를  통해 각각의 foi과 오차를 측정한다.
초기 추정치와 최적화된 추정치를 비교하기 위해 그래프를 통해 데이터를 시각화한다.
foi를 통해 R0, H를 구하여 실질적인 전염병 해소에 도움이 되는 수치들을 계산할 수 있다.
데이터가 더 많은 uk의 경우 연령별로 데이터를 두 파트로 나누어 각각에서 다시 계산본다. 이를 통해 measles의 나이에 다른 감염률의 차이를 확인할 수 있다.
위에서 계산한 전염률을 통해 SEIR모델링을 활용하여 시간에 따른 infectious 수치를 계산, 실제 관측된 'incidence_measles.csv'데이터와 어떤 차이를 보이는지 그래프를 통해 확인한다.

퍼포먼스 

seroprevalence_uk.csv 및 seroprevalence_china.csv을 통해 uk와 중국의 expected foi(lamda)를 구하면 uk는0.122, 중국은 0.278이 나오게 된다. 이는 비감염자에서 감염자로 foi만큼의 비율로 이동한다는 것을 의미한다. 
이를 통해 감염재상산지수 R0를 구하면 uk의 경우에는 7.3, 중국은 16.7의 수치가 나오게 된다. 실제 연구결과 measles의 R0는12ㅇ서 16사이의 값을 가지는데 이를 통해 선진국인 영국의 환경과 의료수준으로 인해 중국보다 감염률이 현저히 낮은것을 확인 가능하다.
이를 통해 백신의 효율적인 보급량에 도움이 되는 H값을 계산할 수 있다. uk의 경우에는 0.86, 중국은 0.94의 수치로 uk는 국민의 86퍼센트만 면역을 갖추면 되는 반면 중국은 94퍼센트의 면역을 갖추어야 한다.
uk의 데이터를 청소년기인 15세 이전과 이후로 나누어서 각각의 데이터에 같은 모델을 통해 foi를 새로 구해보게 되면 15세 이전에서는 0.13, 이후에서는 0.103으로 나타나는것을 확인 할수 있다. 감염되는 비율이 15세 이전에서 높으므로 measeles는 나이가 적은 사람들에게 감염 확률이 높음을 알 수 있다 마찬가지로 R0은 15세 이전이 적은 수치를 가지고  H는 15세 이후에서 높은 수치를 가진다. 따라서 질병의 예방을 위해서 15세 이전의 아이들에게 백신을 접종비율을 높게 잡는 것이 효율적임을 알 수 있다.
위에서 구한 foi, R0등을 통해 SEIR모델과의 그래프로 비교를 한 결과, 대체로 실제 결과와 예측값이 유사하게 나타남을 알 수 있다. 
에측값의 피크가 실제측정의 피크보다 좀 더 앞에 나타나는 것은 실제 현실에서는 질병의 감열 여부가 즉각적으로 업데이트되지 않는 등의 이유를 생각해 볼 수 있다. 
앞서 예측한 foi, R0, H등의 값이 실제 결과와 일치하는 것을 확인 할 수 있는데 이러한 값들을 이용해 질병 감염의 대처 방안을 생각해 볼 수 있다. 
감염력이 높은 나이가 어린 사람들 위주로 청결 유지, 백신 접종 들을 통해 집단면역을 달성하는 방향으로 정책을 설립한다면 효율적으로 질병에 대해 방어할수 있을 것이다.


결론

이 코드의 궁극적인 목표는 혈청 유병률 데이터를 통해 감염병의 전파 속도를 추정하고, 이를 통해 공중 보건 전략을 수립하는 데에 필요한 정보의 제공이다. FOI,R0,H의 정확한 추정은 감염병의 확산을 예측하고 적절한 대응책을 마련하는 데 중요한 역할을 한다.
