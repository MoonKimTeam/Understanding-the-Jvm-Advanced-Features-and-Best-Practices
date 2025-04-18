# 백엔드 컴파일과 최적화

바이트코드를 프로그래밍 언어의 중간 표현이라고 생각하면, 컴파일러가 클래스 파일을 로컬 환경에 맞는 네이티브 코드로 변환하는 과정을 전체 컴파일 과정의 백엔드로 간주할 수 있다. <br>
백엔드 컴파일러의 컴파일 성능과 최적화 품질은 상용 가상 머신의 우수성을 측정하는 핵심 지표다. 상용 가상 머신의 중추이자 기술 수준과 가치를 가장 잘 반영하는 기술인 것이다. <br>

## JIT 컴파일러

현재 가상 머신의 쌍두마차인 핫스팟과 OpenJ9은 자바 프로그램을 먼저 인터프리터로 해석해 실행한다. <br>
그런 다음 아주 자주 실행되는 메서드나 코드 블록이 발견되면 해당 코드를 네이티브 코드로 컴파일하고 다양한 최적화를 적용해 실행 효율을 높인다. <br>
이러한 코드 블록을 핫스팟 코드 또는 핫 코드라고 하며, 런타임에 이작업을 수행하는 백 엔드 컴파일러를 JIT 컴파일러라고 한다. <br>

### 인터프리터와 컴파일러

현재 주류 상용 가상머신인 핫스팟과 OpenJ9은 모두 인터프리터와 컴파일러를 함께 사용한다. <br>
프로그램을 빠르게 시작해야 할 때는 인터프리터가 먼저 나서서 컴파일 없이 곧바로 실행할 수 있다. <br>
프로그램이 시작된 후에는 시간이 흐를수록 컴파일러의 역할이 커진다. 점점 더 많은 코드를 네이티브 코드로 컴파일해 실행 효율을 높이는 것이다. <br>
또한 메모리가 부족한 환경에서는 인터프리터 방식으로 메모리를 절약할 수 있다.

프로그램 시작 응답 속도와 운영 효율 사이에서 최상의 균형을 찾기 위해 핫스팟 가상 머신은 컴파일 서브시스템에 계층형 컴파일 기능을 추가했다. <br>
계층형 컴파일은 컴파일과 최적화 규모와 소요 시간에 따라 다음과 같이 여러 단계의 수준으로 수행 된다.

- 계층 0: 인터프리터가 프로그램을 순수하게 해석 실행한다. 
- 계층 1: 클라이언트 컴파일러를 사용하여 바이트코드를 네이티브 코드로 컴파일하고 실행한다. 이때 간단하고 안정적인 최적화만 수행한다.
- 계층 2: 클라이언트 컴파일러를 사용한다. 메서드 및 반환 횟수 통계 등 몇 가지 성능 모니터 링만 수행한다.
- 계층 3: 여전히 클라이언트 컴파일러를 사용한다. 분기 점프와 가상 메서드 호출 버전 등 모든 성능 모니터링 정보를 수집한다.
- 계층 4: 서버 컴파일러를 사용한다. 서버 컴파일러는 성능 모니터 링 정보를 활용하여 더 오래 걸리는 최적화까지 수행한다.

> 계층형 컴파일이 도입된 뒤로는 인터프리터, 클라이언트 컴파일러, 서버 컴파일러가 협력해 동작하면서 핫 코드가 여러 번 컴파일될 수 있다. <br>
> 빠르게 컴파일할 때는 클라이언트 컴파일러를 사용하고, 성능을 더 높여야 할 때는 서버 컴파일러를 사용한다. <br>
> 서버 컴파일러가 매우 복잡한 최적화 알고리즘을 수행해야 할 때는 우선 클라이언트 컴파일러로 간단한 최적화를 해 놓고, 복잡한 최적화는 느긋하게 마무리하는 방식도 가능하다.

### 컴파일 대상과 촉발 조건

런타임에 JIT 컴파일러가 컴파일하는 대상을 핫 코드라고 한다. 다음은 핫 코드의 가장 대표적인 유형 이다.

1. 여러번 호출되는 메서드
2. 여러번 실행되는 순환문의 본문

두 유형 모두에서 컴파일 대상은 개별 순환문의 본문이 아니라 메서드 전체다. <br>
첫 번째 유형의 컴파일은 메서드 호출에 의해 촉발되므로 컴파일러는 메서드 전체를 컴파일 대상으로 삼는다. 이는 가상 머신의 표준 JIT 컴파일 방법 이기도 하다. <br>
두 번째 유형의 컴파일은 순환문 본문에 의해 촉발되지만 컴파일 대상은 여전히 메서드 전체다.

특정 코드 블록이 핫 코드인지 그래서 JIT 컴파일을 촉발시켜야 하는지 판단하는 동작을 **핫스팟 코드 탐지 또는 핫스팟 탐지**라고 한다. <br>
다음은 현재 핫스팟 탐지에 주로 쓰이는 방식들이다.

- 샘플 기반 핫스팟 코드 탐지: 각 스레드의 호출 스택 상단을 샘플링하여 특정 메서드 또는 메서드의 일부가 자주 발견되면 해당 메서드를 핫 메서드로 간주한다.
- 카운터 기반 핫스팟 코드 탐지: 각 메서드와 코드 블록에 대한 카운터를 설정하고 나서 개별 실행 횟수를 기록한다. 그러다가 실행 횟수가 문턱값을 초과하면 핫 메서드로 간주한다.

**J9은 샘플 방식을 사용하고 핫스팟 가상 머신은 카운터 방식을 사용한다.**

### 컴파일 과정

기본적으로 JIT 컴파일은 백그라운드에서 별도 스레드가 진행하며 , 컴파일이 완료될 때까지는 인터프리터가 프로그램 실행을 이어 간다. <br>
그렇다면 백그라운드 컴파일 과정에서 컴파일러가 하는 일은 정확히 무엇일까? <br>
서버 컴파일러와 클라이언트 컴파일러의 컴파일 절차는 다르다.

#### 클라이언트 컴파일러의 컴파일 과정

클라이언트 컴파일러는 비교적 간단하고 빠른 3단계 컴파일러다. 오래 걸리는 전역 최적화는 포기하고 지역 최적화에 집중한다.

- 단계 1: 플랫폼 독립적 프런트엔드가 바이트코드로부터 타깃 독립적 중간 표현인 HIR을 생성한다. HIR은 코드 값을 정적 단일 할당(SSA)으로 표현해 주어 몇 가지 최적화를 더 쉽게 구현할 수 있게 도와준다. 물론 바이트코드를 HIR로 구성하기 앞서 메서드 인라인이나 상수 전파 등 몇 가지 기본적인 최적화는 수행한다.
- 단계 2: 플랫폼 의존적 백엔드가 HIR로부터 LIR을 생성한다. LIR 생성 전에 HIR을 대상으로 null 검사 제거와 범위 검사 제거 등의 최적화를 수행하여 HIR이 코드를 더 효율적으로 표현하도록 최 적화한다.
- 단계 3: 플랫폼 의존적 백엔드가 선형 스캔 레지스터 할당을 사용하여 UR에 레지스터를 할당하고 핍홀 최적화를 수행한 다음, 네이티브 코드를 생성한다.

#### 서버 컴파일러의 컴파일 과정

서버 컴파일러는 서버용 애플리케이션들의 일반적인 시나리오를 감안하여 서버 측 성능을 극대화하도록 설정된 컴파일러다. <br>
죽은 코드 제거, 순환문 언롤링, 순환문 표현식 호이스팅, 공통 하위 표현 제거, 상수 전파, 기본 블록 재정렬 등 전통적인 최적화를 대부분 수행하며 <br>
범위 검사 제거, null 검사 제거 등 자바 언어에 특화된 최적화도 수행한다. <br>
나아가 인터프리터나 클라이언트 컴파일러가 제공하는 성능 모니터 링 정보를 토대로 가이디드 인라인과 분기 예측처럼 안정성이 다소 떨어지는 예측 최적화도 수행할 수 있다.

## AOT 컴파일러

### AOT 컴파일의 장점과 단점

#### AOT의 매력

AOT 컴파일러 관련 연구는 크게 두 가지 형태로 나뉜다. <br>
하나는 프로그램이 실행되기 전에 프로그램 코드를 네이티브 코드로 컴파일하는 형태로, 기존 C/C++ 컴파일러와 비슷하다. <br>
또 다른 하나는 원래 JIT 컴파일러가 런타임에 수행해야 하는 작업을 미리 수행해 캐시에 저장해 두고 다음번 실행 시 사용하는 형태다.

프로그램 실행에 앞서 네이티브 코드로 컴파일하는 것은 AOT의 전통적인 형태이자 JIT 방식의 가장 큰 약점에 해당한다. <br>
JIT 컴파일은 애플리케이션이 실행되는 동안 컴파일러도 뒤에서 컴퓨팅 자원을 소비한다. <br>
최신 JIT 컴파일러들은 계층형 컴파일을 지원하여 품질은 낮지만 빠른 JIT 컴파일러를 먼저 수행하고, 고품질 JIT 컴파일러가 더 나은 결과를 준비할 시간을 벌어 준다. <br>
**하지만 어떤 경우든 JIT 컴파일에 소요되는 시간은 애플리케이션 실행에 사용할 수 있었던 시간이다.** <br>
**JIT 컴파일에 소비된 컴퓨팅 자원 역시 애플리케이션 실행에 사용할 수 있었던 자원임은 변함이 없다.**

JIT 컴파일러가 런타임에 수행해야 하는 작업을 미리 수행하여 캐시해 두는 것은 캐시 역할을 극대화하여 자바 프로그램 구동 시간을 단축하고, 구동 후 빠르게 최상의 성능을 내는 것이다. <br>
현재 주류 상용 JDK들은 이 방식의 고급 컴파일을 모두 지원한다. <br>
**그런데 이런 AOT 컴파일은 대상 물리 머신뿐 아니라 핫스팟 가상 머신의 런타임 매개 변수도 고려해야 해서 실제로 적용하기는 쉽지 않다.** <br>
예를 들어 가상 머신은 런타임에 여러 종류의 가비지 컬렉터를 이용할 수 있는데, 어떤 가비지 컬렉터는 JIT 컴파일 서브시스템을 이용한다. <br>
AOT 컴파일을 원한다면 이러한 협업 과정을 적절히 변환해 처리해야 한다. 플랫폼 중립성을 해친다는 점과 바이트 팽창 같은 단점도 여전하다.

#### JIT의 반격

JIT가 AOT보다 본질적으로 나은 점 세 가지를 살펴보자.

**첫 번째는 성능 모니터링 기반 최적화다.** <br>
핫스팟의 JIT 컴파일러는 인터프리터나 클라이언트 컴파일러가 실행되는 동안 다양한 성능 모니터링 정보를 수집한다. <br>
예를 들어 프로그램이 참조하는 추상 클래스의 실제 타입, 주로 선택되는 조건 분기, 메서드 호출 시 주로 선택되는 버전, 순환문의 일반적인 반복 횟수 같은 정보를 취합한다. <br>
해당 경로들을 핫 코드로 지정하여 더 많은 자원(분기 예측, 레지스터, 캐시 등)을 배분해 최적화할 수 있다.

**두 번째는 많은 JIT 컴파일 최적화 측정의 기초가 되는 급진적 예측 최적화다.** <br>
JIT 컴파일은 AOT 컴파일처럼 보수적일 필요가 없다. 100% 정확하지는 않더라도 성능 모니터링 정보를 토대로 높은 확률로 정확한 판단을 내릴 수 있다. <br>
가능성 높은 가정을 믿고 과감하게 최적화하는 것이다. 그리고 혹시라도 낮은 확률의 동작이 실행된다면 하위 계층 컴파일러나 인터프리터로 실행하는 식이다.

**세 번째는 링크타임 최적화다.** <br>
자바 언어는 본질적으로 동적으로 링크된다. 클래스가 런타임에 가상 머신 메모리에 로드된 다음 JIT 컴파일러에 의해 최적화된 네이티브 코드로 만들어진다. <br>
AOT 컴파일을 사용하는 언어와 프로그램에서 이와 같은 시나리오가 발생했다고 해 보자. <br>
예를 들어 C/C++ 프로그램이 특정 동적 링크 라이브러리의 특정 메서드를 호출하려는 경우는 확실한 경계가 있어서 최적화하기 어렵다. <br>
메인 프로그램의 코드와 동적 링크 라이브러리는 완전히 독립적으로 컴파일되고 자신만을 고려해 최적화되기 때문이다.


















