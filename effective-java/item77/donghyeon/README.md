---
layout: post
title: 예외를 무시하지 말자
categories: [이펙티브자바]
comments: true 
tags:
- 이펙티브자바
---



너무 뻔한 이야기지만, 반복해 각인시켜야 할 정도로 사람들이 자주 까먹고 있다.

API설계자가 메서드 선언에 예외를 명시하는 까닭은,

그 메서드를 사용할 때 적절한 조치를 취해달라고 말하는 것이다.

# 예외를 무시하기

안타깝게도 예외를 무시하기란 아주 쉽다.

해당 메서드 호출을 try문으로 감싼 후 catch 블록에서 아무 일도 하지 않으면 끝이다.

```java
// catch 블록을 비워두면 예외가 무시된다.
try {
  ...
} catch (SomeException e) {
  // 예외를 먹어버림.
}
```

예외는 문제 상황에 잘 대처하기 위해 존재하는데 **catch 블록을 비워두면 예외가 존재할 이유가 없어진다.**

물론 예외를 무시해야 할 때도 있다.

예를 들어 `FileInputStream` 을 닫을 때가 그렇다.

FileInputStream은 입력 전용 스트림으로 파일의 상태를 변경하지 않았으니 복구할 것이 없으며,

스트림을 닫는다는 건 필요한 정보는 이미 다 읽었다는 뜻이니 남은 작업을 중단할 이유도 없다.

혹시나 같은 예외가 자주 발생한다면 조사해보는 것이 좋을테니 파일을 닫지 못했다는 사실을 로그로 남기는 것도 좋은 생각이다.

**어쨌든 예외를 무시하기로 했다면 catch 블록 안에 그렇게 결정한 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 바꿔놓도록 하자.**

```java
Future<Integer> f = exec.submit(planarMap::chromaticNumber):
int numColors = 4;
try {
  numColors = f.get(1L, TimeUnit.SECONDS):
} catch (TimeoutException | ExecutionException ignored) {
  
}
```

이 내용은 검사예외와 비검사 예외에 똑같이 적용된다.

예측할 수 있는 예외 상황이든 프로그래밍 오류든, 빈 catch 블록으로 못 본 척 지나치면 그 프로그램은 오류를 내재한 채 동작하게 된다.

그러다 어느 순간 문제의 원인과 아무 상관없는 곳에서 갑자기 죽어버릴 수도 있다.

예외를 적절히 처리하면 오류를 완전히 피할 수도 있다.

무시하지 않고 바깥으로 전파되게만 놔둬도 최소한 디버깅 정보를 남긴 채 프로그램이 신속히 중단되게는 할 수 있다.