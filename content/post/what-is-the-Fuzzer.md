---
title: What is the Fuzzer?
date: 2020-10-23
hero: "/images/what-is-the-fuzzer-01.jpg"
excerpt: Article to explain justice of the fuzzer.
authors:
  - devmyong

---

.

# Fuzzing이란 무엇인가?

Fuzzing은 프로그램의 보안 취약점을 탐지하는 다양한 기법 중 하나입니다.

## Fuzzing의 시초

Fuzzing을 처음 언급한 곳은 1990년대 Barton P. Miller 교수의 논문 "An Empirical Study of the Reliability of UNIX Utilities"이고 이를 표현하기를, '대상 프로그램에서 사용할 임의의 문자 스트림을 생성하는 것'이라고 하였습니다. 이 사건 이후, 해당 단어가 본래 정의된 뜻 외에도 다양한 분야에서 많은 용도로 사용되게 됩니다.

## Fuzzing의 현대적 의미

현대에 들어서 밀러 교수님의 연구를 계승하면서, 동시에 가장 보편적으로 사용되는 정의는 '테스트 대상 프로그램의 예상 입력 범위를 벗어나는 입력을 사용해서 테스트 대상 프로그램을 반복하여 실행하는 것'입니다.

조금 더 Fuzzing의 특성을 담아서 다시 정의 내리자면, '테스트 대상의 잠재적 취약점을 식별하는데 사용되는 자동화된 소프트웨어 테스트 기술' 입니다. 즉, 프로그램이 예측하지 못한 값을 입력함으로써, 개발자가 의도하지 않은 행동을 유발할 수 있도록 한다는 것입니다. 중요한 점은, 잠재적 취약점을 자동화 하여 식별한다는 것입니다.

## Fuzzer와 Fuzz Algorithm

이러한 Fuzzing Test를 진행하는 프로그램을 Fuzzer라 부르며, Fuzzer를 구현할 때 사용된 알고리즘을 Fuzz Algorithm이라고 부릅니다. Fuzz Algorithm은 테스트 대상 프로그램으로 전달되는 변수에 따라 좌우됩니다. 즉, 테스트 대상을 괴롭히기 위해 어떤 입력을 사용 할 것인가? 라고 표현할 수 있습니다.

### Fuzz 환경설정

예리한 입력 값을 구성하기 위해 다양한 방식의 설정이 사용되기도 합니다. 

- Random value의 seed값을 언제 변경할 것인가?
- seed값을 어떻게 변경할 것인가?
- 입력된 값을 재사용 할 때 어떤 변화를 줄 것인가?
- 테스트 결과를 다음 반복 때 반영할 것인가?

위와 같이 값의 변경점을 설정하거나 추가적인 데이터를 받아들여 사용하거나 등 여러 방법이 존재합니다. 이를 Fuzz 환경 설정이라 하고, 이는 Fuzz Algorithm의 일부 입니다.

### Fuzzer의 일반적인 알고리즘

Fuzzer는 일반적으로 반복문 안에서 일련의 함수들이 수행되는 구조를 갖습니다. 각 단계가 반복 수행되는 것을 Fuzz iteration이라고 합니다. 이를 의사 코드로 표현하고, 각 단계를 설명해보겠습니다.

```cpp
/* psudo code */
Input : C, t    // Config, Time
Output : B      // Bugs

C <- Preprocess(C)    // (1)
while  (t.delta< t.limit) && IsContinue(C) {    // (6)
	conf <- Schedule(C, t.delta, t.limit)         // (2)
	tcs <- InputGen(conf)                          // (3)
	B`, execinfos <- InputEval(conf, tcs, O.bug)   // (4)
	C` <- ConfUpdate(C, conf, execinfos)           // (5)
	B <- Sum(B,B`)
}
return B
```

1. Preprocess(C) → C

    사용자가 Fuzz 환경설정으로 사용할 값들을 지정하면, 일련의 과정을 거쳐 적절히 수정한 상태로 값을 반환합니다.

2. Schedule(C, t.delta, t.limit) → conf

    현재의 Fuzz 환경설정(C)과 수행시간(t.delta), 종료시점(t.limit)을 입력으로 하여, 이번 Fuzz iteration에서 사용할 설정값 conf를 지정합니다.

3. InputGen(conf) → tcs

    Fuzz 환경설정을 바탕으로 일련의 테스트 케이스들(tcs)을 생성합니다. 테스트 케이스를 생성할 때, 현재 conf에 설정된 파라미터들을 이용합니다. 이 때, Fuzzer 중 일부는 seed 방식을 쓰기도 합니다.

4. InputEval(conf, tcs, O.bug) → B`, execinfos

    Fuzz 환경설정과 테스트 케이스, 발생이 예상되는 버그 시그니쳐 등을 입력받습니다. 이 단계에서 테스트 케이스를 대상 프로그램에 입력한 후, 해당 수행 과정에서 보안 정책 위반이 발생하는지를 버그 시그니쳐를 사용하여 점검합니다. 이 때, 버그가 발견되면 그 버그 결과와 해당 Fuzz run(All Fuzz iterations)에서 사용된 기타 정보들을 반환합니다. (bug oracle은 쉬운 예로 블루 스크린, segment fault입니다.)

5. ConfUpdate(C, conf, execinfos) → C

    Fuzz 환경설정(C)과 현재 설정된 값(conf), 그리고 각각의 fuzz run이 수행될 때의 관련 정보(execinfos)를 입력으로 하여, Fuzz 환경설정을 업데이트하여 반환합니다.

6. IsContinue(C)

    Fuzz 환경설정(C), 현재 설정된 값(conf)을 바탕으로 다음 Fuzz iteration을 수행할지를 결정합니다.

이제 일반적인 Fuzzer의 동작 과정을 알게 되었습니다. 시중에 존재하는 각 Fuzzer는 이러한 과정에서 어떻게 각 단계를 customize 했느냐? 혹은 불필요한 과정을 축소시키며 어떤 성능 향상을 이루는가? 등과 같이 목표하는 바를 달리 설정하며 각 과정을 가감할 수 있습니다. 

# Fuzzer의 분류

Fuzzer는 테스트 대상 프로그램이 필요합니다. Fuzzer의 테스트 대상은 소스 코드가 공개된 프로그램일 수도 있고, 아예 공개가 되지 않은 프로그램일 수도 있습니다. 이를 Black-box, White-box 환경이라 표현합니다. Fuzzer는 테스트 대상 프로그램의 내부 소스를 확인할 수 있는 가를 기준으로 Black-box, Grey-box, White-box Fuzzer로 나눕니다.

## Black-box Fuzzer

오직 테스트 대상 프로그램의 input과 output을 관찰하며 진행합니다. 이는 대상 내부의 설계나 소스 코드를 알 수 없는 경우에 사용하기 적합한 Fuzzer입니다. 블랙박스 환경의 대표적인 예로 MS-Windows가 있습니다. 과거 MS-MVP(Microsoft Most Valuable Professional)의 경우 한적인 경우에 한에 커널 소스가 공개된 적 있으나, 현재는 그렇지 않습니다.

## White-box Fuzzer

테스트 대상 프로그램의 내부 구조와 실행 중에 발생하는 정보들을 바탕으로 테스트 케이스를 생성하는 방법을 사용합니다. 다른 방법보다 테스트 대상 프로그램의 상태에 체계적이며 다각도로 접근하기 편합니다. 

## Grey-Box Fuzzer

테스트 대상 프로그램의 내부 구조와 실행되는 동안의 정보를 획득할 수 있으나, 화이트 박스 방식과 비교하면 테스트 대상 프로그램의 전체적인 범위를 알 수 없는 등 정보가 제한됩니다. 

# Fuzzer의 장단점

Fuzzer는 항상 최적의 결과를 내놓지는 않습니다. 테스트 대상 프로그램을 누구로 설정하느냐, 환경설정과 알고리즘에 어떤 customizing을 했느냐에 따라 Fuzzer의 성능이 좌우되긴 해도, Random 요소가 들어가기 때문에 결국은 운이 아니냐는 말도 있습니다. 그리고 Fuzzer가 모든 취약점을 발견하는 것은 아니므로, 다른 보안 조치들을 대체하는 기술이 아닙니다. 또한 Fuzzer가 더 이상 취약점을 찾지 못한다는 것이 프로그램의 완벽성을 나타내는 것도 아닙니다. 그 외에도 Fuzzer는 악용 불가능한 버그 생성이 가능하기 때문에, 모든 크래시가 가치 있는 것은 아닙니다.

그럼에도 불구하고 Fuzzer는 현대에 각광받는 취약점 분석 프로그램입니다. Fuzzing은 테스트를 무작위로 수행하는 기법이기 때문에 정의된 것만 테스트하는 접근방식으로는 해결할 수 없는 지점을 공략할 수 있습니다. 이론적으로 모든 유형의 버그를 찾을 수 있고, 특히 메모리 손상과 관련된 버그가 쉽게 감지되는 편입니다. 또한 Fuzzer는 확장성이 높습니다. 이는 더 많은 기기에서 프로그램을 가동시킴으로써 쉽게 확장시킬 수 있고, 이를 통해 테스트 시간 또한 쉽게 감소시킬 수 있습니다. 작업을 여러번 반복하는 것도 가능하고, 다시 구현하는 것도 가능하다는 것이 가장 큰 장점입니다.

# 실행 시 주의사항

Fuzzer를 만들었다고 해서 모든 서비스나 프로그램에 함부로 실행시켜선 안됩니다. 무작위 입력을 제공하여 수행하는 과정에서 프로그램의 충돌로 인해 메모리 손상이 일어나는 경우도 있기 때문입니다. 이는 정보통신망법 위반입니다. Fuzzer 사용자는 bug bounty와 연계되어있는 기업, 플랫폼을 통해 합법한 경로로 테스트를 진행해야 합니다.

---

# 참고 자료

- Fuzzing : Art, Science and Engineering
([https://arxiv.org/pdf/1812.00140.pdf](https://arxiv.org/pdf/1812.00140.pdf))
- Whit is a driver - Windows drivers
([https://docs.microsoft.com/ko-kr/windows-hardware/drivers/gettingstarted/what-is-a-driver-](https://docs.microsoft.com/ko-kr/windows-hardware/drivers/gettingstarted/what-is-a-driver-))
- What the Fuzz?
([https://labs.f-secure.com/blog/what-the-fuzz/](https://labs.f-secure.com/blog/what-the-fuzz/))
- Device Stack
([https://richong.tistory.com/268](https://richong.tistory.com/268))
- Windows Internals
([https://docs.microsoft.com/en-us/sysinternals/resources/windows-internals](https://docs.microsoft.com/en-us/sysinternals/resources/windows-internals))
