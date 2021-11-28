---

layout: post
title: 프로그램의 동작을 스레드 스케줄러에 기대지 말자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



여러 스레드가 실행 중이면 운영체제의 스레드 스케줄러가 어떤 스레드를 얼마나 오래 실행할지 정한다. 

**정상적인 운영체제라면 이 작업을 공정하게 수행하지만 구체적인 스케줄링 정책은 운영체제마다 다를 수 있다.**

따라서 잘 작성된 프로그램이라면 이 정책에 좌지우지돼서는 안 된다.

**정확성이나 성능이 스레드 스케줄러에 따라 달라지는 프로그램이람련 다른 플랫폼에 이식하기 어렵다.**

견고하고 빠릿하고 이식성 좋은 프로그램을 작성하는 가장 좋은 방법은 실행 가능한 스레드의 평균적인 수를 프로세서 수보다 지나치게 많아지지 않도록 하는 것이다.

그래야 스레드 스케줄러가 고민할 거리가 줄어든다.

실행 준비가 된 스레드들은 맡은 작업을 완료할 때까지 계속 실행되도록 만들자.

이런 프로그램이라면 스레드 스케줄링 정책이 아주 상이한 시스템에서도 동작이 크게 달라지지 않는다.

여기서 실행 가능한 스레드의 수와 전체 스레드 수는 구분해야 한다.

전체 스레드 수는 훨씬 많은 수 있고, 대기 중인 스레드는 실행 가능하지 않다.

실행 가능한 스레드 수를 적게 유지하는 주요 기법은 각 스레드가 무언가 유용한 작업을 완료한 후에는 다음 일거리가 생길 때까지 대기하도록 하는 것이다.

**스레드는 당장 처리해야 할 작업이 없다면 실행돼서는 안 된다.**

실행자 프레임워크를(Executor Framework)를 예로 들면, 스레드 풀 크기를 적절히 설정하고 작업은 짧게 유지하면 된다.

단, 너무 짧으면 작업을 분배하는 부담이 오히려 성능을 떨어뜨릴 수도 있다.

# 바쁜 대기 상태

스레드는 절대 바쁜 대기(busy waiting) 상태가 되면 안 된다.

공유 객체의 상태가 바뀔 때까지 쉬지 않고 검사해서는 안 된다는 뜻이다.

바쁜 대기는 스레드 스케줄러의 변덕에 취약할 뿐 아니라, 프로세서에 큰 부담을 주어 다른 유용한 작업이 실행될 기회를 박탈한다.

우리가 피해야 할 극단적인 예로, CountDownLatch를 삐딱하게 구현한 코드를 살펴보자.

**끔찍한 CountDownLatch 구현 - 바쁜 대기 버전**

```java
public class SlowCountDownLatch {
    
    private int count;

    public SlowCountDownLatch(int count) {
        if (count < 0)
            throw new IllegalArgumentException(count + " < 0");
        this.count = count;
    }
    
    public void await() {
        while(true) {
            synchronized (this) {
                if (count == 0)
                    return;
            }
        }
    }
    
    public synchronized void countDown() {
        if (count != 0)
            count--;
    }
}
```



**실행코드**

```java
public class CountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        int nThreads = 1000;
        
        //SlowCountDownlatch 이용
        ExecutorService executorService = Executors.newFixedThreadPool(nThreads);
        SlowCountDownLatch slowCountDownLatch = new SlowCountDownLatch(nThreads);

        long start1 = System.currentTimeMillis();
        for (int i = 0; i < nThreads; i++) {
            executorService.submit(() -> slowCountDownLatch.countDown());
        }

        slowCountDownLatch.await();
        executorService.shutdown();
        long end1 = System.currentTimeMillis();
        System.out.printf("slowCountDownLatch : %s ", end1 - start1);

        //CountDownlatch 이용
        long start2 = System.currentTimeMillis();
        ExecutorService executorService2 = Executors.newFixedThreadPool(nThreads);
        CountDownLatch countDownLatch = new CountDownLatch(nThreads);

        for (int i = 0; i < nThreads; i++) {
            executorService2.submit(() -> countDownLatch.countDown());
        }
        countDownLatch.await();
        executorService2.shutdown();
        long end2 = System.currentTimeMillis();
        System.out.printf("countDownLatch : %s ", end2 - start2);

    }
}
```

**결과**

```
slowCountDownLatch : 167 countDownLatch : 126
```

래치를 기다리는 스레드를 1,000개를 만들어서 자바의 CountDownLatch와 비교해보니 약 10배가 느렸다.

> 하지만 실제로 돌려보니 10배까진 아니였다... 아마도 컴파일러가 최적화 하는걸지도?

이 예가 좀 억지스러워 보일지 모르지만, 하나 이상의 스레드가 필요도 없이 실행 가능한 상태인 시스템은 흔하게 볼 수 있다.

이런 시스템은 성능과 이식성이 떨어질 수 있다.

특정 스레드가 다른 스레드들과 비교해 CPU 시간을 충분히 얻지 못해서 간신히 돌아가는 프로그램을 보더라도

`Thread.yield` 를 써서 문제를 고쳐보려는 유혹을 떨쳐내자.

증상이 어느 정도는 호전될 수도 있지만 이식성은 그렇지 않을 것이다.

처음 JVM에서는 성능을 높여준 yield가 두 번째 JVM에서는 아무 효과가 없고,

세 번째에서는 오히려 느려지게 할 수도 있다.

`Thread.yield` 는 테스트할 수단도 없다.

차라리 애플리케이션 구조를 바꿔 동시에 실행 가능한 스레드 수가 적어지도록 조치해주자.

이런 상황에서 스레드 우선순위를 조절하는 방법도 있지만,

역시 비슷한 위험이 따른다.

**스레드 우선순위는 자바에서 이식성이 가장 나쁜 특성에 속한다.**

스레드 몇 개의 우선순위를 조율해서 애플리케이션의 반응 속도를 높이는 것도 일견 타당할 수 있으나,

정말 그래야 할 상황은 드물고 이식성이 떨어진다.

**심각한 응답 불가 문제를 스레드 우선순위로 해결하려는 시도는 절대 합리적이지 않다.**

진짜 원인을 찾아 수정하기 전까지 같은 문제가 반복해서 터져 나올 것이다.
