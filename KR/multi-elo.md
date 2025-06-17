🔍 PCA vs Multi-ELO 비교 분석

  PCA (Principal Component Analysis)

  # 데이터 기반 자동 분해
  components = PCA(n_components=4).fit([
      [avg, obp, slg, ops] for player in all_players
  ])

  # 결과: 수학적으로 최적화된 성분들
  PC1: 0.8*SLG + 0.6*OPS + 0.3*AVG + 0.1*OBP  # "총 타격력"
  PC2: 0.7*AVG + 0.5*OBP - 0.2*SLG - 0.1*OPS  # "접촉 vs 장타"
  PC3: 0.6*OBP - 0.4*AVG + 0.3*SLG + 0.1*OPS  # "선구안"
  PC4: ...  # 잔여 성분

  장점:
  - 데이터가 알려주는 "진짜" 독립적 성분
  - 수학적으로 최적화된 분해
  - 차원 축소 효과

  단점:
  - 해석이 어려움 (PC2가 정확히 뭘 의미하는지?)
  - 야구 도메인 지식과 연결 어려움
  - ELO 업데이트 로직이 복잡해짐

  Multi-ELO System

  # 도메인 기반 의미있는 분해
  elo_ratings = {
      'contact_elo': 1000,    # 명확히 "접촉 능력"
      'power_elo': 1000,      # 명확히 "장타 능력"  
      'discipline_elo': 1000, # 명확히 "선구안"
      'speed_elo': 1000       # 명확히 "발야구"
  }

  # 타석 결과별 직관적 업데이트
  if result == '안타':
      contact_elo += big_update
      power_elo += small_update
  elif result == '홈런':
      power_elo += big_update
      contact_elo += medium_update

  장점:
  - 야구적으로 해석 가능
  - 선수 스카우팅에 직접 활용 가능
  - ELO 업데이트 로직이 직관적
  - 각 능력별 추적 가능

  단점:
  - 능력간 상관관계 무시할 수 있음
  - 주관적 분해 (정말 최적인가?)

  💡 그렇다면 어떤 게 더 나을까?

  저의 추천: Hybrid Approach

  # 1단계: PCA로 데이터가 말하는 진짜 성분 발견
  pca_components = analyze_true_components(historical_data)

  # 2단계: PCA 결과를 야구 도메인 지식으로 해석
  interpreted_components = {
      'pure_hitting': pca_components[0],    # PC1 = 순수 타격력
      'contact_vs_power': pca_components[1], # PC2 = 접촉 vs 장타 트레이드오프
      'plate_discipline': pca_components[2], # PC3 = 선구안
      'situational': pca_components[3]       # PC4 = 상황 대응력
  }

  # 3단계: 해석된 성분들로 Multi-ELO 구성
  for component_name, loadings in interpreted_components.items():
      component_elo[component_name] = update_based_on_loadings(pa_result, loadings)

  🎯 실제로는 더 근본적인 문제가 있을 수 있습니다

  가설 1: "타석 단위"의 한계

  # 현재: 각 타석을 독립적으로 처리
  for pa in plate_appearances:
      update_elo(pa.result)

  # 개선: 연속된 타석들의 패턴 분석
  for game in games:
      game_performance = analyze_game_pattern(game.all_pas)
      update_elo_with_context(game_performance)

  가설 2: "순위 기반 매칭"의 한계

  # 현재: ELO 순위 vs KBO 통계 순위로 매칭
  elo_rank_1 ↔ ops_rank_1
  elo_rank_2 ↔ ops_rank_2

  # 개선: 실제 player_id 기반 직접 매칭 (가능하다면)
  player_A_elo ↔ player_A_real_avg
  player_B_elo ↔ player_B_real_avg

  가설 3: "단일 시즌"의 한계

  AVG/OBP는 SLG보다 더 많은 샘플이 필요한 안정적인 지표일 수 있습니다.

  🚀 제안하는 실험 순서

  1. 먼저 PCA 분석: 실제 데이터가 어떤 성분들로 구성되는지 확인
  2. Multi-ELO 구현: PCA 결과를 참고해서 의미있는 능력별 ELO 설계
  3. Direct Matching: 가능하다면 순위가 아닌 직접 매칭 시도
  4. Temporal Analysis: 더 긴 기간 데이터로 안정성 확인
