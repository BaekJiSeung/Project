이 코드의 목적은 감염병(measles)의 혈청 유병률(seroprevalence) 데이터를 분석하여 감염력(force of infection, FOI)을 추정하는 것이다. FOI는 질병이 특정 인구 내에서 전파되는 속도를 나타내는 매개변수이다. 이를 통해 감염병의 전파 특성을 이해하고, 감염 확산을 예측하고 예방하는 데 도움이 된다.

모티베이션
FOI는 감염병의 전파를 이해하고 예측하는 것에 핵심적인 역할을 하며, 특정 연령대에서 감염이 발생하는 비율을 파악하는 데 사용된다. 
혈청 유병률 데이터는 특정 연령대에서 얼마나 많은 사람들이 항체를 보유하고 있는지를 나타내며, 이는 과거 감염 여부를 간접적으로 알 수 있다.
이 코드의 목적은 혈청 유병률 데이터를 기반으로 FOI를 추정, 그로부터 전염율(transmission rate)을 계산하여 감염병의 전파 동향을 이해하고 예측하는 데에 도움을 주는 것이다.

데이터
이 코드에서는 두 개의 CSV 파일(seroprevalence_uk.csv 및 seroprevalence_china.csv)을 사용한다. 
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
Catalytic Model을 이용해 오차를 최소화하는 λ 값을 찾기 위해 scipy.optimize 모듈의 minimize 함수를 사용한다.
Susceptible(감염가능성있음) 에서 Ever Infected 로 가는 비율을 λ로 계산하여 실제 데이터의 그래프와 가장 적은 오차를 가지는 foi 값을 구할 수 있다.
이 과정에서 주어진 초기 추정치(initial_guess)를 사용하여 최적화를 한다.

코드 상세 설명
우선 클래스에 연령대(time_stamp), 혈청 유병률 데이터(data), 초기 추정치(initial_guess)를 input으로 받아서 infectious를 추정할 수 있는 메소드인 get_seroprev_from_foi 메소드를 입력한다.
그리고 주어진 자료와 최소한의 오차를 가질수 있게 최적화할 수 있는 objective_function 메소드를 입력한다.

이 두 는 초기 추정치를 기반으로 예상 혈청 유병률(expected)을 계산하고, 실제 데이터와의 오차(error)를 계산, 최적화된 FOI(est)를 찾기 위해 least square method를 통해 오차를 최소화한다.
초기 추정치와 최적화된 추정치를 시각적으로 비교하기 위해 그래프를 통해 데이터를 시각화한다.
CSV 파일로부터 데이터를 읽어와 혈청 유병률을 계산하고, 이를 기반으로 FOI를 추정한다.
각 분석 결과를 출력하여 초기 오차, 최적화된 FOI, 최종 오차를 확인한다.
UK 데이터와 중국 데이터 분석
코드는 UK 데이터와 중국 데이터를 각각 분석합니다. 각 데이터 세트에 대해 초기 추정치와 최적화된 FOI를 계산하고, 이를 비교하여 각 지역에서의 감염병 전파 특성을 파악합니다. 또한, UK 데이터를 두 부분으로 나누어 각각에 대해 분석을 수행함으로써 데이터의 시간적 변화에 따른 FOI의 변화를 관찰할 수 있습니다.

결론
이 코드의 궁극적인 목표는 혈청 유병률 데이터를 통해 감염병의 전파 속도를 추정하고, 이를 통해 공중 보건 전략을 수립하는 데 필요한 정보를 제공하는 것입니다. FOI의 정확한 추정은 감염병의 동향을 예측하고 적절한 대응책을 마련하는 데 중요한 역할을 합니다.
