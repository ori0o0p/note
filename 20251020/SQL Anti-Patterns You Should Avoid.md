---
id: SQL Anti-Patterns You Should Avoid
aliases: 피해야 할 SQL 안티패턴
tags:
- sql/anti-patterns/case-when
- sql/performance/index-functions
- sql/schema/views/select-star
- sql/data-quality/distinct-duplicates
- sql/architecture/view-stacking
- sql/query-design/cte
- data-engineering/maintainability/shared-logic
- analytics/join-logic/one-to-one
- data-pipelines/debuggability/nested-subqueries
author: jordan-goodman
created_at: 2025-10-20 10:51
related: []
source: datamethods.substack.com
---

## **1. 3줄 요약 (3-line summary)**

- 본문은 조직 규모가 커질수록 누적되는 **SQL 안티패턴(anti-patterns)**이 유지보수성과 성능을 악화시키며 데이터 신뢰성을 저해하는 핵심 사례들을 제시한다.
- 대표 사례로 **대형 CASE WHEN(CASE WHEN)**의 오남용, **인덱스 컬럼에 대한 함수 적용(Index on column with function)**, **뷰에서의 SELECT *(SELECT *)**, **DISTINCT로 중복 “숨기기”(DISTINCT)**, **뷰 중첩(View stacking)**, **과도한 중첩 서브쿼리(Subqueries)**가 다뤄진다.
- 해결 방향은 **공유 가능한 차원 테이블/뷰(Dimension table/view)**로 로직을 표준화하고, **인덱스를 해치지 않는 조건식(Index-aware predicates)**을 사용하며, **명시적 스키마 관리(Explicit schema management)**와 **조인 관계의 정확화(Join correctness)**, **변환의 평탄화(Materialization/flattening)**, **CTE(Common Table Expressions, CTE)** 활용을 통해 가독성과 성능을 개선하는 것이다.

## **2. 하이라이트/개요 (Highlights/Summary)**

문서는 “빠른 대시보드 제작”이나 “데드라인 압박” 같은 현실적 제약 속에서 발생하는 **반복적 SQL 안티패턴(anti-patterns)**을 체계적으로 지적한다. 특히, 특정 대시보드를 위해 뷰에만 거대한 **CASE WHEN(CASE WHEN)**을 배치하는 방식은 재사용성과 일관성을 깨뜨려, 타 개발자에게 **복붙(copy-paste)**을 강요하거나 **원시 코드(raw status codes)** 사용을 유도해 데이터 해석의 혼란을 야기한다. 동일하게, **인덱스 컬럼에 대한 함수 적용(Index + function)**은 RDBMS의 인덱스 활용을 무력화하여 **풀 스캔(full table scan)**을 유발할 수 있다. [불확실성: 대체 조건 예시는 원문에 구체적으로 제시되지 않음]

또한 뷰 개발 시 **SELECT *(SELECT )* 사용은 스키마 진화에 취약하며 불필요한 컬럼을 포함할 위험을 높인다. **DISTINCT(DISTINCT)**로 중복을 “교정”하려는 관행은 실제로 **조인 조건의 불완전성(join condition incompleteness)**이나 **1:1 관계 부재(non one-to-one)**를 가리는 임시 처방에 불과해 이후 지표 일관성을 무너뜨린다. **뷰 레이어 중첩(View stacking)**은 의존성 사슬을 복잡화하고 성능을 저하하며, **과도한 중첩 서브쿼리(Nested subqueries)**는 가독성과 디버깅 가능성을 크게 떨어뜨리므로 **CTE(CTE)**를 통한 가독성 향상이 권고된다. 최종적으로, 팀은 SQL을 **프로덕션 코드(production code)**처럼 **공유, 버전 관리, 리뷰, 최적화(shared, versioned, reviewed, optimized)**해야 하며, **선명한 설계(clarity)**에 대한 초기 투자로 장기적 재작업 비용을 줄여야 한다.

## **3. 상세 요약 (Detailed Summary)**

### **도입(Introduction)**

문서는 대규모 엔터프라이즈 환경에서 SQL이 표면상 단순해 보이지만, **팀 규모와 시스템 복잡도가 증가**함에 따라 **유지보수성, 성능, 신뢰성**을 저해하는 **안티패턴(anti-patterns)**이 누적되는 현실을 전제한다. 특히 “빠른 개발”을 위한 **작은 지름길(shortcuts)**이 장기적으로 **데이터 신뢰 저하(erosion in trust in data)**와 **쿼리 개발 속도 둔화(slower query development)**를 초래할 수 있음을 경고한다. 결론적으로, SQL은 **프로덕션 코드(production code)**처럼 **공유, 버전 관리, 리뷰, 최적화(shared, versioned, reviewed, optimized)**되어야 한다.

### **대형 CASE WHEN의 오남용(Mishandling Excessive Case When Statements)**

- 문제 인식: **대형 CASE WHEN(CASE WHEN)**으로 애플리케이션 상태 코드(status codes)를 대시보드 전용 뷰에만 번역하는 방식은 **재사용 가능성(reusability)**과 **일관성(consistency)**을 저해한다. 그 결과 타 개발자는 **복붙(copy-and-paste)** 또는 **원시 상태 코드(raw status)** 사용을 강요받아 조직 전체에 **논리 중복(logic duplication)**과 **혼란**을 초래한다.
- 권고: **차원 테이블/뷰(Dimension table/view)**를 원본 로딩 테이블(landed table) 기반으로 구성하여 **공유 가능한 단일 진실 원천(single source of truth)**을 마련한다. 이를 통해 상태 코드 번역 로직이 **접근 가능하고(accessible)** **일관되게(consistently)** 재사용된다.

### **인덱스 컬럼에 대한 함수 적용(Using Functions on Indexed Columns)**

- 문제 인식: **인덱스(Index)**가 있는 컬럼에 **함수(Function)**를 적용하는 조건(예: **WHERE UPPER(name) = ‘ABC’(UPPER)**)은 RDBMS가 인덱스를 활용하지 못하게 만들어 **풀 테이블 스캔(full table scan)**을 유발할 수 있다.
- 권고: **인덱스 친화적(index-aware)** 조건식을 작성하여 데이터베이스가 인덱스를 적절히 활용하도록 한다. [불확실성: 구체적 대안 예시 생략—원문에 “Instead you should:” 이후 상세 예시가 누락됨]

### **뷰에서의 SELECT *(Using SELECT * In Views)**

- 문제 인식: 뷰 정의 말미에 **SELECT *(SELECT *)**를 사용하는 관행은 **스키마 진화(schema evolution)**에 취약하여 **뷰 깨짐(view breakage)** 및 **다운스트림 영향(downstream effects)**을 유발할 수 있다. 또한 **불필요한 컬럼 포함(unnecessary columns)**로 성능 및 가독성이 저하된다.
- 권고: **명시적 컬럼 나열(explicit column lists)**로 스키마 변경에 대한 회복탄력성을 확보하고, 뷰의 **계약(interface/contract)**을 명확히 한다.

### **DISTINCT의 과용으로 중복 “교정”(Overusing DISTINCT to “Fix” Duplicates)**

- 문제 인식: **DISTINCT(DISTINCT)**로 조인 후 발생한 중복을 “임시로 제거”하는 방식은 대개 **조인 조건의 불완전성(incomplete join conditions)** 또는 **1:1 관계의 부재(non one-to-one relationships)**라는 **근본 원인(root cause)**을 숨긴다. 이는 이후 동일 데이터셋으로 구축한 **지표(metrics)**에서 **일관성 붕괴(inconsistent counts)**를 초래한다.
- 권고: **조인 로직(join logic)**을 교정하여 **관계 정의(relationship definition)**를 명확히 하고, **집계(aggregation)** 또는 **리포팅(reporting)** 이전에 **정합성(correctness)**을 보장한다.

### **과도한 뷰 레이어 중첩(Excessive View Layer Stacking)**

- 문제 인식: 조직 확장과 함께 **뷰 위에 뷰를 중첩(view-on-view)**하는 방식은 의존성 사슬을 급격히 복잡화한다. 데이터베이스는 매 쿼리마다 **여러 계층의 로직(expand multiple layers of logic)**을 전개해야 하므로 성능이 저하되고, 디버깅은 **고고학적 발굴(archaeological dig)**에 비유될 만큼 어려워진다.
- 권고: **주기적 평탄화(periodic flattening)**와 **무거운 로직(materialize heavy logic)**의 **명확한 베이스 뷰/테이블(base views/tables)**로의 물리화(materialization)를 통해 **성능(performance)**과 **가독성(readability)**을 회복한다.

### **과도한 중첩 서브쿼리(Nested Subqueries That Do Too Much)**

- 문제 인식: **서브쿼리(Subqueries)**는 초기 필터링/논리 분리에 유용하지만, 3~4 레이어 이상의 깊은 중첩과 누적된 필터/집계는 **수천 라인(>5000 lines)**의 쿼리를 낳아 **디버깅 난이도**를 극대화한다.
- 권고: **CTE(Common Table Expressions, CTE)**를 사용하여 **가독성(readability)**과 **논리 단계화(logical staging)**를 확보하고, **중첩 수준(nesting depth)**을 억제한다.

### **결론(Conclusion)**

문서는 대부분의 안티패턴이 **나쁜 의도**가 아니라 **속도와 마감(deadlines)**에서 비롯되며, 누적될수록 **재작업(rework)**과 **혼란(confusion)**이 커진다고 요약한다. 최선의 접근은 SQL을 **프로덕션 코드(production code)**처럼 **공유(shared)**, **버전 관리(versioned)**, **리뷰(reviewed)**, **최적화(optimized)**하고, 초기 단계에서 **명료성(clarity)** 설계를 수행하여 장기 비용을 줄이는 것이다.

## **4. 결론 및 전문가 견해 (Conclusion and Professional View)**

- *공유 가능한 상태 코드 번역 로직(Dimension table/view)**으로 **일관성**과 **재사용성**을 확보하라.
- *인덱스 친화적 조건식(Index-aware predicates)**을 채택하여 **풀 스캔**을 예방하고 성능을 안정화하라.
- *SELECT *(SELECT )* 대신 **명시적 컬럼 목록**을 유지해 스키마 진화에 견고한 뷰 계약을 확보하라.
- *DISTINCT(DISTINCT)**로 중복을 숨기지 말고 **조인 관계의 정확성(join correctness)**을 근본적으로 수선하라.
- *뷰 중첩(View stacking)**을 정기적으로 **평탄화(materialization/flattening)**하여 의존성 복잡도를 관리하라.
- *CTE(CTE)**로 쿼리 로직을 단계화하여 **가독성**과 **디버그 가능성**을 극대화하라.
- 팀의 SQL 산출물을 **공유, 버전 관리, 리뷰, 최적화(shared, versioned, reviewed, optimized)**의 관행으로 운영하라.
- 초기 **명료성(clarity)** 설계에 투자하여 **데이터 신뢰(Trust in data)**와 **개발 속도**를 장기적으로 개선하라.

## **5. 불확실성 지도 (Uncertainty Map)**

- **원본 모호성 (Source Ambiguity)**: 인덱스 컬럼 함수 적용 섹션에서 “Instead you should:” 뒤의 **구체적 대안 예시**가 누락되어 있다. [불확실성: 대체 조건 예시 부재]
- **판단 근거 (Judgment Calls)**: 각 안티패턴의 문제와 권고를 **원문 표현의 범위 내**에서 구조화했으며, “프로덕션 코드처럼 다루라”는 결론을 **조직적 관행**으로 요약했다.
- **해석 불확실성 (Interpretive Uncertainty)**: “뷰 레이어 중첩”의 성능 저하 원인은 DB 엔진별로 상이할 수 있으나, 본문은 **일반적 성능 악화**로 기술한다. [불확실성: 엔진 특성 미제시]
- **단순화/추상화 영역 (Simplification/Abstraction Areas)**: 상태 코드 번역의 **소스 테이블(landed table)** 구체 정의, **조인 정합성**을 위한 실무적 규칙 등은 본문 예시 수준으로만 요약되었다.
- **의견 전환 질문 (Opinion-Changing Questions)**:
    1. 인덱스 친화적 조건식의 **구체적 대안 패턴**(예: 대소문자 불변 비교 전략)은 무엇인가?
    2. **뷰 평탄화/물리화**의 주기와 기준은 어떤 **운영 메트릭**(예: 쿼리 계획 복잡도, 실행 시간, 의존성 깊이)에 의해 결정되어야 하는가?
