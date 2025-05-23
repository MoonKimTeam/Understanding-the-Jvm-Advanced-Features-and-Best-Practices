# 백엔드컴파일과 최적화

## JIT 컴파일러

JIT 컴파일러는 자바 가상 머신의 핵심적인 최적화 수단으로, 핫스팟(HotSpot)이나 OpenJ9 같은 JVM 구현체에서 동작한다. 이러한 가상 머신은 자바 프로그램을 처음 실행할 때 인터프리터를 통해 바이트코드를 한 줄씩 해석하며 실행한다. 그러나 특정 메서드나 코드 블록이 자주 실행되는 것으로 판단되면, 해당 부분을 네이티브 코드로 변환하여 더 빠르게 실행할 수 있도록 한다. 이 과정을 수행하는 컴포넌트가 바로 JIT(Just-In-Time) 컴파일러이다.

이러한 최적화된 코드 블록을 흔히 ‘핫스팟 코드’ 또는 ‘핫 코드’라고 부른다. 이 핫 코드는 인터프리터 방식보다 훨씬 높은 실행 효율을 보여주며, 실행 중에 선택적으로 컴파일된다는 특징 때문에 JIT 컴파일러를 ‘적시 컴파일러’라고도 부른다. JIT 컴파일러는 단순한 번역기로서의 역할을 넘어서, 다양한 실행 최적화 기법을 적용하여 런타임 성능을 끌어올린다. 예를 들어 메서드 인라이닝, 루프 언롤링, 불필요한 객체 제거, 조건문 분기 최적화 등이 이에 포함된다.

JIT 컴파일러는 인터프리터와 병행되어 동작한다. 처음에는 인터프리터가 빠르게 실행을 시작하고, JIT 컴파일러는 해당 코드의 실행 통계를 수집한 뒤 일정 기준 이상으로 자주 실행되는 코드에 대해 컴파일을 수행한다. 이러한 병행 구조 덕분에 JVM은 빠른 시작 시간과 실행 중 성능 향상이라는 두 마리 토끼를 모두 잡을 수 있다.

또한, 핫스팟 가상 머신은 상황에 따라 다양한 JIT 컴파일러를 사용할 수 있도록 설계되어 있다. 이는 다양한 하드웨어 및 실행 환경에 최적화된 실행 성능을 확보하기 위한 전략이다. 예를 들어 서버 환경에서는 정교하고 깊은 최적화를 수행하는 C2 컴파일러가, 클라이언트 환경에서는 빠른 컴파일을 우선시하는 C1 컴파일러가 선택될 수 있다.

프로그램 실행 도중 어느 시점에 인터프리터가 사용되고 어느 시점에 JIT 컴파일러가 개입되는지, 또 어떤 코드가 네이티브로 컴파일되는지는 JVM 내부의 판단 로직과 설정에 따라 결정된다. 이러한 내부 동작은 흔히 ‘JIT 컴파일 임계값’이나 ‘트리거 조건’이라는 이름으로 설정할 수 있으며, JVM 옵션을 통해 제어 가능하다.

### 인터프리터와 컴파일러

자바 가상 머신에서 인터프리터와 컴파일러는 각각 독립된 방식이 아니라, 함께 협력하여 프로그램 실행 성능을 극대화하기 위해 병렬적으로 동작한다. 자바 프로그램이 처음 실행되면 인터프리터가 바이트코드를 한 줄씩 해석하며 빠르게 실행을 시작한다. 이 방식은 초기 구동 속도를 확보하기에 유리하다. 그러나 프로그램 실행 도중 특정 메서드나 코드 블록이 반복적으로 실행되는 ‘핫 코드’로 판단되면, 이를 네이티브 코드로 변환해 실행 성능을 높인다. 이 작업은 바로 JIT(Just-In-Time) 컴파일러가 수행한다.

JIT 컴파일러는 단순히 코드를 네이티브로 변환하는 것을 넘어, 메서드 인라이닝, 루프 언롤링, 불필요한 객체 제거, 조건문 분기 최적화 등 다양한 실행 시간 최적화 기법을 적용하여 성능을 대폭 향상시킨다. 이렇게 최적화된 코드는 핫스팟 코드라고도 불리며, 실행 효율이 매우 높다. JIT 컴파일러는 특정 코드가 일정 횟수 이상 실행되었는지를 기준으로 컴파일 여부를 결정하는데, 이러한 기준은 JIT 컴파일 임계값 또는 트리거 조건이라 불리며 JVM 옵션으로 제어할 수 있다.

핫스팟 JVM은 이러한 JIT 컴파일을 유연하게 적용하기 위해 두 가지 컴파일러를 제공한다. C1 컴파일러는 클라이언트 환경에 적합한 빠른 컴파일러이며, C2 컴파일러는 서버 환경에 적합한 고성능 최적화 컴파일러이다. 실제로는 두 컴파일러가 계층형 컴파일 체계로 함께 동작한다. 즉, 실행 초기에는 인터프리터로 빠르게 시작하고, 이후 C1 컴파일러가 가벼운 최적화를 수행하며, 코드가 더욱 자주 실행되면 C2 컴파일러가 깊은 최적화를 수행하는 방식이다.

계층형 컴파일 체계에서는 인터프리터, 클라이언트 컴파일러(C1), 서버 컴파일러(C2)가 협력적으로 동작한다. 예를 들어, 서버 컴파일러가 수행해야 할 복잡한 최적화를 C1이 대신 수행하는 것이 아니라, 먼저 C1이 가볍게 최적화를 수행한 후, C2가 나중에 더 무거운 최적화를 적용하는 식이다. 이 방식은 빠른 초기 실행과 높은 장기 성능을 동시에 추구할 수 있게 한다.

핫스팟 JVM은 실행 환경과 하드웨어 특성에 따라 어떤 컴파일러를 우선 사용할지를 결정하고, 필요에 따라 동적으로 다른 컴파일러로의 전환도 가능하도록 설계되어 있다. 이렇게 인터프리터와 JIT 컴파일러가 협력 구조로 동작함으로써, JVM은 자바의 플랫폼 독립성과 이식성은 유지하면서도 높은 실행 성능을 달성할 수 있게 된다.

### 컴파일 대상과 촉발 조건 

JIT 컴파일러가 실행 중 특정 코드를 네이티브 코드로 변환하는 대상은 ‘핫 코드’라 불린다. JIT 컴파일을 촉발하는 대표적인 조건은 두 가지다. 첫째, 여러 번 호출되는 메서드이며, 둘째, 반복적으로 실행되는 순환문이다. 첫 번째 조건은 메서드 호출 횟수가 누적되며 자연스럽게 핫 코드로 간주되는 경우다. 두 번째는 루프나 반복문이 많이 실행되는 경우로, 코드 일부만 반복되더라도 해당 메서드 전체가 JIT 컴파일 대상이 되는 경우가 많다.

JIT 컴파일러는 메서드 전체를 컴파일 단위로 삼기 때문에, 순환문의 실행 횟수가 아무리 많더라도 해당 순환문을 포함한 메서드 전체가 컴파일된다. 이 방식은 메서드가 첫 실행되는 진입점(바이트코드 시작 지점)을 기준으로 판단되며, 실행 중인 메서드가 아직 스택 프레임 위에 존재하는 상황에서 컴파일이 수행되면 곧바로 온스택 치환(on-stack replacement)이 발생한다.

JIT 컴파일이 실제로 트리거되는 시점과 조건은 ‘얼마나 자주 호출되었는가’에 대한 판단으로 결정된다. 그러나 이 판단을 위해 호출 횟수를 명시적으로 셀 필요는 없다. 대신, JVM은 핫 코드를 탐지하는 두 가지 주요 방식을 활용한다. 첫째는 샘플 기반 방식으로, 각 스레드의 호출 스택을 주기적으로 샘플링하여 자주 등장하는 메서드를 핫 코드로 간주한다. 구현이 간단하고 효율적이나, 정확도가 떨어질 수 있고 외부 요인의 영향을 받는다.

둘째는 카운터 기반 방식으로, 각 메서드나 코드 블록에 대한 실행 횟수를 수치로 기록한다. 실행 횟수가 임계값을 초과하면 해당 메서드를 핫 코드로 간주하고 컴파일을 트리거한다. 이 방식은 구현이 복잡하더라도 보다 정확하고 강력한 결과를 제공한다.

#### 메서드 호출 카운터에 의한 JIT 컴파일 촉발

JIT 컴파일러는 메서드가 자주 호출될 경우 이를 감지하여 해당 메서드를 네이티브 코드로 컴파일하는데, 이를 위한 핵심 기준이 ‘메서드 호출 카운터’이다. 메서드 호출 카운터에는 말 그대로 메서드가 호출된 횟수가 기록되며, 이 값이 특정 임계값을 초과할 경우 JIT 컴파일러는 컴파일 요청을 수행한다. 이 임계값은 클라이언트 모드에서는 기본적으로 1500회, 서버 모드에서는 1만 회로 설정되어 있으며, 필요에 따라 -XX:CompileThreshold 옵션을 통해 수동 조정도 가능하다.

메서드가 호출되면 JVM은 해당 메서드에 대해 이미 JIT 컴파일된 버전이 존재하는지를 먼저 확인한다. 존재한다면 해당 네이티브 코드를 실행하며, 존재하지 않을 경우 호출 카운터를 1 증가시킨다. 이후 카운터 값이 백 에지(back edge) 카운터와 함께 문턱값을 넘으면 JIT 컴파일러에 컴파일을 요청하게 된다. 이 과정은 런타임에서 백그라운드로 비동기적으로 수행되며, 컴파일이 완료되기 전까지 해당 메서드는 인터프리터 방식으로 계속 실행된다. 컴파일이 완료되면 시스템은 자동으로 메서드의 진입 지점을 새 주소로 갱신하여 이후부터는 네이티브 코드를 사용하게 된다.

카운터 방식은 단순히 횟수를 누적하는 방식 외에 ‘카운터 감쇠’(counter decay) 기능도 갖추고 있다. 이는 일정 시간 동안 메서드가 호출되지 않으면 호출 카운터를 절반으로 줄이는 기능이며, 이를 통해 오래 전에 자주 호출되었지만 현재는 거의 사용되지 않는 코드에 대한 컴파일을 방지한다. 이 기능은 -XX:+UseCounterDecay 옵션으로 비활성화할 수 있으며, 반감기 시간은 -XX:CounterHalfLifeTime으로 초 단위 설정이 가능하다.

#### 백 에지 카운터에 의한 JIT 컴파일 촉발

백 에지 카운터는 특정 순환문의 본문 코드가 실행되는 횟수를 계산하는 데 사용되며, 그 목적은 온스택 치환(On-Stack Replacement, OSR) 컴파일을 촉발하는 것이다. 즉, 인터프리터가 반복 실행하는 순환문이 충분히 많이 실행됐다고 판단되면 해당 반복 실행 부분만을 네이티브 코드로 치환해 실행 효율을 높이려는 것이다. 이를 위해 백 에지 카운터는 순환문의 반복 실행 횟수를 누적 측정한다.

과거에는 백 에지 카운터의 임계값을 직접 설정하는 -XX:BackEdgeThreshold 옵션이 있었으나, 현재는 -XX:OnStackReplacePercentage 옵션을 통해 간접적으로 조절한다. OSR이 촉발되는 조건은 메서드 호출 카운터와 백 에지 카운터의 값을 조합한 수치가 일정 문턱값을 넘을 때이다. 클라이언트 모드에서는 단순히 두 값의 곱을 100으로 나눈 수치가 기준을 넘으면 되고, 서버 모드에서는 여기에 인터프리터 모니터링 비율도 고려해 OSR 조건을 정한다.

예를 들어, 클라이언트 모드에서 OSR 비율이 933이고, 메서드 호출 카운터 문턱값이 1500일 경우 백 에지 카운터 문턱값은 약 13,995가 된다. 서버 모드의 경우 OSR 비율이 140이고 인터프리터 모니터링 비율이 33이므로, 백 에지 카운터의 문턱값은 약 10,700이다.

인터프리터는 루프 실행 중 백 에지 명령어를 만나면 해당 지점의 네이티브 코드가 존재하는지 확인하고, 없으면 백 에지 카운터를 1 증가시킨다. 이때 두 카운터의 합이 문턱값을 넘으면 컴파일러에 OSR 컴파일을 요청하고, 카운터 값을 조정한 후 인터프리터 방식으로 코드를 계속 실행한다. 컴파일이 완료되면 메서드 주소가 시스템 레벨에서 변경되고, 이후 호출부터는 컴파일된 버전이 사용된다.

이때 백 에지 카운터는 메서드 호출 카운터와 달리 감쇠(decay)를 적용하지 않고 절대 실행 횟수를 기준으로 동작하며, 한번 임계값을 초과하면 메서드 호출 카운터도 오버플로 상태로 전환되어 표준 컴파일 절차로 이어질 수 있다. 정리하면 백 에지 카운터는 루프 중심의 코드 최적화에 중점을 둔 OSR 컴파일의 핵심 지표로 작용하며, JIT 컴파일 타이밍을 보다 세밀하게 조정하는 데 기여한다.

### 컴파일 과정

JIT 컴파일은 기본적으로 백그라운드 스레드에서 수행되며, 컴파일이 완료되기 전까지는 인터프리터가 프로그램 실행을 계속 이어 간다. 메서드 호출에 의해 촉발된 컴파일 요청이나 OSR 요청은 여기에서 마무리된다. 만약 백그라운드 컴파일을 원하지 않는 경우, -XX:-BackgroundCompilation 옵션으로 이를 비활성화할 수 있다. 이 상태에서는 컴파일 요청을 한 스레드가 컴파일이 완료될 때까지 대기하게 되며, 그 이후에 생성된 네이티브 코드를 실행한다.

백그라운드 컴파일이 내부적으로 어떤 작업을 수행하는지 이해하기 위해서는 서버 컴파일러와 클라이언트 컴파일러의 차이를 알아야 한다. 클라이언트 컴파일러는 빠르고 간단한 세 단계로 구성된다. 첫 번째 단계에서는 바이트코드를 SSA 기반의 중간 표현(HIR)으로 변환하고, 기본적인 인라이닝과 상수 전파 같은 최적화를 수행한다. 두 번째 단계에서는 HIR에서 불필요한 검사 등을 제거하고 더 간결한 LIR로 변환한다. 마지막 세 번째 단계에서는 레지스터 할당과 펄핑 최적화를 수행하고, 네이티브 코드로 변환한다. 클라이언트 컴파일러는 전체 컴파일 속도가 빠르며 로컬 최적화에 집중한다.

반면, 서버 컴파일러는 복잡하고 정교한 최적화를 수행한다. 서버 측 성능 극대화를 목표로 하며, 많은 전통적인 최적화(루프 언롤링, 범위 검사 제거, null 검사 제거, 공통 하위 표현식 제거 등)를 적용한다. 일부 최적화는 자바 언어에 특화되어 있고, 실행 중 인터프리터나 클라이언트 컴파일러가 수집한 실행 프로파일링 정보를 바탕으로 예측 기반 최적화도 가능하다. 서버 컴파일러는 클라이언트 컴파일러에 비해 상대적으로 느리지만, 생성된 네이티브 코드의 품질이 더 높다.

JDK 9부터는 기본 모드가 서버 모드로 설정되었고, 클라이언트 컴파일러의 단점을 보완하면서도 더 나은 실행 성능을 제공하고자 하였다. 컴파일 과정은 기술적으로 매우 복잡하고, 일반적인 개발자는 이를 명확히 인식하지 못한 채 사용하겠지만, JVM 내부에서 매우 핵심적인 최적화 경로에 해당한다. 

### 실전: JIT 컴파일 결과 확인 및 분석

JIT 컴파일 과정은 자바 가상 머신 내부에서 사용자나 애플리케이션의 인지 없이 자동으로 진행된다. 프로그램은 처음에는 인터프리터로 실행되며, 일정 조건이 만족되면 해당 메서드를 네이티브 코드로 컴파일하여 성능을 향상시킨다. 이러한 컴파일 작업은 일반적으로 디버그용 출력이 없이는 외부에서 확인하기 어렵지만, HotSpot 가상 머신은 이를 관찰할 수 있는 다양한 옵션을 제공한다. 대표적으로 -XX:+PrintCompilation 옵션을 통해 어떤 메서드가 언제 컴파일되었는지 확인할 수 있다.

컴파일된 메서드는 여러 번의 컴파일 단계를 거칠 수 있으며, 각 단계는 클라이언트 컴파일러인지, 서버 컴파일러인지에 따라 최적화 수준이 다르다. 클라이언트 컴파일러는 빠른 반응 속도를 중시하고, 서버 컴파일러는 복잡하고 깊은 최적화를 수행한다. JIT 컴파일러는 코드 블록의 실행 횟수, 루프의 반복 횟수, 메서드 호출 횟수 등을 모니터링하여 ‘핫 코드’라고 판단되면 컴파일을 트리거한다. 이러한 조건은 기본적으로 설정된 임계값을 넘는 순간 발생하며, 사용자는 JVM 옵션을 통해 이를 조정할 수 있다.

실제로 어떤 메서드가 컴파일되었는지는 -XX:+PrintCompilation으로 출력된 로그를 통해 확인할 수 있다. 이 로그에는 메서드 이름, 바이트코드 크기, 컴파일 ID, OSR 여부 등 다양한 정보가 함께 제공된다. 특히 OSR(온스택 치환)이 발생한 경우, 어느 시점의 바이트코드에서 치환되었는지를 알려주는 BCI 정보가 함께 출력된다. 이 정보를 통해 어떤 루프 내부에서 컴파일이 수행되었는지도 유추할 수 있다.

실험 예제에서는 doubleValue()와 calcSum() 메서드를 작성하여 각각 10만 번, 1만 5천 번 호출되도록 하였고, 그 결과 두 메서드 모두 JIT 컴파일이 수행되었다. 또한 여러 번 컴파일된 후 일부 최적화 결과가 없어서 ‘not entrant’ 상태로 변경되었으며, 이는 인라이닝 최적화나 특정 정책에 따른 컴파일 취소가 있었음을 나타낸다.

추가적으로 -XX:+PrintInlining 옵션을 사용하면 각 메서드가 인라인 되었는지 여부를 알 수 있으며, 인라이닝이 실패한 이유도 함께 제공된다. 이 정보를 통해 개발자는 어떤 메서드가 인라인되지 않았는지, 어떤 정책이나 조건으로 인해 인라이닝이 거부되었는지를 알 수 있다. 또한 -XX:+PrintOptoAssembly와 같은 옵션은 최적화된 어셈블리 코드를 출력하여 보다 세부적인 분석이 가능하게 한다.

보다 고급 분석을 위해 IGV(Ideal Graph Visualizer) 도구를 사용할 수 있으며, -XX:+PrintIdealGraphFile과 같은 옵션을 통해 컴파일러의 중간 표현(HIR, LIR 등)의 최적화 과정을 시각적으로 확인할 수 있다. IGV는 컴파일 단계마다의 최적화 상태를 그래프 형태로 보여주며, 각 단계에서 어떤 블록이 유지되거나 제거되었는지를 비교하는 기능도 제공한다.

## AOT 컴파일러 

AOT 컴파일러는 자바 기술 시스템에서 새로운 주제가 아니다. 자바가 공식적으로 구동 환경을 갖춘 1996년 JDK 1.0의 출시와 함께 자바의 실행 방식에 대한 논의가 시작되었고, 이후 1996년 7월에 등장한 JDK 1.0.2에서는 최초의 JIT 컴파일러가 탑재되었다. 하지만 이보다 먼저, IBM이 자사 자바 언어용으로 AOT(Ahead-Of-Time) 컴파일러를 개발하여 High Performance Compiler for Java라는 이름으로 선보였다. 이처럼 자바 진영에는 JIT 컴파일러보다 앞서 AOT 컴파일러가 존재했으며, 1998년에는 GCC에 자바용 AOT 컴파일러 GCJ가 추가되었다. 이 GCJ는 OpenJDK가 배포되기 이전까지 리눅스 기반 자바 구현의 핵심 요소 중 하나였다.

그러나 자바 생태계 내에서 AOT 컴파일러는 큰 주목을 받지 못했다. 자바는 ‘한 번 작성하면 어디서든 실행된다’는 플랫폼 독립성을 핵심으로 하였고, 이로 인해 실행 환경에서의 즉시 컴파일을 통한 유연성과 이식성이 중시되었기 때문이다. 그 결과 GCJ가 등장한 이후 15년간 AOT 컴파일러에 대한 논의는 거의 사라졌다.

이러한 흐름이 바뀌기 시작한 계기는 2013년 안드로이드의 ART(안드로이드 런타임) 도입이었다. ART는 AOT 컴파일러 기반으로, JIT 대신 미리 컴파일된 네이티브 코드를 사용하는 구조를 채택하였다. ART의 등장은 기존 달빅 가상 머신(Dalvik VM)을 완전히 대체하며 안드로이드 실행 구조의 혁신을 가져왔다. 특히 안드로이드 4.4 이후 버전에서는 AOT 방식의 ART가 기본 런타임으로 채택되었고, 이는 안드로이드 성능을 획기적으로 향상시키는 계기가 되었다.

물론 안드로이드는 자바와 완전히 같다고 할 수는 없으나, 바이트코드 구조나 GC 메커니즘 등에서 상당 부분 자바와 유사한 구조를 유지하고 있었고, 이를 통해 자바 AOT 컴파일의 실행 가능성을 다시 주목받게 되었다. 다만 AOT 컴파일이 모든 상황에서 무조건 JIT보다 뛰어나다고 말하기는 어렵다. 플랫폼 독립성 문제, 바이트 부풀기(byte bloat), 동적 확장성 제약 등은 여전히 AOT 컴파일러의 단점으로 지적된다.

결국 AOT 컴파일러는 특정 환경, 특히 자원 제약이 크거나 예측 가능한 실행 환경에서 강점을 가지는 반면, JIT 컴파일러는 일반적인 범용성과 동적 최적화 측면에서 여전히 유리하다. 이러한 점에서 AOT는 JIT의 대체제가 아닌 보완적인 역할로 이해하는 것이 타당하며, 모든 상황에서의 완벽한 대체제로 보기에는 무리가 따른다.

### AOT 컴파일의 장점과 단점

AOT 컴파일러는 자바 기술 시스템에서 새로운 주제는 아니며, 이미 1996년 JDK 1.0이 출시되었을 당시 자바가 정식 구동 환경을 얻은 이후 AOT 컴파일러 형태는 존재했다. 그 시기에 나온 JIT 컴파일러보다 약간 앞선 시점인 JDK 1.0.2가 1996년 7월에 출시되었고, 불과 몇 개월 후 IBM이 자바 언어용으로 최초의 AOT 컴파일러인 High Performance Compiler for Java를 발표하면서 AOT 컴파일러 개념이 등장했다. 이후 1998년에는 GNU 프로젝트에 포함된 GCC의 자바용 컴파일러 GCJ가 등장했고, 이는 OpenJDK의 배포판들에 기본적으로 포함되기도 했다. 그러나 이들은 자바 생태계에서 별다른 주목을 받지 못했다. 자바는 ‘한 번 작성하면 어디서나 실행된다’는 플랫폼 독립성을 핵심 가치로 내세웠기 때문이다.

이러한 배경 속에서 2013년 안드로이드가 AOT 컴파일러 기반의 런타임인 ART를 도입하면서 AOT 컴파일러가 다시 주목을 받기 시작했다. ART는 기존 JIT 방식과 달리 앱 설치 시점에 네이티브 코드로 변환하여 실행 중 부담을 덜고, 실행 지연이나 배터리 사용량을 줄이는 효과를 가져왔다. 이로 인해 안드로이드는 JIT 대신 AOT 방식을 채택하게 되었다.

AOT 컴파일은 JIT에 비해 몇 가지 장점을 가진다. 첫째, 프로그램 실행 전에 전체 코드를 미리 컴파일함으로써 런타임 시 JIT 컴파일에 소비되는 시간을 제거할 수 있다. 둘째, 메서드 호출 시 마다 반복되는 JIT 작업을 피할 수 있어 런타임 오버헤드를 줄일 수 있다. 셋째, 자주 사용되는 코드의 경우 JIT보다 더 정교한 최적화가 가능하며, 이는 LLVM 기반의 Clang이나 OpenJ9의 Testarossa 같은 고급 JIT 컴파일러에서도 실현되는 방식이다. 넷째, AOT 방식은 JIT에 비해 시스템 간 코드 공유나 캐시 재활용에도 유리하다. 예를 들어 공통 라이브러리 코드를 미리 AOT 컴파일한 후 여러 자바 프로세스가 공유해 사용할 수 있다.

하지만 AOT 컴파일은 단점도 명확하다. 가장 큰 단점은 JIT처럼 프로그램 실행 중 성능 모니터링 정보를 수집하지 못해 실행 특성에 따른 최적화가 불가능하다는 점이다. JIT 컴파일러는 실제 실행 데이터를 바탕으로 가장 자주 호출되는 메서드나 반복문, 분기 조건 등을 최적화할 수 있지만, AOT는 이러한 실행 시간 정보를 전혀 알 수 없다. 또한 AOT 방식은 프로그램 설치 또는 배포 시점에 최적화가 완료되어야 하므로 최적화 품질에 있어 제한이 생긴다. 예를 들어 특정 라이브러리 메서드가 실제 실행 시점에 어떤 방식으로 호출될지 예측하기 어렵기 때문에, 링크 타임 최적화(link-time optimization)도 제한적이다. 이로 인해 AOT 방식은 전체 프로그램의 성능을 근본적으로 개선하기보다는 예측 가능한 특정 시나리오에 최적화된 방식이라 할 수 있다.

또한 AOT 방식은 컴파일 시간과 설치 시간, 코드 크기 측면에서 불리할 수 있다. 프로그램을 AOT 컴파일하면 설치 시 시간이 더 오래 걸리고, 생성된 네이티브 코드의 용량도 커질 수 있다. 특히 플랫폼 독립성을 중시하는 자바 생태계에서 네이티브 코드가 플랫폼마다 다르게 생성된다는 점은 여전히 해결되지 않은 문제이다. 이는 애플리케이션을 다양한 환경에서 배포하려는 경우 부담이 될 수 있다.

JIT 컴파일은 AOT보다 훨씬 많은 런타임 최적화 기법을 활용할 수 있다. 예를 들어 조건 분기 예측, 레지스터 재할당, 힙 객체 생존 기간 추적 등을 통해 다양한 수준의 성능 개선이 가능하다. 자바의 JIT 컴파일러는 이처럼 실행 시점에 프로그램을 분석하고 그에 따라 최적화 전략을 조정하는 능력을 갖춘 반면, AOT는 고정된 최적화 결과만을 제공한다는 한계를 가진다. 자바의 메서드 인라이닝(inlining) 역시 대부분 JIT 환경에서만 제대로 작동하며, AOT에서는 기본적으로 어렵거나 불가능한 경우도 많다.

따라서 AOT 컴파일러는 JIT의 대체재가 아니라 보완재로 보는 것이 더 적절하다. 실행 환경이 예측 가능한 모바일 환경이나 설치형 환경에서는 AOT가 적합할 수 있지만, 다양한 런타임 시나리오와 성능 튜닝이 필요한 서버 환경이나 데스크톱 애플리케이션에는 여전히 JIT이 더 유리하다. 최근에는 이러한 특성을 활용해 JIT과 AOT를 병행하는 하이브리드 방식도 연구되고 있으며, 자바 백엔드 컴파일 기술은 계속해서 진화하고 있다.

### 실전: jaotc의 AOT 컴파일

JDK 9는 클래스 파일과 모듈의 AOT 컴파일을 지원하는 도구인 jaotc를 도입했다. jaotc는 프로그램 구동과 예열 시간을 줄여 최대 성능을 빠르게 끌어낸다. 하지만 이 기능은 특정 하드웨어에서 특정 가상 머신 매개 변수를 설정해 사용해야 하며, 제한이 너무 많기 때문에 잘 이해하고 활용하는 개발자가 많지 않았다. 실제로 관련 코드와 도구는 너무 적어, 이후 그럴VM 쪽으로 넘기고 JDK 16부터는 표준 JDK에서 제거되었다. 따라서 이번 절의 예시는 LTS 버전인 OpenJDK 11로 진행한다.

그럴VM에 포함된 그럴 컴파일러와 jaotc는 목적과 사용면에서 차이가 있다. jaotc는 코드 일부를 네이티브로 미리 컴파일해 두지만, 여전히 자바 가상 머신에서 구동되는 것을 가정한다. 반면 그럴 컴파일러의 AOT는 가상 머신 없이 실행되는 네이티브 이미지를 생성한다.

OpenJDK 11 환경에서 jaotc를 사용하는 실습은 java.base 모듈을 사전 컴파일하여 자바 기본 클래스의 실행 효율을 높이는 것으로 시작한다. 간단한 예제로는 HelloWorld 코드를 준비한다. 먼저 HelloWorld.java를 작성한 후 컴파일하고, jaotc를 통해 libHelloWorld.so라는 네이티브 라이브러리를 생성한다. 이 라이브러리는 nm 명령어를 통해 생성자와 main() 메서드가 포함되었는지를 확인할 수 있으며, ldd 명령어로 정적 링크 여부도 확인 가능하다.

이렇게 생성된 라이브러리를 실행할 때는 -XX:+UnlockExperimentalVMOptions -XX:AOTLibrary 옵션을 통해 VM에게 AOT 라이브러리를 사용하도록 알린다. HelloWorld를 실행하면 main() 메서드는 사전 컴파일된 버전으로 동작하게 된다. 이를 -XX:+PrintAOT 옵션을 추가하여 실행하면, 실제 어떤 메서드가 aot 라이브러리로부터 로드되었는지 확인할 수 있다.

libjava.base.so와 같은 AOT 라이브러리를 명시적으로 로딩하지 않으면, 기본적으로 HelloWorld의 main 메서드만 사전 컴파일된 버전을 사용한다. 하지만 libjava.base.so를 추가로 로드하면, java.lang.Object를 포함한 java.base 모듈 전체의 메서드들도 사전 컴파일된 버전을 사용하는 것을 확인할 수 있다. 실제로 -XX:+PrintAOT 출력에서 java.lang.Object.()이나 finalize() 같은 메서드도 aot된 상태로 출력된다.

이번 실습의 핵심은 jaotc를 통해 java.base 모듈 전체를 AOT 컴파일하고 이를 바탕으로 자바 프로그램의 부트스트랩 성능을 개선하는 것이었다. 하지만 jaotc는 내부적으로 그럴 컴파일러를 기초로 구현되었기 때문에 ZGC와 같이 가비지 컬렉터 선택이 제한된다. jaotc는 PS+PS(Parallel Scavenge + Serial Old)와 G1만 지원한다. 따라서 특정 GC를 사용하는 환경에서는 사용할 수 없다.

java.base 전체를 컴파일하는 명령은 다음과 같다. --compile-commands 옵션으로 컴파일할 메서드를 지정한 텍스트 파일을 사용하고, --output으로 출력 대상 라이브러리를 지정한다. 총 6272개의 클래스 중 56501개의 메서드 중 50070개가 컴파일에 성공했고, 실패한 수는 0개였다. 총 소요 시간은 약 312초였으며, 생성된 라이브러리는 353MB에 달했다.

이렇게 생성된 libjava.base-coop.so를 HelloWorld 실행 시에 함께 사용하면, java.base 모듈의 많은 메서드들이 사전 컴파일된 버전으로 실행된다. 실제로 -XX:+PrintAOT 출력에서 java.lang.Object 이외에도 다양한 java.base 모듈의 메서드들이 aot 라이브러리에서 로딩되는 것을 확인할 수 있다.