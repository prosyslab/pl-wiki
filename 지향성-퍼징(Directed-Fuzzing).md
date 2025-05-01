## 1. 개요
지향성 퍼징은 퍼징 기법의 한 종류로서 프로그램의 특정 지점에 도달하는 입력을 생성하는 것을 목표로 한다.
예를 들어 패치나 정적분석 결과를 검사해야 할 때, 우리는 프로그램 전체보다는 주어진 패치나 정적분석 알람과 연관된 프로그램 지점에 관심을 가지게 된다.
이 때, 일반적인 퍼징 기법을 사용한다면 전체 프로그램을 살펴보느라 정작 우리가 관심있는 프로그램 지점은 제대로 검사하지 못할 수 있다.
따라서 이런 경우에는 주어진 프로그램 지점에 집중할 수 있는 지향성 퍼징이 더 효과적일 것이다.

## 2. 퍼징
지향성 퍼징에 대해 더 자세히 설명하기 위해서는 그 상위 개념인 퍼징에 대해서 간략히 알고 갈 필요가 있다.
퍼징은 자동으로 무수한 입력을 생성하여 프로그램을 테스트하는 기술이다.
특히 무수한 입력을 무작위성의 힘을 빌려 생성하기 때문에 그 중에는 개발자가 미처 예상하지 못한 입력도 있기 마련이고, 이를 통해 미처 몰랐던 결함을 발견할 수도 있다.
퍼징은 대상 프로그램에 대한 정보가 거의 없는 상황에서도 프로그램을 효과적으로 관통하는 입력을 만들어내는 등, 그 강력한 성능으로 인해 많은 사랑을 받아왔다.

### 2.1. 퍼징의 탄생<sup>[1](#fuzzing_begin)</sup>
비바람이 몰아치던 어느날 밤, Wisconsin-Madison 대학의 Miller 교수는 집에서 야근을 하고 있었다.
야근하는 것도 서러운데 그날따라 학교 서버에서 원격으로 돌아가는 유닉스 유틸리티 프로그램들에 자꾸만 알 수 없는 오류가 발생했다고 한다.
하지만 뼛속까지 학자였던 Miller 교수는 그 원인을 추측하기에 나섰는데, 그것은 바로 번개로 인해 발생한 네트워크상의 노이즈가 자신의 정상적인 입력을 변형시켰고,
그것이 모종의 이유로 프로그램들의 오류를 유발했다는 것이었다.
그 해 Miller 교수는 자신의 운영체제 수업에서 이 가설을 실제로 검증하기 위한 과제를 내었고, 이것이 바로 퍼징의 시초이다.

### 2.2. 대표적인 퍼징 도구
가장 많이 알려진 대표적인 퍼징 도구는 단언컨대 AFL<sup>[2](#AFL)</sup> 일 것이다. 
AFL은 변형 기반 반투명(Mutation-Based Greybox) 퍼징 도구이다.
변형 기반 반투명 퍼징은 주어진 입력을 변형하여 새로운 입력을 만들고, 그 새로운 입력의 프로그램의 실행 결과를 엿보아 그것이 좋은 입력인지 아닌지 판단하는 것이다.
이때, 반투명(Greybox)이라 함은 프로그램 실행을 자세히 지켜보지는 않되 그 결과로 나오는 실행 커버리지 정도의 정보는 확인한다는 것을 뜻한다.
참고로 프로그램 내부를 샅샅히 들여다보면 투명(Whitebox), 아예 들여다보지 않고 프로그램의 입력과 출력만 본다면 불투명(Blackbox) 퍼징이라 한다.
또한 좋은 입력인지 아닌지는 보통 새로운 커버리지를 달성하였는지에 따라 결정을 하며, 좋은 입력의 경우 새로운 입력을 만들 재료로서 선택된다.

즉, 다음과 같은 순서로 퍼징이 진행된다.

0. 사용자가 주는 입력으로 초기 입력 집합 구성 (없다면 빈 입력으로 시작)
1. 입력을 하나 선택
2. 해당 입력을 변형하여 새로운 입력들 생성
3. 새로운 입력들로 프로그램 실행
4. 각각의 커버리지 결과를 바탕으로 입력 집합에 넣을 것인지, 아니면 폐기할 것이지 결정
5. 1번부터 다시 시작.

AFL의 이름의 유래는 American Fuzzy Lop이라 불리는 털이 복실복실한 토끼 품종이다.
이런 이름은 AFL이 가지는 퍼징의 특성을 잘 드러낸다.
왜냐하면 AFL은 토끼의 솜털같이 삐죽빼죽 튀어나오는 다양한 입력들을 통해 삐죽빼죽하게 프로그램의 구조를 탐색해 나가기 때문이다.
재미있는 점은 또 다른 퍼징 도구인 Angora<sup>[3](#Angora)</sup>  역시 앙고라 토끼의 이름을 따왔다는 것인데,
그 이유는 앙고라 토끼의 털이 AFL 토끼보다 더 빽빽하고 긴 것처럼 Angora가 프로그램을 더 깊고 넓게 탐색하기 때문이라고 한다.

## 3. 지향성 퍼징의 역사
초기의 지향성 퍼징은 KATCH<sup>[4](#KATCH)</sup> 등의 도구처럼 기호실행(Symbolic Execution)을 활용한 방법들로 제안되었다.
그러나 기호실행은 프로그램 크기가 커질수록 속도가 많이 느려지기에 그다지 빛을 발하지 못했다.
하지만 2017년에 AFLGo라는 논문이 등장하며 본격적으로 지향성 퍼징의 역사가 시작되었다.
전체 프로그램이 아닌 특정 지점에 집중하는 퍼징은 정적 분석등의 결과를 검사하거나 패치를 검사할 때 유용하다는 점에서 많은 연구자들의 관심을 끌었다.
특히 연속개발(Continuous integration)과 같은 환경에서는 프로그램 수정이 빈번하게 일어나기에 수정된 부분에만 집중하는 지향성 퍼징은 희소식이 되었다.
따라서 활발한 연구가 진행되었고 2022년에 이르러서는 보안 분야의 최고 수준 학회(Security and Privacy)와 소프트웨어공학 분야의 최고 수준 학회(ICSE)에
지향성 퍼징 논문이 각각 한편씩 나오게 되었다.

## 4. 주목할 만한 지향성 퍼징 도구들
- AFLGo<sup>[5](#AFLGo)</sup>:
최초의 AFL기반, 즉 변형기반 엿보기 퍼징에 지향성을 추가한 퍼징 도구이다.
AFLGo는 각 입력이 목표지점과 얼마나 가까운 프로그램 지점들을 지났는지를 고려한다.
이 때 프로그램 지점들 사이의 거리를 계산하기 위해 실행흐름그래프(Control Flow Graph)를 사용하게 된다.
실행흐름그래프 상에서 목표지점과 더 가까운 프로그램 지점을 실행한 입력이 있다면, 해당 입력으로부터 더 많은 새로운 입력을 생산하는 것이다.
즉, 위의 변형기반 반투명 퍼징의 순서에서 2번 단계를 건드리는 것이다.

- Beacon<sup>[6](#Beacon)</sup>:
핵심 아이디어는 가망이 없는 입력의 실행을 조기에 종료하는 것이다.
Beacon은 우선 프로그램 분석을 통해 각 프로그램 지점에서 목표 지점에 도달하기 위해 미리 만족해야 하는 조건을 찾아낸다.
그리고 입력의 실행 시 그 조건을 만족하지 못한다면 굳이 더 볼 필요가 없으니 미리 프로그램을 종료하는 것이다.
이것은 위의 변형기반 반투명 퍼징의 순서에서 3번 단계를 건드리는 것이다.

- DAFL<sup>[7](#DAFL)</sup>:
[관련 문서](https://github.com/prosyslab/pl-wiki/wiki/DAFL) 참고.

- Optimuzz<sup>[8](#Optimuzz)</sup>:
[관련 문서](https://github.com/prosyslab/pl-wiki/wiki/Optimuzz) 참고.

이 도구들의 공통점은 동적으로 프로그램을 검사하는 대표적인 기법인 퍼징에 프로그램 분석, 즉 정적으로 프로그램을 검사하는 기법을 적용했다는 점이다.


## 5. 지향성 퍼징 도구의 평가 시 주의점
자세한 내용은 [지향성 퍼징 도구 평가](https://github.com/prosyslab/pl-wiki/wiki/지향성-퍼징-도구-평가) 문서를 참고.

## 6. 참고

[<a name="fuzzing_begin">1</a>] https://www.cs.wisc.edu/2021/01/14/the-trials-and-tribulations-of-academic-publishing-and-fuzz-testing/

[<a name="AFL">2</a>] https://afl-1.readthedocs.io/en/latest/index.html

[<a name="Angora">3</a>] "Angora: Efficient Fuzzing by Principled Search", Chen and Chen, S&P 2018

[<a name="KATCH">4</a>] "KATCH: High-Coverage Testing of Software Patches", Marinescu and Cadar, FSE 2013

[<a name="AFLGo">5</a>] "Directed Greybox Fuzzing", Bohme et al., CCS 2017

[<a name="Beacon">6</a>] "BEACON : Directed Grey-Box Fuzzing with Provable Path Pruning", Huang et al., S&P 2022

[<a name="DAFL">7</a>] "[DAFL : Directed Grey-Box Fuzzing Guided by Data Dependency](https://prosys.kaist.ac.kr/publications/sec23.pdf)", Kim et al., USENIX Security 2023

[<a name="Optimuzz">8</a>] "[Optimuzz: Optimization-Directed Compiler Fuzzing for Continuous Translation Validation](https://prosys.kaist.ac.kr/publications/pldi25.pdf)", Kwon et al., PLDI 2025
