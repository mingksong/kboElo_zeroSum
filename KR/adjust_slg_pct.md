⏺ ELO 계산 과정에서 가중치 팩터 적용 방법

  V3.2 ELO 시스템에서 가중치 팩터들이 적용되는 과정을 단계별로 설명드리겠습니다:

  1. 가중치 팩터 종류

  - power_weight: 장타력 보너스 (홈런, 3루타, 2루타)
  - contact_weight: 접촉 능력 보너스 (안타)
  - clutch_weight: 클러치 상황 보너스 (득점 상황)

  2. 적용 과정 (v32_proper_calculator.py:107-125)

  def calculate_power_enhancement(self, result_type: str, runs_scored: int) -> float:
      """장타력 강화 팩터 계산"""
      enhancement = 1.0  # 기본값

      # 장타력 보너스
      if result_type == '홈런':
          enhancement += self.power_weight * 1.5    # 홈런: 1.5배 보너스
      elif result_type == '3루타':
          enhancement += self.power_weight * 1.2    # 3루타: 1.2배 보너스
      elif result_type == '2루타':
          enhancement += self.power_weight * 0.8    # 2루타: 0.8배 보너스
      elif result_type in ['안타']:
          enhancement += self.contact_weight         # 안타: contact_weight 적용

      # 클러치 보너스 (득점 상황)
      if runs_scored > 0:
          enhancement += self.clutch_weight * min(runs_scored, 3) / 3  # 최대 3득점까지
          
      return enhancement
      # 4점이 아닌 이유는, 주자는 3명까지만 나갈수 있으며, 클러치 상황이라 함은 '주자가 있는 상황을 뜻함' 
      # 즉 4점의 득점일 경우 1점은 '나자신의 득점'이므로 클러치의 정의에 위배됨 
  3. ELO 계산 메인 프로세스 (v32_proper_calculator.py:161-178)

  # 1단계: 실제 vs 예상 run value 차이 계산
  rv_diff = actual_rv - expected_rv

  # 2단계: 가중치 팩터 적용
  power_enhancement = self.calculate_power_enhancement(result_type, runs_scored)

  # 3단계: 차이값에 가중치 곱하기 ✨핵심✨
  adjusted_diff = rv_diff * power_enhancement

  # 4단계: tanh 정규화
  normalized_diff = np.tanh(adjusted_diff / self.scale_factor)

  # 5단계: K팩터 적용하여 최종 ELO 변화량 계산
  batter_change = batter_k * normalized_diff
  pitcher_change = -pitcher_k * normalized_diff  # 제로섬

  4. 실제 예시

  최적화 결과에서 나온 최고 파라미터:
  - power_weight: 0.4
  - contact_weight: 0.05
  - clutch_weight: 0.1

  홈런 + 2득점 상황:
  enhancement = 1.0 + (0.4 * 1.5) + (0.1 * 2/3) = 1.0 + 0.6 + 0.067 = 1.667
  adjusted_diff = rv_diff * 1.667  # 원래 차이값이 66.7% 증폭

  안타 + 1득점 상황:
  enhancement = 1.0 + 0.05 + (0.1 * 1/3) = 1.0 + 0.05 + 0.033 = 1.083
  adjusted_diff = rv_diff * 1.083  # 원래 차이값이 8.3% 증폭

  핵심 포인트

  가중치는 168번째 라인에서 adjusted_diff = rv_diff * power_enhancement로 적용되어, 좋은 결과(홈런, 클러치 상황)일 때 ELO 변화량을 더
  크게 만드는 역할을 합니다.
