# DAFL: 데이터 의존성 정보를 활용한 지향성 반투명 퍼징


## 1. 배경
[지향성 퍼징](https://github.com/prosyslab/pl-wiki/wiki/%EC%A7%80%ED%96%A5%EC%84%B1-%ED%8D%BC%EC%A7%95(Directed-Fuzzing)) 문서를 먼저 참고하면 본 문서의 내용을 이해하는 데 많은 도움이 될 것이다.

## 2. 개요
2017년, AFLGo를 시작으로 지금까지 다양한 지향성 반투명 퍼징 기법이 제안되어 왔다. AFLGo는 AFL을 기반으로 하였기에 변형 기반 반투명 퍼징의 특성을 가진다.
즉, 가지고 있는 입력 집합에서 하나를 선택한 후, 다양한 변종들을 생성하고 그 중 새로운 커버리지를 달성한 입력들만 다시 입력 집합에 추가하는 방식이다.
추가로 여기에 지향성의 개념을 부여하기 위해  목표와의 **거리**를 계산해 그 거리가 작은, 즉, 목표에 도달할 가능성이 더 큰 입력에 집중한다.
이 때, 각 입력과 목표 사이의 거리는 실행흐름그래프(Control Flow Graph)상에서 입력이 실행한 프로그램 지점들과 목표 지점 사이의 거리를 계산하여 산출한다.
AFLGo 이후로 대부분의 기법들도 이와 같은 방식을 채택해왔다. 그러나 이런 방식으로는 대상 프로그램이 크고 복잡할수록 목표 지점까지의 정확한 가이드를 제공하기 어렵다.
따라서 본 문서는 구체적으로 어떠한 문제가 있고, 왜 이러한 문제가 발생하는지, 그리고 DAFL이라는 새로운 기법은 이 문제를 어떻게 해결하는지 설명하려 한다.

## 3. 부정확한 거리 점수
앞서 언급하였듯이, 기존의 지향성 퍼징 기법들은 매 입력마다 목표와의 **거리**를 계산해 점수를 매긴다.
그리고 이 점수는 해당 입력을 통해 얼마나 많은 새로운 입력을 생성할 것인지 결정할 때 가중치로써 사용된다.
따라서 거리 점수는 지향성 퍼저가 탐색 공간에서 집중할 방향을 결정하는 요소가 된다.
그러나 프로그램이 크고 복잡해질수록 이 점수는 실제 목표와의 거리를 잘 대표하지 못하게 된다.
그 이유는 다음과 같다.
특정 입력과 목표 지점 사이의 거리는 해당 입력이 실행한 프로그램 지점들과 목표 지점 사이의 거리를 계산하여 산출한다.
그런데 크고 복잡한 프로그램에서는 각 입력의 실행이 길어진다. 그리고 실행이 길어질수록 목표 지점과는 연관성이 낮은 프로그램 지점들도 많이 실행된다.
이렇게 특정 입력의 거리 점수를 계산할 때 목표 지점과 연관성이 낮은 프로그램 지점들이 많이 개입될수록 해당 거리 점수가 가지는 의미는 퇴색된다.


### 3.1. 해결책: 의미 기반 연관성 점수 사용
DAFL은 기존의 거리 점수를 대체하기 위해 의미 기반 연관성 점수를 사용한다.
의미 기반 연관성 점수는 목표 지점과 의미적으로 연관된 프로그램 지점만 고려한 점수이다.
이를 위해 먼저 DAFL은 대상 프로그램의 정의-사용 관계를 분석함으로써 데이터 의존성 정보를 수집한다. 

```c
1 int x = input();
2 int y = x + 1;
```
위의 예제에서 y를 정의하기 위해 x가 사용되었다. 따라서 1번과 2번 라인 사이에 정의-사용 관계가 성립한다.
이러한 정의-사용 관계를 분석하면 각 프로그램 지점들 사이에 어떠한 데이터 의존성이 존재하는지 드러내는 정의-사용 그래프를 그릴 수 있다.
그 후 DAFL은 목표 지점을 기준으로 따라 올라가며 정의-사용 그래프를 잘라내어 목표 지점에 영향을 주는 프로그램 지점만 남긴다.
이렇게 잘라낸 그래프의 각 노드, 즉 프로그램 지점들은 목표 지점과 가까울수록 높은 점수를 부여받는다.
따라서 목표 지점과 가장 먼 프로그램 지점이 1점, 목표 지점은 가장 높은 점수를 부여받게 된다.
이 때, 어떠한 입력의 의미 기반 연관성 점수는 해당 입력이 실행한 프로그램 지점들의 점수의 합으로 계산된다.
즉, 특정 입력이 목표 지점에 더 직접적인 영향을 끼칠수록, 그리고 자주 끼칠수록 더 높은 점수를 부여받게 되는 것이다.
이로써 DAFL은 주어진 입력이 실행한 프로그램 지점들 중 목표 지점과 상관 없는 것들은 무시하고, 목표 지점과 연관된 프로그램 지점들만 고려하여 그 중요성을 평가한다.


## 4. 불필요한 커버리지 정보
기존의 지향성 퍼징 기법들은 AFL로 대표되는 변형기반 반투명 퍼징의 기본 원리를 그대로 채용해왔다. 즉, 새로운 커버리지를 달성한 변종은 모두 입력 집합에 추가한다.
이 말은 새로운 커버리지가 목표 지점과 연관이 없더라도 입력 집합에 추가된다는 것이다.
이러한 방식은 프로그램의 규모가 커질수록 지향성 퍼징에는 방해가 된다. 왜냐하면 프로그램이 커질수록 목표 지점과 연관이 없는 프로그램 지점들이 많아지기 때문이다.
예를 들어 입력 집합에 101개의 입력이 들어 있는데 목표 지점과 연관된 지점을 지나는 입력이 1개, 목표 지점과 연관이 없는 지점을 지나는 입력이 100개라고 하자.
이 때, 목표 지점과 연관된 지점을 지나는 입력에서 새로운 입력을 생성할 때 가중치를 10배로 준다고 가정하자.
그럼에도 불구하고 애초에 목표 지점과 연관이 없는 지점을 지나는 입력이 100배 더 많았기 때문에 최종적으로 목표 지점과 연관이 없는 지점을 지나는 입력이 더 많이 생성될 것이다. 이러한 입력들은 목표 지점과 연관이 없기 때문에 불필요한 입력이다.


### 4.1. 해결책: 선별적 커버리지 정보 사용
DAFL은 기존의 방식과는 다르게 애초에 목표 지점과 연관이 없는 프로그램 지점으로부터는 커버리지 정보를 수집하지 않는다.
이를 위해 DAFL은 앞서 언급한 정의-사용 그래프를 다시 한번 활용한다. 이때는 목표 지점을 기준으로 잘라낸 정의-사용 그래프가 지나는 함수들의 집합을 구한다.
즉, 목표 지점에 영향을 주는 함수들의 집합을 구하는 것이다. 그리고 이 함수들의 집합에 속하지 않는 함수들로부터는 커버리지 정보를 수집하지 않는다.
따라서 모든 커버리지는 목표 지점과 연관된 프로그램 지점들에서만 수집되며, 기존과 달리 목표 지점과 연관이 없는 프로그램 지점들을 새로 커버한 입력은 입력 집합에 추가되지 않는다.
결과적으로 입력 집합에 불필요한 입력을 추가하지 않으므로 퍼징의 효율성이 향상된다.

## 5. DAFL의 성능
DAFL은 의미 기반 연관성 점수와 선별적 커버리지 정보의 유용함을 보여주기 위한 도구로써 기존의 무지향성 퍼저인 AFL을 기반으로 구현되었다.
지향성 퍼저의 성능 평가는 주로 주어진 특정 결함을 얼마나 빨리 재현하는지를 기준으로 한다 ([지향성 퍼징 도구 평가](https://github.com/prosyslab/pl-wiki/wiki/지향성-퍼징-도구-평가) 문서를 참고).
따라서 DAFL의 성능을 평가하기 위해 41개의 실제 결함을 재현하는 실험을 진행하였는데 비교 대상으로는 무지향성 퍼저 AFL과 지향성 퍼저인 AFLGo, Beacon, WindRanger를 사용하였다.
그 결과, DAFL은 AFL, AFLGo, Beacon, WindRanger보다 각각 1.93, 4.99, 4.31, 그리고 20.08 배 빠르게 결함을 재현하였다.  
또한 의미 기반 연관성 점수와 선별적 커버리지 정보 각각 그 자체로도 성능 향상을 가져왔다.
구체적으로는 기반이 되는 AFL보다 각각 1.39와 1.73 배 빠르게 결함을 재현하였다.


## 6. 결론
본 문서에서는 기존 지향성 퍼징의 문제점을 분석하고, 이를 해결하기 위해 제안된 DAFL이라는 새로운 기법을 소개하였다.
기존 지향성 퍼징의 가장 큰 문제는 목표 지점과 연관된 프로그램 지점과 그렇지 않은 프로그램 지점을 구분하지 않았다는 것이다.
그러나 DAFL은 정의-사용 관계 분석을 통해 목표 지점과 연관된 프로그램 지점들을 파악하였고, 이를 각각 입력의 점수 (의미 기반 연관성 점수)와
커버리지 정보 수집의 기준 (선별적 커버리지 정보)에 활용하였다.
기존 지향성 퍼저들과 비교하여 DAFL은 41개의 실제 결함을 재현하는 실험에서 최소 1.93 배 빠르게 목표 결함을 재현하였다.

## 7. 참고

- [DAFL : Directed Grey-Box Fuzzing Guided by Data Dependency](https://prosys.kaist.ac.kr/publications/sec23.pdf), Kim et al., USENIX Security 2023

- [발표 영상](https://www.youtube.com/watch?v=BjtKhyzLtyo&t=16s), USENIX, 2023

- [발표 자료](https://prosys.kaist.ac.kr/publications/sec23-slides.pdf), Kim et al., 2023

## 기타

### 주제곡
AI 도구를 활용하여 DAFL의 주제곡을 생성했다.

먼저 본 문서의 내용을 기반으로 ChatGPT가 다음과 같이 가사를 생성하였다.
```
[Verse 1]
끝없는 미로 속에 찾는 길 하나
목표를 향해 나아가는 작은 입력들
하지만 길어지는 경로에 가려진
복잡한 그림자 속 흐려진 의미들

[Chorus]
우린 DAFL의 빛을 찾아
정의와 사용, 데이터의 연결 속에
의미 있는 거리, 그곳에 닿기 위해
불필요한 그림자, 이제는 버릴래

[Verse 2]
의미의 연관성, 그래프 속에 새겨진
목표와 닿은 지점에 더 큰 힘을 실어
멈춰버린 퍼저, 반복되는 입력들
우린 더 빠르고 정확히 그 결함을 향해

[Chorus]
우린 DAFL의 빛을 찾아
정의와 사용, 데이터의 연결 속에
의미 있는 거리, 그곳에 닿기 위해
불필요한 그림자, 이제는 버릴래

[Bridge]
선별된 길을 통해 우리는 간다
목표와 닿지 않는 모든 점들 버려가며
수많은 오류, 빠르게 정복하며
과거의 틀을 넘어선 새로운 길 위에

[Chorus]
우린 DAFL의 빛을 찾아
정의와 사용, 데이터의 연결 속에
의미 있는 거리, 그곳에 닿기 위해
불필요한 그림자, 이제는 버릴래

[Outro]
지향성 퍼징, 더 이상 흐릿하지 않아
DAFL과 함께 우린 목표에 닿아
정의와 사용, 의미 속의 연결로
더 나은 퍼징, 새로운 길을 향해
```

그 후, [Suno](https://suno.com/)라는 웹사이트에서 앞서 생성한 가사를 입력으로 주제곡 음원을 생성했다.
생성된 음원은 [유튜브](https://www.youtube.com/watch?v=tI6umHgKTL8)에서 확인할 수 있다.
