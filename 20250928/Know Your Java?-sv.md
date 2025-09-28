---
id: Know Your Java?
aliases: 자바를 제대로 알고 있나요?
tags:
- java/language/nuances/autoboxing-overloading
- java/collections/arrays-as-list-vs-list-of
- java/streams/purity-lazy-evaluation-parallel
- development/practices/immutable-collections
- architecture/programming-paradigms/functional-programming
author: java
created_at: 2025-09-28 23:06
related: []
source: '[불확실: 원본 YouTube URL을 페이지에서 확인할 수 없습니다]'
---

# 번역 및 요약

## 하이라이트

발표자는 자바(Java) 사용 중 경험 많은 개발자도 종종 발을 걸게 되는 언어·라이브러리의 미묘한 함정과 API 설계상의 모호성(ambiguity)을 실전 코드로 보여주며, 타입 추론(type inference), 오토박싱(autoboxing), 오버로딩(overloading)과 오버라이딩(overriding), 그리고 컬렉션 및 스트림(Streams)에서의 순수성(purity)·불변(immutability) 원칙을 강조했습니다. 특히 List의 remove(인덱스 vs 객체) 혼동, Arrays.asList의 “확장 불가지만 변경 가능”이라는 함정, var로 인해 구체 타입(ArrayList)로 고정되는 문제, Collectors.toList와 Stream.toList의 가변/불변 차이, forEach 내 측효과(side effect)와 parallelStream에서의 비결정성 등 실무에서 바로 사고로 이어질 수 있는 포인트를 명확히 짚습니다.

핵심 메시지는 “코드는 의도를 드러내야 한다”와 “함수의 순수성은 게으른 평가(lazy evaluation)와 병렬 실행의 정확성에 필수”입니다. 리스트 초기화는 List.of(Java 9+) 같은 진정한 불변 컬렉션을 기본값으로 삼고, 스트림 연산의 람다는 순수(pure)해야 하며, 외부 변경 가능한 상태에 의존하거나 외부를 변경하지 말아야 한다는 두 가지 조건을 반드시 만족하라고 강조합니다. 마지막 예에서 외부 factor 값 변경으로 인해 lazy evaluation 시점에 0으로 평가되어 결과가 000이 되는 사례를 통해, 규칙 2(외부 변경 가능 상태에 의존 금지)의 중요성을 재확인합니다.

## 상세 요약

### 0:00–5:00

발표 서두에서 다수의 개발자 경험에도 불구하고 자바의 미묘한 동작 차이로 발을 “찌르는” 순간이 있음을 공유합니다. 형식적으로는 상호작용형 퀴즈를 통해 코드 동작을 예측하게 만들며, 정적 타입 언어와 자동화 테스트의 장점을 인정하면서도, 컴파일러·테스트가 놓치는 “의미론(semantics)” 이슈가 있음을 밝힙니다.

첫 기술 예고: API 설계의 의도 표현이 모호하면 개발자가 실수하기 쉬워지고, 상속 계층에서 오버로딩/오버라이딩이 섞일 때 타입 차이로 혼란이 커진다는 문제의식이 제시됩니다.

### 5:00–10:00

첫 코드 예: `List<Integer>`에서 `numbers.remove(1)`는 인덱스 1의 값(2)을 제거하여 `[1, 3]`이 되지만, 참조를 `Collection<Integer>`로 바꾸면 `remove(1)`가 오토박싱되어 `Integer(1)` 객체 제거로 해석되어 `[2, 3]`이 됩니다. 이 모호성은 API가 의도를 드러내지 못한 결과이며, 메서드명을 `removeAtIndex` 같은 명시적 의도로 설계했다면 혼동을 줄였을 것이라고 지적합니다.

교훈: “코드는 당신이 ‘의미한 바’가 아니라 ‘타이핑한 것’대로 수행한다.” Integer는 컬렉션 요소 타입과 인덱스로 모두 쓰일 수 있어 edge case가 발생했음을 인정하면서도, 테스트와 의도 명시가 중요함을 강조합니다.

```java
List<Integer> list = new ArrayList<>(List.of(1, 2, 3));
list.remove(1);           // 인덱스 제거 → [1, 3]
Collection<Integer> c = list;
c.remove(1);              // 객체 제거(오토박싱) → [2, 3]
```

### 10:00–15:00

타입 추론(`var`) 사례: `var numbers = new ArrayList<>(…);`를 쓰면 `numbers`의 정체는 `List`나 `Collection`이 아니라 구체 타입 `ArrayList`로 확정됩니다. 이후 API 사용에서 “상위 인터페이스로 느슨한 결합”을 의도했다면 실패이며, `var`는 문맥상 “아는 범위” 내에서만 타입을 추론하기 때문에 구체 타입에 고정되는 점을 경계해야 합니다.

권장 실무 습관: IDE에서 변수에 마우스를 올려 실제 추론 타입을 확인하고, 의도(예: `Collection<Integer>`)와 다르면 `var`를 사용하지 말 것. 타입 추론의 장단을 인정하되 “항상” 쓰지 말고 맥락에 맞춰 판단하라.

### 15:00–20:00

`Arrays.asList` 함정: `Arrays.asList(1,2,3)`은 “크기 확장 불가(Unsupported add)”지만 “요소 교체(set)는 가능”한 고정 크기 리스트입니다. 발표자는 초판 서적에서 이를 불변(immutable)로 오해하여 수정 메일을 받은 일화를 공유합니다. 정정: `add`는 실패하지만 `set`은 성공합니다.

권장 대안: Java 9의 `List.of`는 진정한 불변 컬렉션(요소 변경·확장 모두 불가)을 반환하며, `Arrays$ArrayList`처럼 특수 타입이 아니라 “진짜 immutable collections”로 표기되는 구현을 제공합니다. 테스트 데이터나 상수 데이터 초기화에는 `List.of`/`Set.of`/`Map.of`를 사용하라고 권합니다.

```java
var a = Arrays.asList(1, 2, 3);
a.add(4);   // UnsupportedOperationException
a.set(0, 9);// OK

var b = List.of(1, 2, 3);
b.add(4);   // UnsupportedOperationException
b.set(0, 9);// UnsupportedOperationException (진정한 불변)
```

### 20:00–25:00

Stream 수집의 변화: 예전에는 `Collectors.toList()`가 가변 리스트를 반환했고(Java 12에는 `Collections.toUnmodifiableList`도 있었음), 최근 `Stream.toList()`는 불변 리스트를 반환합니다. 이름은 유사하지만 반환 리스트의 변경 가능성(mutable vs immutable)이 다르므로 API 반환 성질을 반드시 확인해야 합니다.

실무 권장: 기본으로 불변을 선택하고, 변이가 필요할 때만 명시적으로 가변 구조로 변환. “의도적으로 변경 가능한 컨테이너”라는 신호를 코드에 남겨 리뷰·유지보수 시 혼동을 줄입니다.

### 25:00–30:00

`forEach` 내 측효과(side effect) 사례: `names.stream().map(String::toUpperCase).forEach(uppercase::add)`처럼 외부 컬렉션에 추가하는 패턴은 “티키타임봄(ticking time bomb)”입니다. 평소엔 잘 동작하는 듯 보이다가 `parallelStream`으로 바꾸거나 실행 환경이 달라지면 비결정적 결과(누락, 중복)가 발생할 수 있습니다.

같은 코드라도 여러 번 돌리면 결과가 7, 6, 5 등 달라질 수 있음을 시연합니다. 병렬에서 외부 상태 변경은 경합 조건(race condition)과 가시성 문제를 일으켜 테스트로도 포착하기 어렵습니다.

안전한 대안:
```java
List<String> uppercase = 
names.stream()
.map(String::toUpperCase)
.toList();     // 불변 리스트 반환 (Java 16+)
```

혹은 병렬에 안전한 수집기 사용:
```java
List<String> uppercase = 
names.parallelStream()
.map(String::toUpperCase)
.collect(Collectors.toList()); // 가변 리스트지만 수집기 내부에서 안전하게 모음
```

외부 공유 상태를 `forEach`에서 변경하지 말고, 수집으로 결과를 반환하는 “순수” 파이프라인을 구성합니다.

### 30:00–35:00

람다의 순수성(pure function): 두 가지 규칙을 모두 충족해야 합니다. (1) 외부를 변경하지 않는다(no side effects). (2) 변경 가능한 외부 상태에 의존하지 않는다(no dependency on mutable external state). 많은 개발자가 (1)은 떠올리지만 (2)를 간과합니다. 그러나 병렬/게으른 평가에서는 (2)가 특히 중요합니다.

비유를 통해 “변이(mutation)는 옷 갈아입기와 같아, 필요하지만 ‘남이 보는 곳(외부 가시성)’에서 하면 곤란하다”라고 설명합니다. 함수 내부 비가시적 변이는 허용되지만, 외부에 보이는 변이는 금지입니다.

### 35:00–40:29

게으른 평가(lazy evaluation) 함정: 스트림 파이프라인이 구성만 되고, 최종 연산(terminal operation) 시점에 외부 값이 바뀌면 그 변경이 반영되어 예상과 다른 결과가 나옵니다. 마지막 예에서 외부 `factor[0]`을 2에서 0으로 바꾼 뒤 `map`이 실행되어 모든 결과가 0이 됩니다(000). 이는 규칙 (2) 위반의 전형입니다.

```java
// 코드 예(개념적)
List<Integer> src = List.of(1, 2, 3);
int[] factor = {2};             // 외부 변경 가능 상태
var result = src.stream()
.map(n -> n * factor[0]) // 외부 상태 의존
.toList();
factor[0] = 0;                   // terminal 이전에 변경되면
System.out.println(result);      // lazy 평가 시 0,0,0
```

결론: 함수형 프로그래밍은 효율을 위해 게으른 평가를 활용하므로, 정확성을 위해서는 람다의 순수성과 컬렉션 불변성이 필수입니다.

## 결론 및 의견

* 코드의 의도를 드러내는 API 설계가 중요하다. `remove(인덱스)` vs `remove(객체)` 같은 모호성은 실수를 유발한다.
* `Arrays.asList`는 “확장 불가지만 변경 가능”하다. 진정한 불변 초기화에는 `List.of`/`Set.of`/`Map.of`를 기본으로 사용하라.
* `var`(타입 추론)는 구체 타입으로 고정될 수 있다. 상위 인터페이스로 의존하려면 명시적 선언을 유지하라.
* `Stream.toList`(불변)와 `Collectors.toList`(가변)를 구분하라. 반환 컬렉션의 변경 가능성은 유지보수성·안전성에 직결된다.
* `forEach`에서 외부 컬렉션을 수정하는 패턴은 병렬에서 비결정성·데이터 손실을 초래한다. 수집으로 결과를 반환하는 순수 파이프라인을 구성하라.
* 람다의 순수성은 두 가지: 외부를 변경하지 않고, 변경 가능한 외부 상태에 의존하지 않는다. 둘 다 지켜야 병렬/게으른 평가가 안전하다.
* 게으른 평가로 인해 파이프라인 구성 이후의 외부 상태 변경이 최종 연산 시 반영된다. 외부 상태 의존을 제거하라.
* 테스트가 모든 의미론적 함정을 잡아주지 않는다. API 동작·반환 특성(가변/불변, 구체/추상 타입)을 스스로 확인·명시하라.
* 기본 전략을 “불변 우선”으로 잡고, 변이가 필요할 때만 국소적으로 허용하라.
* 실무에서는 작은 차이가 대형 사고로 이어진다. 위 사례들을 팀 코딩 규약에 반영하여 예방하라.
