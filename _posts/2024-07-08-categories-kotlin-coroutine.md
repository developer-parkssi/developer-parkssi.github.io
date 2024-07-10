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

```
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

```
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
