---
title: "Coroutine이 뭔데 그렇게 좋을까?"
excerpt: "Coroutine 하나 때문에 Kotlin 전환하는 건에 대하여.."

categories:
  - "Kotlin"
tags:
  - [Kotlin]

permalink: /categories/dev/kotlin/coroutine

toc: true
toc_sticky: true

date: 2024-07-08
last_modified_at: 2024-07-10
---

# 코루틴 하나만으로 코틀린?
## 눈앞이 깜해지는 비동기 구현
- 내가 맡은 프로젝트는 여러 API를 묶어서 리턴해주는 Aggregation 책임이 많아서 비동기 처리할 일이 많았다
- Restful API를 개발할 때 동시성 프로그래밍이 필요하면 이와 같이 구현했다

```java
public String processA () {
  System.out.println("작업A");
  return "작업A";
}

public String processB () {
  System.out.println("작업B");
  return "작업B";
}
CompletableFuture<String> deferredA = CompletableFuture.supplyAsync(() -> processA());
CompletableFuture<String> deferredB = CompletableFuture.supplyAsync(() -> processB());

CompletableFuture<Void> combinedFuture = deferredA.thenCombine(deferredB, (resultA, resultB) -> {
  System.out.println("Result A: " + resultA);
  System.out.println("Result B: " + resultB);
  return null;
});

combinedFuture.get();
```
- 이게 몇개씩 쌓이니깐 가독성이 상당히 나빠지고, 무엇보다 개발하기가 빡세다
- 만약에 이러한 API를 다음에 개발한다고 하면 RxJava를 이용해서 Reactive하게 구현하려 했다
- 그런데 러닝커브도 높고, 비동기식 코드가 눈에 쉽사리 들어오지가 않았다ㅜ
- 이때 요즘 핫하다는 코틀린, 그것도 코루틴이 눈에 들어왔다

## 뭐하러 비동기처럼 구현해? 동기식으로 구현하면 되잖아?
- 나는 비동기식만의 코드? 문법? 이런게 되게 까다로웠다
- 물론 학습하고, 개발하다보면 익숙해지는데 러닝커브가 높은 방식은 입문하기가 어려운 사실
- 하지만 우리의 킹틀린은 그렇지 않다..!

```kotlin
suspend fun processA(): String {
  System.out.println("작업A");
  return "작업A";
}

suspend fun processB(): String {
  System.out.println("작업B");
  return "작업B";
}

runBlocking {
  val deferredA = async { processA() }
  val deferredB = async { processB() }

  val resultA = deferredA.await()
  val resultB = deferredB.await()

  println("Result A: $resultA")
  println("Result B: $resultB")
}
```
- 비동기 처리할 부분만 감싸주면 위와같이 쉽게 개발이 가능하다
- 비동기 처리를 동기식처럼 개발할 수 있다는 이점만으로 써보기로 결정했다

# 그럼 코루틴이 무엇이냐?
- 찾아보니깐 코루틴은 보통 크게 3가지로 설명한다. 각각 간단하게만 설명한다
1. 협력형 멀티 태스킹
2. 동시성 프로그래밍
3. 비동기를 동기처럼

## 협력형 멀티 태스킹
![img.png](/assets/images/posts_img/dev/kotlin/why_kotlin/img.png)
- co-routine이라는 말답게 코루틴 블럭 내부에 suspend를 만나면 밖으로 오고갈 수 있다
- 예를 들어서 아래와 같이 `processAll` 함수에 들어가서 `processA` 함수가 끝날 때까지 기다리는게 아니라 `processAll` 함수밖으로 나와 다른 코드를 실행하다 와도 된다는 것

```kotlin
fun processAll() {
  startCoroutine {
    processA()
    processB()
  }
}

suspend fun processA() {
  delay(5000)
}

suspend fun processB() {
  delay(6000)
}
```

## 동시성 프로그래밍
- 동시성과 병렬성이 헷갈릴 수 있는데, 동시성은 1개의 스레드로 여러 작업을, 병렬성은 여러개의 스레드로 여러 작업을 하는거다
- 위의 협력형 멀티 태스킹으로 '동시성 프로그래밍' 구현이 가능하다
- 각각의 코루틴이 suspend를 만날 때마다 번갈아가면서 실행이 가능하기 때문이다

```python
Time -->
Coroutine A: |-----|     |-----|     |-----|
Coroutine B:     |-----|     |-----|     |-----|
```

## 비동기를 동기처럼
- 사실 내가 코루틴을 사용하려는 가장 큰 이유다
- `suspend` 키워드만 달아주면 비동기 함수를 정말 편하게 만들 수 있다

```kotlin
suspend fun processA() {
  taskA()
  taskB()
  taskC()
}
```

# 결론
- 웹 개발을 하다보면 비동기 처리를 해야되는 순간이 맞이할 수 밖에 없다고 생각한다
- 요즘 가상스레드가 떠오르고 있어서, 비동기 처리라는 개념을 고려해볼 필요가 없다곤 하지만... 팀에서 테스트 해봤을 때는 아직은 무리인듯 하다..
- 지인분 한분의 얘기가 공감이 갔다. '가상 스레드가 그렇게 난리였으면 RxJava 처음 나왔을 때처럼 아주 난리가 났겠지?'
- 어쨌든 비동기 처리에 대한 고민을 코틀린이 해결해주는 것 같다. 다음 프로젝트는 킹틀린.. 너로 정했다..
