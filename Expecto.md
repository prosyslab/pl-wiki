# Expecto: 자연어 요구사항을 엄밀한 명세로 바꾸는 시스템

## 1. 배경

본 문서를 이해하기 위해 다음 문서들을 먼저 읽어보는 것을 권장한다.

- [코딩 에이전트](<https://github.com/prosyslab/pl-wiki/wiki/코딩-에이전트-(Coding-Agent)>)

## 2. 개요

Expecto는 자연어로 작성된 프로그램 요구사항을 컴퓨터가 검사할 수 있는 엄밀한 명세로 바꾸는 도구다.

최근 명세 주도 개발(spec-driven development) 방식으로 자연어 명세로부터 구현을 생성하는 도구들(Spec Kit<sup>[1](#spec_kit)</sup>, Kiro<sup>[2](#kiro_specs)</sup>)이 주목받고 있다.
그러나 자연어 명세와 구현의 일관성을 검사하고 보장하는 것은 여전히 사람의 몫이며, 이는 매우 어려운 작업이다.
이때 사용자의 요구사항을 엄밀한 명세(formal specification)로 표현할 수 있다면 엄밀한 검증(formal verification)을 사용하여 코드가 올바름을 보장한다는 것을 자동으로 증명할 수 있다.
Expecto는 자연어로 작성된 사용자의 요구사항을 엄밀한 명세로 변환하여 엄밀한 검증을 가능하게 해주는 핵심 기술이다.

## 3. 예제

다음 자연어 지시사항을 생각해보자.

```text
창고는 n × m 크기의 격자이다. 각 칸은 빈 칸 "."이거나 벽 "*"이다.
폭탄을 (x, y)에 놓으면, 그 행 x와 열 y에 있는 모든 벽이 사라진다.
폭탄 하나로 모든 벽을 없앨 수 있는지 판단해야 한다. 폭탄은 빈 칸에도 놓을 수 있고, 벽이 있는 칸에도 놓을 수 있다.
폭탄 하나로 모든 벽을 없앨 수 있는지와 어느 좌표에 두어야 모든 벽을 제거할 수 있는지 출력하라.
```

입출력 예시는 다음과 같다.

```
Input:
n = 3, m = 5, depot = [
    [".", ".", ".", ".", "*"],
    ["*", "*", ".", ".", "*"],
    [".", ".", ".", ".", "*"],
]
Output:
possible=True x = 2 y = 5
```

위 자연어 지시사항을 한 번에 엄밀한 명세로 바꾸게 되면 언어 모델에게 큰 인지적 부하가 발생하게 된다.
Expecto는 자연어 지시사항을 바꾸는 과정을 여러 단계로 쪼개 언어 모델이 손쉽게 바꿀 수 있는 단위의 문제를 여러 번 풀어 합치는 방식을 사용한다.

Expecto는 명세의 가장 핵심이 되는 논리식을 가장 먼저 생성한다.

```
Spec(n: int, m: int, depot: list[list[str]], possible: bool, x: int, y: int)=
  (possible →
    ValidPos(x, y, n, m) ∧ WipesAll(x, y, n, m, depot))
  ∧
  (¬possible →
    ∀r, c. ValidPos(r, c, n, m) → ¬WipesAll(r, c, n, m, depot))
```

이 과정에서 `ValidPos`, `WipesAll`이라는 새로운 함수가 사용되는데, 이 두 가지는 논리식으로 즉시 번역하지 않고, 해당 함수의 동작을 자연어로 설명하는 간단한 정의(informal definition) 방식으로 정의한다.

```
ValidPos(x: int, y: int, n: int, m: int): n x m 격자에서 (x, y)는 유효한 좌표인가?
WipesAll(x: int, y: int, n: int, m: int, depot: list[list[str]]): n x m 격자 모양의 창고 depot에서 x, y에 폭탄을 두었을 때 모든 벽이 제거되는가?
```

간단한 정의로 정의된 함수는 작업 목록(worklist)에 등록되고, 목록에서 하나씩 꺼내 엄밀한 정의로 변환하는 과정을 반복한다.
이 과정에서 새로운 간단한 정의가 추가되는 경우 작업 목록에 새로 추가한다.
위 과정을 작업 목록이 빌 때까지 반복하며, 이를 [**하향식 명세 합성(Top-down specification synthesis)**](#4-하향식-명세-합성)이라고 한다.

## 4. 하향식 명세 합성

명세 번역의 핵심은 자연어 요구사항을 한 번에 완성된 엄밀한 명세로 바꾸는 것이 아니라, 점진적으로 바꾸어 가는 것이다.
Expecto는 전체 명세를 여러 개의 함수 정의로 구성된 집합으로 본다.
이 정의들은 두 종류로 나뉜다.

1. 이미 논리식으로 작성된 엄밀한 정의(formal definition)
2. 아직 논리식으로 작성되지는 않았지만, 타입과 자연어 설명을 가진 간단한 정의(informal definition)

Expecto는 이 두 종류의 정의를 함께 사용하여 간단한 정의를 하나씩 엄밀한 정의로 바꾸어 간다.

간단한 정의는 단순한 메모가 아니라, 앞으로 번역해야 할 하위 명세를 명시적으로 나타내는 중간 표현이다.
각 간단한 정의는 함수 이름, 입력과 출력 타입, 그리고 그 함수가 만족해야 할 자연어 설명으로 구성된다.
따라서 Expecto는 아직 모든 명세를 완성하지 않았더라도, 현재까지 어떤 부분이 엄밀하게 정의되었고 어떤 부분이 아직 남아 있는지 명확하게 추적할 수 있다.

Expecto는 매 단계에서 작업 목록에 남아 있는 간단한 정의 하나를 선택하고, 이를 엄밀한 논리식으로 번역한다.
이때 번역된 정의가 다시 다른 보조 개념을 필요로 하면, 그 보조 개념을 새로운 간단한 정의로 작업 목록에 추가한다.
이 과정을 반복하면 전체 요구사항은 상위 수준의 명세에서 시작해 더 작은 하위 명세로 점차 분해된다.

하향식 명세 합성은 전체 명세를 한 번에 생성하는 방식보다 안정적이다.
언어 모델이 한 번에 처리해야 하는 문제가 작아지기 때문이다.
각 단계에서 언어 모델은 전체 프로그램 요구사항을 한꺼번에 번역하는 대신 특정 함수 하나의 의미만 엄밀하게 표현하면 된다.
또한 생성된 명세가 여러 작은 정의로 나뉘기 때문에 사람이 읽고 검토하기도 쉽다.

## 5. 명세 검사하기(Specification Validation)

Expecto는 하향식 명세 합성의 매 단계마다 현재 명세가 올바른 명세로 완성될 가능성이 있는지 검사한다.
새로운 엄밀한 정의가 하나 만들어질 때마다 형식이 올바른지, 논리적으로 말이 되는지, 그리고 주어진 입출력 예시와 맞는지를 검사한다.
이를 통해 잘못된 명세를 조기에 걸러내고, 오류 정보를 다시 언어 모델에게 전달하여 명세를 수정하게 한다.

문법 및 타입 검사는 생성된 명세가 Expecto의 명세 언어의 문법을 만족하는지, 타입이 올바른지 검사한다.
두 검사는 명세의 깊은 의미를 따지기 전에, 생성된 명세가 기본적으로 분석 가능하고 실행 가능한 형태인지 확인할 수 있는 가벼운 검사다.

논리적 일관성 검사는 명세가 항상 거짓이거나 항상 참이 되는지를 확인한다.
항상 거짓인 명세는 어떤 올바른 출력도 받아들이지 못하므로 너무 강한 명세이다.
반대로 항상 참인 명세는 어떤 잘못된 출력도 거부하지 못하므로 너무 약한 명세이다.
Expecto는 SMT solver를 사용하여 명세가 모순되지 않는지, 그리고 아무 조건도 검사하지 않는 빈 명세처럼 동작하지 않는지 확인한다.
즉, 논리적 일관성 검사는 명세가 최소한의 의미 있는 제약을 가지고 있는지 확인하는 단계이다.

테스트 케이스 일관성 검사는 생성된 명세가 주어진 입출력 예시와 일관되는지 확인한다.
올바른 입출력 예시가 주어진 경우 명세는 주어진 입출력 예시를 올바른 것으로 판정해야 한다.
반대로 잘못된 입출력 예시가 주어진 경우, 명세는 주어진 입출력 예시를 거부해야 한다.

Expecto의 중요한 특징은 아직 완성되지 않은 명세도 검사할 수 있다는 점이다.
일부 하위 정의가 아직 자연어 설명으로만 남아 있더라도, Expecto는 이를 의미가 정해지지 않은 함수(uninterpreted function)로 두고 SMT solver에 전달한다.
이를 통해 간단한 정의가 남아 있는 명세도 논리적 일관성 검사와 테스트 케이스 일관성 검사를 수행할 수 있다.

## 6. 안전성과 완전성

생성된 명세는 안전성과 완전성을 만족하는지 확인하는 방식으로 평가된다.

안전한 명세는 잘못된 입출력 쌍을 거부해야 한다.
예를 들어,

```
[
    ["*", "*", "*"],
    ["*", "*", "*"],
    ["*", "*", "*"],
]
```

이렇게 생긴 격자에서 폭탄 하나로 모든 벽을 제거하는 것은 불가능하다.
만약 `possible=True`가 출력에 있다면 명세가 이것이 잘못된 입출력 쌍임을 알아차릴 수 있어야 한다.

완전한 명세는 올바른 입출력 쌍을 통과시켜야 한다.
폭탄 예제에서 `(True, 1, 1)`이 실제로 올바른 출력이라면, 완전한 명세는 이 출력을 받아들여야 한다.

예를 들어, 다음 명세는 올바른 입출력 쌍을 모두 통과시키므로 완전하지만, 잘못된 입출력 쌍을 걸러낼 수 없어 안전하지는 않다.

```
(¬ possible) ∨ (1 <= y <= n ∧ 1 <= x <= m)
```

논문에서는 결과를 다음 네 가지로 분류한다.

| 분류 | 의미                                 |
| ---- | ------------------------------------ |
| S&C  | 안전하고 완전한 명세이다.            |
| S    | 안전하지만 완전하지 않은 명세이다.   |
| C    | 완전하지만 안전하지 않은 명세이다.   |
| W    | 안전하지도 완전하지도 않은 명세이다. |

좋은 명세는 S&C에 속해야 한다. 즉, 틀린 출력은 막고, 맞는 출력은 받아들여야 한다.

## 7. 평가

Expecto는 언어 모델로 gpt-4.1-mini를 사용하였고 SMT Solver로 Z3<sup>[6](#z3)</sup>를 사용하였다.
벤치마크는 HumanEval+<sup>[3](#humanevalplus)</sup>, APPS<sup>[4](#apps)</sup>, Defects4J<sup>[5](#defects4j)</sup>를 사용하였다.

| 벤치마크   | 설명                                                                                                                                                  |
| ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| HumanEval+ | Python 함수 구현 문제로 구성된 코드 생성 벤치마크이다. 논문에서는 164개 문제를 사용했다.                              |
| APPS       | 알고리즘, 자료구조, 수학적 추론이 필요한 경쟁 프로그래밍 문제 모음이다. 논문에서는 테스트 케이스가 충분한 127개 문제를 사용했다. |
| Defects4J  | 실제 Java 오픈소스 프로젝트의 버그를 모은 벤치마크이다. 논문에서는 336개 버그와 관련된 501개 메서드를 사용했다.             |

HumanEval+와 APPS에서는 문제 설명을 자연어 명세로 사용했다.
Defects4J에서는 메서드의 Javadoc 주석을 자연어 명세로 사용했다.

Expecto는 프로그램의 명세를 한 번에 사후조건으로 변환하는 NL2Postcond<sup>[7](#nl2postcond)</sup>의 두 가지 변형을 비교 대상으로 사용했다.

| 비교 대상          | 설명                                                                                                 |
| ------------------ | ---------------------------------------------------------------------------------------------------- |
| NL2Postcond Base   | 요구사항 전체를 한 번에 포괄하는 사후조건을 만들려고 한다.                                           |
| NL2Postcond Simple | 프로그램 동작 전체를 모두 표현하기보다, 중요한 일부 조건을 더 간단한 사후조건으로 표현하도록 요청한다. |

Base는 더 강한 명세를 만들 수 있지만, 복잡한 문제에서는 사후조건이 지나치게 복잡해져 오류가 생기기 쉽다.
Simple은 더 안정적인 사후조건을 만들 수 있지만, 요구사항 전체를 충분히 표현하지 못할 수 있다.

주요 결과는 다음과 같다.

| 벤치마크   | Expecto S&C | NL2Postcond Base S&C | NL2Postcond Simple S&C |
| ---------- | ----------: | -------------------: | ---------------------: |
| HumanEval+ |         103 |                   86 |                     83 |
| APPS       |          59 |                   24 |                     21 |
| Defects4J  |          42 |                    6 |                     14 |

Expecto는 NL2Postcond보다 더 많은 S&C 명세를 생성하며, 특히 APPS처럼 복잡한 문제에서 차이가 크다.
이는 Expecto가 복잡한 명세를 작은 하위 명세로 나누고, 중간 결과를 계속 검증하기 때문이다.

## 8. 결론

Expecto는 자연어 요구사항을 한 번에 완성된 명세로 바꾸는 대신, 여러 하위 정의로 나누어 점진적으로 엄밀한 명세를 합성하는 하향식 명세 합성 기법을 도입했다.
이 과정에서 문법, 타입, 논리적 일관성, 테스트 케이스 일관성을 매 단계마다 검사한다.
그 결과 Expecto는 NL2Postcond보다 더 많은 안전하고 완전한 명세를 생성했으며, 특히 APPS처럼 복잡한 문제에서 큰 차이를 보였다.

## 9. 관련 자료

- 논문: [Expecto: Extracting Formal Specifications from Natural Language Description for Trustworthy Oracles](https://prosys.kaist.ac.kr/publications/pldi26.pdf)
- 프로젝트 웹사이트: https://prosys.kaist.ac.kr/expecto

## 참고 문헌

- [<a id="spec_kit">1</a>] GitHub. (2026). _GitHub Spec Kit documentation_. GitHub. from https://github.github.com/spec-kit/
- [<a id="kiro_specs">2</a>] Kiro. (2026). _Specs_. Kiro Docs. from https://kiro.dev/docs/specs/
- [<a id="humanevalplus">3</a>] Liu, J., Xia, C. S., Wang, Y., & Zhang, L. (2023). _Is your code generated by ChatGPT really correct? Rigorous evaluation of large language models for code generation_. Advances in Neural Information Processing Systems.
- [<a id="apps">4</a>] Hendrycks, D., et al. (2021). _Measuring coding challenge competence with APPS_. Neural Information Processing Systems Track on Datasets and Benchmarks.
- [<a id="defects4j">5</a>] Just, R., Jalali, D., & Ernst, M. D. (2014). _Defects4J: A database of existing faults to enable controlled testing studies for Java programs_. International Symposium on Software Testing and Analysis.
- [<a id="z3">6</a>] de Moura, L., & Bjørner, N. (2008). _Z3: An efficient SMT solver_. International Conference on Tools and Algorithms for the Construction and Analysis of Systems.
- [<a id="nl2postcond">7</a>] Endres, M., Fakhoury, S., Chakraborty, S., & Lahiri, S. K. (2024). _Can large language models transform natural language intent into formal method postconditions?_ ACM International Conference on the Foundations of Software Engineering.
