---
id: "Tidy First? A Daily Exercise in Empirical Design • Kent Beck • GOTO 2024"
aliases: "정리부터? 경험적 설계의 일일 연습 • 켄트 벡 • GOTO 2024"
tags:
  - software-design/coupling/propagation-of-change
  - software-design/structure-vs-features/investment-balance
  - agile/xp/tidy-first
  - economics/npv/optionality
  - refactoring/parallel-change/interruptible-design
  - clean-code/communication-through-code/technical-debt
  - architecture/growth/differentiation-unification
  - team/social-trust/authority-responsibility
  - risk/power-law/large-change-costs
author: kent-beck
created_at: '2025-09-29 00:28'
related: []
source: GOTO Conferences (url://74)
---

# Tidy First? A Daily Exercise in Empirical Design

## 1. 3줄 요약 (3-line summary)

소프트웨어 설계는 인간관계의 문제이며, 기능(Features)과 구조(Structure) 투자 간 균형이 핵심이다.

결합도(coupling)의 본질은 “한 요소를 바꾸면 다른 요소도 바꿔야 하는가”이며, 대규모 변경 비용은 변화의 전파로부터 발생한다.

경제적 제약(Net Present Value, Optionality, Survival)을 고려한 “중단 가능(Interruptible) 병렬 변경(parallel change)” 설계가 지속 가능성을 높인다.

## 2. 하이라이트/개요 (Highlights/Summary)

본 강연은 경험적 소프트웨어 설계(Empirical Software Design) 관점에서, 기능과 구조의 상호작용이 실제 변경 비용을 어떻게 비선형적으로 증폭시키는지 설명한다. 결합도(coupling) 를 “특정 변경에 대해 한 요소의 변경이 다른 요소의 변경을 필요로 하는 관계”로 엄밀히 정의하고, IDE 지원/외부 API 등 맥락에 따라 결합도의 실제 비용이 달라짐을 계층적으로 해부한다.

경제적 제약은 세 가지로 정리된다. 첫째 생존(Survival): 초기 탐험/확장 단계에서 실패의 기본 상태를 넘어서 지속 가능성 임계값을 통과해야 한다. 둘째 현재가치(Net Present Value): “더 빨리 벌기=더 많이 벌기”이며, 비용을 미래로 미루는 설계는 경제적으로 가치 창출이다. 셋째 선택권(Optionality): 구조에 대한 투자는 미래 변화 가능성을 확장하여 가치가 커진다(환경 변동성이 클수록 선택권 가치 증가).

조직·문화 측면에서는 신뢰, 권한-책임 정렬, 상이한 인센티브(Product vs Engineering) 조율이 필요하다. 해법으로 중단 가능 병렬 변경(parallels), 미시적 수준에서의 기능/구조 변경의 절대적 분리, 나무처럼 성장하는 분화/중복 시 통합을 제시한다.

## 3. 상세 요약 (Detailed Summary)

### Intro/XP의 발상 (Extreme Programming)

- 극단적 프로그래밍(Extreme Programming, XP) 은 “가치 없는 활동을 최소화하고, 본질적 활동만 최적화”하는 접근에서 출발한다. 엔지니어링은 항상 트레이드오프의 문제이며, 목표는 “낭비를 0으로”가 아니라 “낭비의 최소화”다.
- 문서화(Documentation) 의 대량 생산은 실사용이 없을 경우 시간 낭비이며, 오늘날에도 “항시 최신 상태 유지” 시도는 눈덩이처럼 작업이 폭증해 지속 불가능하다는 지적이 반복된다.

### Tidy First?의 문제 정의

- 정리부터(Tidy First?) 는 “작업하려는 코드가 엉망일 때, 먼저 정리할 것인가?”라는 미시적 설계 의사결정의 최소 단위다. 답은 상황 의존적이며, “대개 예(과도한 정리는 금물)”이라는 실천적 규칙을 제시한다.
- 자기와의 관계(Self-care) 로서 작은 도우미 추출 등으로 작업 난이도를 낮추는 행위는 타당하며, 다음 단계로 팀 간 함께 정리(Tidy Together) 가 필요해진다(대인관계로 확장).

[불확실성: 책/시리즈 상세 구성은 강연 맥락상 예고 수준]

### 고수준 개발 프로세스: 아이디어-기능-구조 상호작용

- 설계(Design) 의 목적은 “변경 가능성(변화 용이성)”이며, 시스템 외부에서 관찰되는 행동 변경이 요구될 때 설계가 중요해진다.
- 구조(Structure) 는 기능 구현 비용에 큰 영향을 미치며, 많은 경우 “먼저 구조를 바꿔서 기능을 쉽게 만든다(정리부터 흐름)”가 합리적이다.
- 가시성(Governance): 기능은 모든 이해관계자에게 가시적이지만 구조 투자는 간접적 관찰만 가능해 투자 부족의 원인이 된다(속도 저하/결함 증가로 늦게 표출).

### 결합도와 대규모 변경 비용 (Coupling & Cost of Change)

- 결합도(coupling): “특정 변경에 대해 한 요소를 바꾸면 다른 요소도 바꿔야 하는 관계”로 정의되며, 변경의 전파가 비용의 본질이다.
- 맥락 의존성(Context-dependence): 이름 변경은 IDE 리팩토링 지원 시 결합도가 사실상 제거되지만, 외부 고객이 있는 API 명세 변경은 고비용 외부 결합을 야기한다.
- 파워 법칙(Power-law)적 비용 분포: 대부분 변경은 작지만, 드물게 폭발적 연쇄 변경(눈사태)이 발생한다. 이 연쇄는 결합된 요소들의 계층적 의존으로 증폭된다.

### 경제적 제약 (Economics: Survival, NPV, Optionality)

- 생존(Survival): 초기 단계(탐험/확장)는 기본 실패 상태이며, 지속 가능성 임계값을 넘겨야 장기 존속이 가능하다.
- 현재가치(Net Present Value, NPV): “더 빨리 벌기=더 많이 벌기”, “나중에 쓰기=덜 쓰기”이므로, 설계 결정을 미래로 미루면 가치 창출이다. 과도한 선행 설계(Big Design Up Front)는 이 경제 원리를 무시해 비효율적이다.
- 선택권(Optionality): 구조에 대한 투자는 미래 기능 변화의 선택지를 늘려 자산 가치를 높인다. 변동성이 클수록 선택권 가치가 커진다.

### 함의: 성장/활동/병렬/분리

- 나무처럼 성장(Tree-like growth): 분화(분할·확장)가 기본이며, 중복 발생 시 통합(Unification)이 병행된다.
- 활동으로서의 설계(Design as activity): 단계가 아닌 지속 활동이며, 기능과 구조 투자는 교차·균형되어야 한다. “6개월 정지 후 설계”는 관계·경제 현실을 무시한다.
- 병렬 변경(Parallel change): 구형과 신형 설계를 일정 기간 공존시켜, 언제든 기능 개발로 중단·재개 가능한 중단 가능(Interruptible) 상태를 유지한다(파라미터 타입 교체의 미시적 예시).
- 미시적 분리(Micro-separation): 동일 커밋/변경에서 기능 변경과 구조 변경을 혼합하지 않고 절대 분리함으로써, “기능은 계속 증가, 설계는 꾸준히 개선”이라는 환상을 유지한다.

### 사회적 제약과 리더십

- 신뢰/관계(Trust/Relationships): 권한과 책임을 정렬하여, 타인에게 작업을 발생시키면 그 학습·변경을 함께 지원한다.
- 인센티브 불일치(Incentives): 제품(Features)과 엔지니어링(Structure)의 동기가 다르므로, 다음 단계에 대한 합의를 형성해야 한다.
- 압박 vs 목적(Pressure vs Purpose): 압박으로는 창의적 설계를 이끌 수 없으며, 목적 중심 리더십이 변화의 추진력과 창의 공간을 동시에 제공한다.

## 4. 결론 및 전문가 견해 (Conclusion and Professional View)

- 결합도(coupling)를 “변경 전파의 존재”로 정의하고, 변경 비용 최소화의 1차 대상임을 재확인한다.
- 기능/구조의 투자 균형을 유지하는 경험적 설계가 지속 가능성을 좌우한다.
- NPV 관점에서 선행 설계의 과대 투자보다 “결정의 연기”가 더 큰 경제적 가치를 낳는다.
- 선택권(Optionality)을 확장하는 구조 투자는 환경 불확실성에서 전략적 우위를 제공한다.
- 병렬 변경(parallel change)로 중단 가능성을 확보하고, 미시적 수준에서 기능/구조 변경을 절대 분리한다.
- 팀 차원에서는 신뢰·권한-책임 정렬·인센티브 조율이 필수이며, 압박보다 목적 기반 리더십이 유효하다.
- OOAD/아키텍처 측면에서, 분화-통합의 성장 모델과 구조적 투자 타이밍은 모놀리식/모듈리스/마이크로서비스 등 어떤 형태에도 적용 가능한 상위 원리로 작동한다.

[불확실성: 아키텍처 유형의 구체적 비교는 강연에 직접 제시되지 않음]

## 5. 불확실성 지도 (Uncertainty Map)

- **원본 모호성 (Source Ambiguity):**
  - “Tidy Together” 및 3부작 계획은 개념적 예고 수준으로, 범위·방법론·증거 수준이 제한적이다.
  - 파워 법칙 분포에 대한 수학적 상세는 질문 유도만 있고 강연 내 엄밀한 정량 근거는 생략된다.
- **판단 근거 (Judgment Calls):**
  - 섹션 구분을 챕터/주요 논점으로 재구성하여 결합도·경제 제약·병렬 변경을 중심 축으로 배치하였다(기능/구조 균형의 논리적 선후 강화).
  - 실례(페이롤 계산기 중복→통합→마케팅 옵션)는 선택권 가치를 설명하는 대표 사례로 우선 제시했다.
- **해석 불확실성 (Interpretive Uncertainty):**
  - “경험적 설계”가 측정·피드백 중심의 실험적 방법론인지, 혹은 경제/사회 제약을 반영한 실천적 프레임인지 경계가 명확히 그려지지 않는다.
  - “문서화는 해법이 아니다”는 주장 범위가 어떤 도메인/규모/규정 준수 환경까지 포괄하는지 해석의 여지가 있다.
- **단순화/추상화 영역 (Simplification/Abstraction Areas):**
  - 결합도의 다층성(동적 의존·계약 안정성·조직 경계)을 “변경 전파”로 수렴 요약하였다.
  - 경제 제약(NPV/Optionality/Survival)의 정량 모델은 강연 수준에서 개념화로 추상화하였다.
- **의견 전환 질문 (Opinion-Changing Questions):**
  - 어떤 계측 지표로 기능/구조 투자 균형을 실시간 합의할 수 있는가(속도·결함·옵션 가치의 공동 메트릭 설계)?
  - 병렬 변경의 비용 상한과 조직 수용성은 어느 범위에서 경제적으로 정당화되는가(기술 부채/복잡성 증가의 임계점)?
