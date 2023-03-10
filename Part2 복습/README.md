## 김다영

### ch.4
### 멀티 프로그래밍 대신 멀티 스레드를 사용하는 이유?
> 프로세스 생성은 많은 시간과 자원이 필요하고, 프로세스 간 데이터를 공유하기 위한 IPC 기법은 구현, 관리가 힘듭니다.
> 스레드는 자동으로 그들이 속한 프로세스의 자원과 메모리를 공유하고, 생성 속도도 더 빠르기 때문에 프로세스보다 더 효율적입니다.

### 동시성과 병렬성의 차이는?

> 동시성은 싱글 프로세서 시스템에서 프로세서가 여러 개의 스레드를 번갈아가며 수행함으로써 동시에 실행되는 것처럼 보이게 하는 방식이고,
> 병렬성은 멀티코어 시스템에서 여러 개의 코어가 각 스레드를 동시에 수행하는 방식입니다.

### ch.5
### CPU 스케줄링 결정이 발생하는 상황 4가지
> 1. 한 프로세스가 running -> waiting 상태로 전환될 때
> 2. 프로세스가 running -> ready 상태로 전환될 때
> 3. 프로세스가 waiting -> ready 상태로 전환될 때
> 4. 프로세스가 종료할 때

### 스케줄링 기준은 어떻게 나요?
> CPU 이용률과 처리량을 최대화 하고, 총 처리시간, 대기 시간, 응답 시간을 최소화하도록 스케줄링 해야합니다.

### 선점형과 비선점형 스케줄링의 차이점은?
> 선점 스케줄링은 운영체제가  CPU를 강제로 다른 프로세스에게 할당하는 방식이고,
> 비선점 스케줄링은 프로세스가 스스로 다음 프로세스에게 자리를 넘겨주는 방식입니다.

### ch.6
### 데이터가 다수의 프로세스에 의해 공유될 때의 문제점은?
> 프로세스가 동시 또는 병렬로 실행될 때, 여러 프로세스가 공유하는 데이터의 무결성에 문제가 발생하는 race condition이 발생할 수 있습니다.

### 스핀락의 단점과 이를 해결하기 위한 방안은?
> 스핀락은 락을 획득할 때까지 바쁜대기를 하는데, 이는 CPU 사이클을 낭비하게 됩니다.
> 이에 대한 해결 방법으로 Mutex lock과 Semaphore가 있는데, 락이 이미 사용중일 때 대기하는 프로세스들을 대기 큐에 넣어두고, 락이 반환되었을 때 재실행을 하는 방식을 사용합니다.

### 세마포어 대신 Spinlock을 사용하는 경우는 언제인가요?
> 뮤텍스 락에선 스레드가 큐에 들어가고, 나중에 깨워주는 과정에서 context switch가 발생한다. context switch는 오버헤드가 발생하는 방법입니다.
> 따라서 context switch가 발생하는 것보다 critical section에서 작업이 더 빨리 끝난다면, 스핀락 방식을 사용하는 것이 더 효율적입니다.
> 다만, 이는 멀티 코어 환경일때만 해당합니다. 멀티 코어에서 대기하는 스레드와 락을 취득한 스레드가 서로 다른 코어에서 실행중이라면, context switch가 발생하지 않지만, 싱글 코어 환경에선 context switch가 무조건 발생하기 때문에 전혀 이점이 없습니다.

### 이진 세마포어와 뮤텍스 차이점
> 뮤텍스는 락을 걸은 스레드만이 임계영역을 나갈 때 락을 해제할 수 있다.
> 하지만 세마포어는 락을 걸지 않은 스레드도 signal을 사용해 락을 해제할 수 있다.

## 조민서
# Chapter 4. 스레드와 병행성

## 스레드

### 스레드와 프로세스의 차이점은 무엇인가요.

프로세스는 메모리 상에서 실행중인 프로그램을 말하며, 스레드는 이 프로세스 안에서 실행되는 흐름 단위를 말한다.

프로세스는 최소 하나의 스레드를 보유하고 있으며, 각각 별도의 주소공간을 독립적으로 할당 받는다.(code, heap, stack)
스레드는 이중에 stack만 따로 할당받고 나머지 영역은 스레드끼리 서로 공유한다.

### **프로세스의 문제점은 무엇인가요?**

- 프로세스 생성에 큰 오버헤드가 있다. ( 프로세스를 생성할때 많은 시간이 소요됨0
- 프로세스 컨텍스트 스위칭의 비효율성, 오버헤드가큼
- 프로세스 사이에 통신이 어렵다는점 (IPC사용해야함)

### **스레드의 출현 목적은 무엇인가요?**

- 프로세스보다 크기가 작은 실행 단위 필요
- 프로세스의 생성 및 소멸에 따른 오버헤드 감소
- 빠른 컨텍스트 스위칭
- 프로세스들의 통신 시간, 방법 어려움 해소

### 멀티 스레드 이점에 대해 설명해 주세요.

프로세스는 IPC기법을 통해 자원을 공유 할 수 있습니다. 하지만 스레드는 자신이 속한 프로세스의 힙, 데이터, 파일 자원 등을 공유하므로 스레드끼리 문맥교환 하는것이 경제성이 좋습니다.

다중 처리기 구조에서 단일스레드 인 경우 처리기가 아무리 많더라도 한 처리기에서만 실행 되지만, 멀티 스레드 인 경우 여러 처리기에서 실행 할 수 있습니다. 그래서 사용자에게 응답이 빠릅니다.

### 데이터 병렬 실행, 태스크 병렬 실행

![image](https://user-images.githubusercontent.com/64322765/219954794-19019a45-415b-4e60-95c9-4cce801595fc.png)

### **스레드 풀은 무엇인가요?**

스레드를 요청할 때마다 새로운 스레드를 생성, 수행, 삭제를 통해 성능이 저하되는 것을 방지하기 위해 미리 스레드 풀에 여러개 스레드를 만들어두고, 요청이 오면 스레드 풀에서 할당해주는 방법

### 스레드 로컬 저장장치(**TLS (Thread local storage)) = 레드풀** 에 대해 설명해 주세요.

스레드 로컬 저장장치는 각 스레드가 자신의 스레드 로컬 변수(Thread Local Variables, TLV)를 가지는 메모리 공간을 말합니다. 이것은 멀티 스레드 환경에서 각 스레드가 자신의 데이터를 안전하게 저장하고 공유하지 않아도 되도록 하는 기술입니다.

스레드 로컬 저장소는 전역 변수와 비슷하지만, 전역 변수는 모든 스레드에서 공유되는 반면, 스레드 로컬 변수는 각 스레드에서만 사용할 수 있습니다. 이는 각 스레드에서 자신의 데이터를 안전하게 보호하면서, 다른 스레드에서는 그 데이터에 직접 접근할 수 없도록 하는 장점이 있습니다.

TLS는 C++, Java, Python 등 다양한 프로그래밍 언어에서 지원하고 있으며, 스레드 풀, 웹 서버, 클라우드 서비스 등의 다양한 분야에서 사용되고 있습니다.

### 스레드 로컬저장 장치 대신에 그냥 메모리구조에서 스택영역 쓰면 각 스레드마다 각자의 독립된 데이터를 가지는거 아니야?

네, 맞습니다. 스레드별로 각자 독립적인 스택 영역이 존재하기 때문에, 스택을 이용하여 각 스레드마다 독립된 데이터를 저장할 수 있습니다.

스레드 로컬 저장소(TLS)는 스레드마다 독립된 데이터를 저장하기 위한 또 다른 방법 중 하나로, 전역 변수와 달리 스레드 간 데이터의 공유나 동기화 문제를 해결할 수 있습니다. 

TLS를 사용하면 여러 스레드가 동시에 실행되는 멀티 스레드 환경에서도 각 스레드마다 독립적인 데이터를 안전하게 저장할 수 있습니다.

따라서, TLS를 사용하지 않고 스택 영역을 이용하여 각 스레드마다 독립된 데이터를 저장하는 것도 가능합니다. 하지만 이 경우 스택 영역의 크기가 고정되어 있기 때문에, 저장할 수 있는 데이터의 크기에 제한이 있을 수 있습니다. 또한, 스택 영역은 함수 호출 시 생성되는 임시 데이터의 저장에 사용되기 때문에, TLS보다는 약간 느릴 수 있습니다.

### 커널 모드로 전환되면 함수 실행을 커널 수준 스레드가 하는게 아니야?

트랩이 발생하면 실행 중인 스레드는 커널 모드로 전환되고, 해당 스레드가 실행 중이던 사용자 모드 함수는 커널 함수로 대체되고, 트랩 시스템 콜을 호출하면, 트랩은 커널 스레드에 의해 처리됩니다.

### 사용자 수준 스레드가 모드비트를 0으로 바꾸고 시스템콜을 호출하면 커널 수준 스레드가 시스템콜을 처리한다는거지?

네, 사용자 수준 스레드가 모드비트를 0으로 바꾸고 시스템 콜을 호출하면 커널 수준 스레드가 시스템 콜을 처리합니다.

사용자 수준 스레드는 운영체제에서 직접 관리되지 않기 때문에 시스템 콜을 직접 처리할 수 없습니다. 대신 사용자 수준 스레드는 시스템 콜을 호출하면 커널 모드로 전환되어 커널 수준 스레드가 시스템 콜을 처리합니다. 이 때 커널 수준 스레드는 사용자 수준 스레드의 컨텍스트를 저장하고, 시스템 콜이 완료되면 결과를 반환하고 사용자 수준 스레드로 다시 전환됩니다.

따라서, 사용자 수준 스레드는 시스템 콜을 호출하여 커널 수준 스레드가 시스템 콜을 처리하도록 하는 것이 일반적인 방법입니다.

## 멀티 스레드 모델

### 다대일 모델

![image](https://user-images.githubusercontent.com/64322765/219954807-05c1d7f7-ed84-4f2e-9502-e28101c37e26.png)


하나의 스레드가 봉쇄되면 전체 프로세스가 봉쇄되기 때문에 시스템을 껐다켜야 합니다.

그리거 개발자가 원하는 만큼의 사용자 스레드를 생성하지만, 한번에 하나의 커널 스레드만 스케줄 할 수 있기 때문에 병렬 실행을 할 수 없습니다.

### 일대일 모델

![image](https://user-images.githubusercontent.com/64322765/219954818-93f10b81-26c8-47c9-9b54-39d985a2f9db.png)

하나의 스레드가 봉쇄되더라도 다른 스레드가 실행 될 수 있기 때문에 다대일 모델보다 더 많은 병렬 실행을 제공합니다. 

하지만 개발자가 너무 많은 스레드를 생성하지 않도록 주의 해야합니다. 사용자 스레드를 만들면 커널 스레드도 만들어야 하기 때문에 커널 스레드가 시스템 성능에 부담을 줄 수 있습니다.

### 다 대 다 모델

![image](https://user-images.githubusercontent.com/64322765/219954837-ef00739e-9860-4d4c-a5a3-2d28644ee1e2.png)
사용자 스레드가 동기화되었을 때 커널 스레드는 다른 사용자 스레드를 스케줄 할 수 있습니다.

하지만 실제로는 구현하기가 어렵고 기술이 발전하면서 각 처리기의 코어 수가 증가함으로써 커널 스레드 수를 제한하는 것의 중요성이 줄어들었습니다.

그래서 대부분의 운영체제는 일대일 모델을 사용합니다.

### 두 수준 모델

![image](https://user-images.githubusercontent.com/64322765/219955408-bc428672-8e85-441a-ba23-0d96bce493dc.png)

다대다 모델 + 일대일 모델

### 스레드와 관련된 문제들 (Threading Issues)

### 우리는 **다중 스레드 프로그램에서 fork()와 exec()의 의미**에 대해서 생각해 보아야 한다.

1. 만일 한 프로그램의 스레드가 fork()를 호출하면 새로운 프로세스는 모든 스레드를 복제해야 하는가 아니면 한 개의 스레드만 가지는 프로세스여야 하는가?
    - UNIX 시스템은 fork() API로써 이 둘의 기능을 다 지원한다.
2. 보통 어떤 스레드가 exec() 시스템 콜을 부르면 exec()의 매개변수로 지정된 프로그램이 모든 스레드를 포함한 전체 프로세스를 대체시킨다.

**우리는 운영체제에서 지원해주는 기능에 따라 적절히 fork()와 exec()를 사용해야 한다.**

- 문제
    
    ### 멀티코어 멀티스레드 시스템에서 fork 시스템 콜을 호출할 때 프로세스는 모든 스레드를 복사하나요? 아니면 fork를 호출하는 스레드 하나만 복사하나요?
    
    fork를 하는 이유가 2개 있습니다.
    
    1. fork()만 하는 경우
    2. fork()를 하고 exec()를 호출하는 경우
    
    fork만 하는 경우에는 모든 스레드를 복사 해주고
    
    fork하고 exec하는 경우에는 fork하자 마자 exec하는 프로그램이 모든 것을 대체하기에 모든 스레드를 복사해 줄 필요가 없습니다.
    

### **신호처리는 UNIX에서 프로세스에 어떤 이벤트가 일어났음을 알려주기 위해 사용된다.**

신호는 비동기적, 동기적으로 발생할 수 있는데 이와 상관없이 모든 신호는 다음과 같은 형태로 전달되어야 한다.

1. 신호는 특정 이벤트가 일어나야 생성된다.
2. 생성된 신호가 프로세스에 전달된다.
3. 신호가 전달되면 반드시 처리되어야 한다.

모든 신호는 둘 중 하나의 처리기에 의해 처리된다.

1. 디폴트 신호 처리기
2. 사용자 정의 신호 처리기

프로세스가 여러 스레드를 가지고 있는 경우 동기식 신호는 그 신호를 야기한 스레드에 전달되어야 하고 다른 스레드에 전달되면 안 된다. 반면에 비동기 신호의 경우에는 그 프로세스 내 모든 스레드에 전달되어야 한다.

### **스레드 취소(thread cancellation)는 스레드가 끝나기 전에 그것을 강제 종료시키는 작업을 일컫는다.**

> (ex. 여러 스레드가 데이터베이스를 병렬로 검색하고 있다가 그 중 한 스레드가 결과를 찾았다면 나머지 스레드는 취소 되어야 하는 경우, …)
> 

이 처럼 취소되어야 할 스레드를 **목적 스레드(target thread)**라고 부른다. 목적 스레드는 다음과 같은 두 가지 방식으로 취소할 수 있다.

1. **비동기식 취소(asynchronous cancellation):** 한 스레드가 즉시 목적 스레드를 강제 종료시킨다.
2. **지연 취소(deferred cancellation):** 목적 스레드가 주기적으로 자신이 강제 종료 되어야 할지를 점검한다. (질서 정연하게 강제 종료 가능)

**스레드 취소를 어렵게 만드는 것은 취소 스레드들에 할당된 자원의 문제가 가장 크기 때문에 이를 잘 고려해서 스레드 취소를 해야 한다.**

비동기식 취소의 경우 OS가 모든 시스템 자원을 다 회수하지 못하는 경우가 있다. 그래서 보통 기본 스레드 취소 유형은 지연취소 이다. 즉, 스레드가 취소 점에 도달한 경우에만 취소가 발생한다.

Pthreads에서는 pthread_cancel() 함수를 사용하여 스레드를 취소할 수 있다. pthread_cancel()을 호출하면 대상 스레드를 취소하라는 요청만 표시된다. 그러나 실제 취소는 요청을 처리하기 위해 대상 스레드가 설정되는 방식에 달려 있다. 대상 스레드가 최종적으로 취소되면 취소 스레드의 pthread_join() 호출이 반환된다.

Pthreads는 아래와 같이 3가지 취소 모드를 지원한다.
![image](https://user-images.githubusercontent.com/64322765/219955426-131da270-5b42-48cf-ac37-5afb3da7af2a.png)

Pthreads는 스레드가 취소될 때 **정리 핸들러(clean handler)**라고 하는 스레드가 획득한 모든 자원을 해제할 수 있는 함수를 제공한다.

Java의 스레드 취소는 Pthread의 지연 취소와 유사한 정책을 사용한다. 스레드를 취소하려면 thread객체.interrupt() 메소드를 호출하여 대상 스레드의 인터럽트 상태를 true로 설정하면 된다.

### 스레드-로컬 저장장치 (Thread-Local Storage)

한 프로세스에 속한 스레드들은 그 프로세스의 데이터를 스레드간에 모두 공유한다. 하지만 상황에 따라서는 각 스레드가 **자기만 액세스할 수 있는 데이터를 가져야 할 필요도 있다. 그러한 데이터를 스레드-로컬 저장장치(thread-local storage, TLS)**라고 부른다.

> ex) 트랜잭션 처리 시스템에서 각 트랜잭션을 독립된 스레드가 처리해 준다고 가정할 때 스레드마다 고유한 식별자를 연관시키기 위해서는 TLS가 있어야만 한다.
> 

### 스케줄러 액티베이션 (Scheduler Activations)

다 대 다 모델 또는 두 수준 모델을 구현하는 많은 시스템은 사용자와 커널 스레드 사이에 중간 자료구조를 둔다. 이 자료구조는 통상 **경량 프로세스 또는 LWP**라고 불리며 아래의 그림과 같다.

**사용자 스레드 라이브러리에 LWP 방식은 어플리케이션이 사용자 스레드를 수행하기 위하여 스케줄 할 가상 처리기(virtual processor)처럼 보인다.**

![image](https://user-images.githubusercontent.com/64322765/219955421-d25fc0c8-1599-443c-abb3-a25e9f3a6350.png)

각 LWP는 하나의 커널 스레드에 부속되어 있으며 Processor가 스케줄 하는 대상은 바로 이 커널 스레드이다.

**사용자 스레드 라이브러리와 커널 스레드 간의 통신 방법의 하나는 스케줄러 액티베이션이라고 알려진 방법이다.** 커널은 어플리케이션에 LWP의 집합을 제공하고 어플리케이션은 사용자 스레드를 가용한 가상 처리기(LWP)로 스케줄 한다. 커널은 어플리케이션에 특정 이벤트에 대해 알려줘야 한다. 이 프로시저를 **`upcall`**이라고 부른다.

Upcall은 스레드 라이브러리의 upcall 처리기에 의해 처리되고, upcall 처리기는 가상 처리기상에서 실행되어야 한다.

# Chapter 6. 프로세스 동기화

### 경쟁 상황에 대해 설명해주세요.

여러 프로세스가 공유 자원에 동시에 접근할 때 실행 순서에 따라 실행 결과가 달라질 수 있는 상황입니다.

### 임계 구역에 대해 설명해주세요.

임계영역은 공유 데이터가 조작될 수 있으며, 경쟁 상황이 발생할 수 있는 코드영역입니다. 임계영역 문제는 데이터를 협력적으로 공유하기 위하여 각 프로세스의 활동을 동기화하는 프로토콜을 설계해야합니다.

### 임계 영역 문제(경쟁상항)를 해결하기 위해서는 아래의 3가지 조건을 충족해야 합니다.

상호배제(Mutual exclusion) : 특정 프로세스가 임계영역 내에서 실행중일 때, 다른 프로세스는 임계영역에 들어갈 수 없다.

진행(progress) : 임계영역를 진행 중인 프로세스가 없을 경우에, 임계영역에 진입을 요구할 경우 유한한 시간내에 진입해야한다. 정당한 진입 프로세스를 막으면 안된다.

한정된 대기(bounded waiting) : 임계영역에 대해 진입요청을 했을 경우, 무한히 기다리지 않고 유한한 시간내에 진입해야한다.

### 피터슨 해결안이 현대 컴퓨터에서 사용하기 힘든 이유는 무엇입니까?

피터슨 해결안은 하나의 프로세스당 true또는 false의 값을 가질 수 있으므로 시간복잡도가 O(2^프로세스)이다. 배열의 크기가 2 즉 프로세스가 2개면 상관없지만 배열의 크기가 64(프로세스 64개)만 되더라도 O(2^64)로 시간 복잡도가 기하급수적으로 증가한다. 또한 2개가 아니라 여러 개의 프로세스를 소프트웨어적으로 병렬 프로그래밍하기는 쉽지 않다. 따라서 현대 컴퓨터에서 사용하기 힘들다.

### 임계구역 문제를 해결하기 위해 하드웨어가 지원하는 세 가지 명령어가 무엇인가요? 그리고 세 가지를 설명해 주세요.

메모리 장벽, Compare And Swap(CAS), 원자적 변수입니다.

메모리 장벽은 다중프로세서에서 한 프로세스가 메모리를 변경하고 다른 프로세서의 프로세스 메모리가 바로 변경되지 않아 데이터 일관성 문제(가시성 문제)가 발생하는데 로직 사이에 메모리 장벽을 두어 다른 프로세서에서 실행 중인 프로세스에 변경된 메모리 값이 보이는 것을 보장해 줍니다.

![image](https://user-images.githubusercontent.com/64322765/219955325-f9cdf7cc-7b6f-4a20-ae00-58620552c13d.png)

Compare And Swap(CAS)은 멀티 쓰레드 환경, 멀티 코어 환경에서 각 CPU는 메인 메모리에서 변수값을 참조하는게 아닌, 각 CPU의 캐시 영역에서 메모리를 값을 참조하게 된다. (*[그림2] CPU 캐시 메모리*
 참고) 이때, 메인 메모리에 저장된 값과 CPU 캐시에 저장된 값이 다른 경우가 있다. (이를 가시성 문제라고 한다.) 그래서 사용되는 것이 CAS 알고리즘이다. **현재 쓰레드에 저장된 값과 메인 메모리에 저장된 값을 비교하여 일치하는 경우 새로운 값으로 교체하고, 일치 하지 않는 다면 실패하고 재시도를 한다.**
이렇게 처리하면 CPU캐시에서 잘못된 값을 참조하는 가시성 문제를 해결하고 원자적 연산을 보장합니다.

원자적 변수는 단일변수 카운트가 증가하는 동안 데이터 경쟁이 있는 상황에 상호배제를 보장하고, CAS연산을 사용하여 구현됩니다.

### 뮤텍스 락에 대해 설명해 주세요.

프로세스 혹은 스레드 간의 통신 시에 shared memory를 쓰는 경우 하나의 자원에 두 개 이상의 프로세스 혹은 스레드가 접근하는 경우에 문제가 발생한다.

뮤텍스 : 상호배제라고도 하며, 임계영역을 가진 스레드의 Running Time이 서로 겹치지 않도록 각각 단독으로 실행하게 하는 기술이다. 뮤텍스는 상태가 0, 1 두개 뿐인 이진 세마포어. synchronized 또는 lock을 통해 해결한다.
하지만 한개의 스레드가 CPU를 점유하면 다른 스레드는 한개의 스레드가 끝날때까지 기다려야하는 바쁜대기 문제가 있습니다.

### 세마포어란 무엇입니까?
세머포어 정수 변수는, 공유 자원의 개수를 나타냅니다. 예를 들어  시골마을에 농부들은 마을회관에서 삽을 가져가 쓰는데 뮤택스 락인 경우에는 삽 하나를 농부들이 돌아가면서 쓰는 상황이고, 세마포어는 삽 여러개를 농부들이 돌아가면서 씁니다. 여기서 차이점은 농부들이 쓸 수 있는 **삽의 개수**입니다.

세마포어는 이 공유된 자원의 데이터를 여러 스레드에서 동시에 접근 하는것을 동기화 시킵니다. 그리고 뮤택스락 의 바쁜대기를 해결했습니다.

### 세마포는 스핀락의 바쁜대기 문제를 어떻게 해결했나요?
스핀락에서, 임계영역에 진입해야하는 프로세스는 진입 코드를 계속 반복 실행해야 하며, 바쁜대기를 하며 CPU시간을 낭비했었다.  

세마포는 각 세마포마다 세마포 대기 리스트를 만들어 임계영역 자리가 없는데 우선순위가 더 높은 스레드가 임계영역 진입을 하려고하면 세마포 대기 리스트에 현재 임계영역을 사용중인 세마포를 넣어놓고 우선순위가 더 높은 스레드를 임계영역에 넣은다 그리고 다시 임계영역 자리가 날 때 다시 세마포 대기 리스트에 있는 스레드를 깨우는 방식을 사용하여 바쁜대기 문제를 해결한다.

### 모니터가 무엇인지 설명해주세요.
모니터는 프로그래밍언어에 지원하는 공유 자원에 대한 상호배제를 추상화한 개념입니다.
상호배제는 여러 프로세스나 스레드가 동시에 공유 자원에 접근하면 문제가 발생할 수 있는 상황에서, 상호배타적으로 자원을 사용하도록 제어하는 것을 의미합니다. 모니터는 이러한 상호배제를 추상화한 도구로, 공유 자원에 대한 접근을 모니터 객체를 사용하여 동기화하고, 상호배제를 구현합니다.
또한 모니터는 조건변수를 통해, 공유 자원에 대한 조건부 접근을 구현할 수 있습니다. 예를 들어, 공유 자원이 특정 조건을 만족해야 접근이 가능한 경우, 조건변수를 사용하여 자원에 대한 조건이 충족될 때까지 대기(wait)하고, 조건이 충족되면 신호(signal)를 보내어 대기중인 프로세스나 스레드가 자원에 접근할 수 있도록 제어합니다.
따라서 모니터는 상호배제와 조건부 접근을 추상화한 개념으로, 여러 프로그래밍 언어에서 동기화를 구현하는데 사용됩니다.

### 다중코어시스템에서 스핀락을 어떤 경우에 사용할 수 있나요?
대기중인 스레드가 실행상태가 되기 위해서는 두번의 문맥교환을 합니다.
 1.  실행중인 스레드를 대기상태로 옮기는 문맥교환
 2.  대기중인 스레드를 실행상태로 옮기는 문맥교환

이 2개의 문맥교환 시간보다 락 유지시간(임계영역에서 유지 시간)이 짧으면 다중처리시스템에서 스핀락을 사용하기도 합니다. 하지만 단일처리 시스템인 경우 스레드가 무한대기가 일어 날 수 있으므로 스핀락을 사용해서는 안됩니다.

### 세마포어와 모니터의 차이
세마포어에 비해서 모니터 쪽이 공유자원에 접근할 수 있는 키의 획득과 해제를 모두 처리해서 간단하다. 세마포어는 직접 키해제와 공유자원 접근 처리를 해주어야한다


### 라이브니스에 대해 설명해 주세요.
라이브니스는 프로세스가 실행 수명주기 동안 진행되는것을 보장하기 위해 시스템이 충족해야하는 일련의 속성입니다.
예를들어 바쁜대기는 라이즈니스를 실패한 예시 입니다. 바쁜대기가 일어나면 대기중인 프로세스들은 특정 이벤트(signal())를 무한히 기다리는 데드락이 발생할 수 있습니다. 
또한 우선순위가 낮은 프로세스가 바쁜대기를하면 우선순위가 높은 프로세스는 낮은 프로세스의 바쁜 대기때문에 실행하지 못하는 우선순위 역전 상태가 됩니다.


## 조예은
