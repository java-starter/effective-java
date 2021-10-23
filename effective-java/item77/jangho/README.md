# [이펙티브 자바] Item77 - 예외를 무시하지 말라

API 설계자가 메서드 선언에 예외를 명시하는 이유는, 그 메서드를 사용할 때 적절한 조치를 취해달라고 말하는 것이다. 하지만 사람들이 자주 어기고 흘려버리는 경우가 많다. 

예외를 무시하는 방법은 아주 쉽다. 단순히 메서드 호출을 try-catch로 감싼 후 catch에서 아무일도 하지 않으면 된다.

```jsx
// catch를 비워두면 예외는 무시된다.
try {
    ......
} catch(SomeException e) {
    // 아무일도 하지 않는다.
}
```

# catch 블록을 비워두지 말자

예외는 문제 상황에 대처하기 위해 존재하는데 **catch 블록을 비워두면 예외가 존재할 이유가 사라진다.**  비유하자면 화재경보를 무시하는 수준이 아니라 아예 꺼버려, 다른 누구도 화재가 발생했는지 모르게하는 것과 같다.

이러한 상황은 끔직한 참사로 이어질 수 있으니 예외를 무시해야 하는 상황이 아니라면 catch 블록은 비워두지 말자.

# 예외를 무시해야 하는 상황이라면..

예외를 무시해야 할 때도 있다. 

대표적으로 FileInputStream을 닫을 때를 예로 들 수 있다. FileInputStream은 입력 전용 스트림이므로 파일의 상태를 변경하지 않았으니 복구할 것이 없고, 스트림을 닫는다는 건 필요한 정보를 다 읽었다는 뜻이니 남은 작업을 중단할 이유도 없다.

따라서 **예외를 무시하기로 했다면 catch 블록안에 이유를 주석으로 남기고 예외 변수의 이름도 ignored로 변경하도록하자.**

```jsx
Future<Integer> f = exec.submit(planarMap::chromaticNumber);
int numColors = 4; // 기본값

try {
   numColors = f.get(1L, TimeUnit.SECOND);
} catch (TimeoutException | ExcutionException ignored) {
   // 기본값을 사용한다.
}
```

# 핵심 정리

이번 아이템의 내용은 **검사와 비검사 예외**에도 똑같이 적용된다. 빈 catch 블록을 그냥 지나치면 그 프로그램은 오류를 내재한 채 동작하게 된다.

예외를 적절히 처리하면 오류를 완전히 피할 수도 있다. 또한, 바깥으로 전파되게만 놔둬도 최소한 디버깅 정보를 남긴 채 프로그램이 중단되게 할 수 있다.

**그러니 예외를 무시하지 말자.**