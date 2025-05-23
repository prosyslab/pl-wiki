## 1. 개요
번역 검산 (translation validation)은 컴파일러 등의 번역기가 원본 코드를 결과 코드로 잘 번역했는지 검사하는 기술이다.
1994년 Pnueli가 발표한 "Translation Validation"<sup>[1](#tv)</sup>에서 처음 제시되었다. 

## 2. 컴파일 = 번역
번역이란, 원본의 의미를 유지하면서 다른 언어로 형식을 변환해 주는 것이다.
여기서 가장 중요한 것은 의미를 유지한다는 것이다.
예를 들어, "화장실이 어디있나요?"를 번역한 결과물이 "Where is the restaurant?"라면 이는 안 하느니만 못한 번역이 될 것이다.
우리가 파파고나 구글 번역기를 좋아하는 이유는 위 한국어 문장과 의미가 같은 영어 문장을 제시해주기 때문이다. (물론 간혹 틀릴 때도 있다.)

개발자라면 누구나 사용하는 컴파일러 또한 번역기의 일종이다.
컴파일러의 역할을 떠올려 보자. 컴파일러는 개발자가 작성한 프로그램을 컴퓨터, 또는 다른 프로그램이 이해할 수 있는 프로그램으로 변환해주는 일을 한다.
당연히 이때 결과 프로그램의 의미는 원본과 같아야 할 것이다. 만약 이러한 성질이 보장되지 않는다면, 그 컴파일러는 마찬가지로 안 하느니만 못한 번역을 하게 될 가능성이 있다.
극단적인 예로, 사용자가 무슨 프로그램을 작성하든 컴파일러가 악성 소프트웨어로 컴파일할 수 있다. 이러한 맥락에서, 의미의 동일성은 컴파일러가 반드시 갖춰야 할 성질 중 하나로 여겨진다.

## 3. 검증과 검산
소프트웨어가 어떠한 성질을 만족하는지 보이는 분야로 소프트웨어 <b>검증 (Verification)</b>이 있다.
검증은 엄밀한, 수학적인 증명을 추구한다. 소프트웨어를 수식으로 기술하고, 대상 성질이 만족되는 지를 연역적으로 증명한다.
일단 검증이 되면 반례가 존재하지 않기에, 소프트웨어의 품질을 보증할 수 있는 가장 강력한 방법의 하나로 꼽힌다.

그런데 검증을 실용적으로 사용하려면 맞닥뜨리게 되는 몇 가지 장애물이 있다.
1. 소프트웨어의 설계가 복잡해지거나, 크기가 커지면 검증의 난이도가 극히 어려워진다.
2. 증명을 편하게 하기 위해 개발 단계에서부터 제한된 언어의 기능으로 프로그래밍을 해야할 수도 있다.
3. 아무리 사소한 업데이트라도, 일단 이뤄지면 해당 프로그램의 검증을 다시 해야할 수도 있다.

이러한 장애물은 개발자들에게 부담이 되기 쉽다. 특히 개발 속도, 비용 또는 성능을 중요시하는 일반적인 개발 환경에서는 검증을 사용하기 어렵게 한다.

위의 단점을 보완할 수 있는, 다소 완화된 증명책으로 <b>검산 (Validation)</b>이 있다.
검산은 검증과 달리 소프트웨어 자체를 증명하지 않는다.
대신, 특정 입력에 대한 해당 소프트웨어의 출력이 올바름을 보이는 것을 목적으로 한다.
예를 들어, 번역기가 있고, 원문과 번역문이 주어져있을 때, '과연 번역문과 원문의 의미가 같은가'를 보이는 것이 번역기의 검산에 해당한다.
번역기 설계와 구현에 독립적이므로, 개발자에게 가중되는 부담이 없다는게 큰 장점이다. 

단, 검산은 어디까지나 특정 입력에 대응되는 출력이 올바른가를 보이는 것이므로, 대상 소프트웨어의 성질을 증명하는 것과는 상관이 없음을 유의해야 한다.
다시 말해, 아무리 많은 입력과 출력 쌍으로 검산에 성공했더라도, 소프트웨어가 그 외의 입력에 대해 올바른 출력을 한다는 것은 보장할 수 없다.

## 4. 컴파일러와 번역 검산
컴파일러는 의미를 보존하면서 소스 프로그램을 번역해야 한다. 그런데 실제의 컴파일러는 보통 규모가 크고 복잡하므로, 이를 검증하는 것은 매우 어렵다.
그리고 검증할 수 있는 컴파일러를 만들도록 요구하는 것은 개발의 활력을 잃게 만들기 쉽다. 그래서 차선의 방법으로 Pnueli가 제시한 것이 입력 프로그램과 출력 프로그램만을 활용하는 번역 검산이다.

## 5. 번역 검산기의 구성 요소
번역 검산기를 만들기 위해서는 다음 세 요소가 필요하다.
1. 변환기: 원본 언어와 대상 언어의 의미 구조를 논리식으로 변환해주는 도구
2. 증명식: '올바른 번역이란 무엇인가'에 대한 정의
3. 증명기: 원문과 번역문을 논리식으로 변환한 결과에 대해, 2의 증명식이 만족됨을 보일 수 있는 증명 도구

## 6. 번역 검산기의 예
1. Alive2<sup>[2](#alive2)</sup>: LLVM 최적화기를 대상으로 하는 번역 검산기. 원본 언어와 대상 언어 모두 LLVM의 중간 언어이다. LLVM 중간 언어를 일차 논리식으로 변환해주는 변환기, 충족 가능성 문제 (SMT, Satisfiability Modulo Theorem)를 해결해주는 Z3를 증명기로 하여 구성되었다.

2. TurboTV<sup>[3](#TurboTV)</sup>: V8 자바스크립트 엔진의 JIT 컴파일러인 TurboFan을 대상으로 하는 번역 검산기. TurboFan 중간 언어 프로그램을 대상으로 하며, TurboFan의 올바른 최적화를 위해서 특정 최적화 전후 중간 언어 프로그램의 의미가 같은지 검사한다.
자세한 내용은 [TurboTV](https://github.com/prosyslab/pl-wiki/wiki/TurboTV) 참고.

## 7. 번역 검산기를 사용하는 예

1. - Optimuzz<sup>[4](#Optimuzz)</sup>: 컴파일러 최적화에 지향성 퍼징을 결합해 번역 검산을 위한 입력 프로그램을 생성하는 프레임워크. 목표 최적화를 실행하는 입력 프로그램을 생성해 최적화의 올바름을 검사한다.
자세한 내용은 [Optimuzz](https://github.com/prosyslab/pl-wiki/wiki/Optimuzz) 참고.


## 7. 참고 문서
[<a name="tv">1</a>] ["Translation Validation", A. Pneuli, M. Siegel, E. Singerman, TACAS 1998](https://link.springer.com/content/pdf/10.1007/BFb0054170.pdf)

[<a name="alive2">2</a>] ["Alive2: Bounded Translation Validation for LLVM", Nuno P.Lopes, Juneyoung Lee, Chung-Kil Hur, Zhengyang Liu, John Regehr, PLDI 2021](https://web.ist.utl.pt/nuno.lopes/pubs/alive2-pldi21.pdf)

[<a name="TurboTV">3</a>] ["Translation Validation for JIT Compiler in the V8 JavaScript Engine",
Seungwan Kwon, Jaeseong Kwon, Wooseok Kang, Juneyoung Lee, Kihong Heo,
ICSE 2024](https://prosys.kaist.ac.kr/publications/icse24.pdf)

[<a name="Optimuzz">4</a>] "[Optimuzz: Optimization-Directed Compiler Fuzzing for Continuous Translation Validation](https://prosys.kaist.ac.kr/publications/pldi25.pdf)", Kwon et al., PLDI 2025
