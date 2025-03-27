---
title: "범위 지정 함수란?"
excerpt: "apply, with, let, also, run, ..."

categories:
  - "Kotlin"
tags:
  - ['Kotlin']

permalink: /categories/dev/kotlin/range-function

toc: true
toc_sticky: true

date: 2024-08-20
last_modified_at: 2025-03-27
---

# 범위 함수란?

- 특정 객체에 대한 작업을 블록 안에 넣어 실행할 수 있도록 하는 함수
- 블록은 특정 객체에 대해 할 작업의 범위가 되며, 따라서 범위 지정 함수이라 한다
- 특정 객체에 대한 작업을 블록안에 넣게 되면 가독성이 증가하여 유지 보수가 쉬워진다
- 5가지의 범위 지정함수 지원

```kotlin
1. apply
2. run
3. with
4. let
5. also
```

- 범위 지정함수는 다른 말로 수신객체 지정 람다(함수)라 한다
- 수신객체를 명시하지 않거나 it을 호출하는 것만으로 람다 안에서 수신객체의 메서드를 호출할 수 있도록 해주기 때문
- 블록(block) 람다식에서 수신객체를 람다의 입력 파라미터 혹은 수신객체로 사용해서 가능하다

# also와 apply의 차이

- also에서의 block은 람다식의 입력 파라미터로 also의 수신객체(T)를 지정

![img.png](/assets/images/posts_img/dev/kotlin/activation_function/img.png)

- apply에서의 block은 람다식의 수신객체로 apply의 수신객체(T)를 지정

![img2.png](/assets/images/posts_img/dev/kotlin/activation_function/img2.png)

- 람다 블록에서 수신객체 지정함수의 수신객체를 명시하지 않고 접근 가능하거나 it으로 접근 가능
- 표로 정리하면 다음과 같다


![img3.png](/assets/images/posts_img/dev/kotlin/activation_function/img3.png)

<br/>

![img.png](/assets/images/posts_img/dev/kotlin/activation_function/img4.png)

# 차례대로 살펴보기

- 다음의 클래스를 통해 좀 더 이해해보자

```kotlin
data class Employee(
    var name: String = "",
    var rank: String = "",
    var age: Int = 0
)
```

## apply

- 수신객체 내부 프로퍼티를 변경한다음 수신객체 자체를 반환하기 위해 사용되는 함수
- 객체 생성 시에 다양한 프로퍼티를 설정해야 하는 경우 자주 사용


- apply에서의 block은 람다식의 수신객체로 apply의 수신객체(T)를 지정한다
- 람다식 내부에서 수신객체에 대한 명시를 하지 않고 함수를 호출 가능

```kotlin
public inline fun <T> T.apply(block: T.() -> Unit): T
```

- apply를 활용하면 다음의 방법으로 수신객체의 프로퍼티 지정이 가능하다
- 람다식의 수신객체가 apply의 수신객체이기 때문에 수신객체에 대한 명시를 생략하는 것이 가능

```kotlin
val employee = Employee().apply { 
  name = "James"
  rank = "Junior"
  age = 32
}
```

## run

- apply와 똑같이 동작하지만 수신 객체를 return하지 않고, run 블록의 마지막 라인을 return
- 수신객체에 대해 특정한 동작을 수행한 후 결과값을 리턴 받아야 할 경우 사용

```kotlin
public inline fun <T, R> T.run(block: T.() -> R): R
```

- 나이가 30살 이상이면 건강검진 대상자라고 해보자

```kotlin
data class Employee(
    var name: String = "",
    var rank: String = "",
    var age: Int = 0
) {
  fun isHealthCheck(): Boolean = age >= 30
}

fun main() {
  val employee = Employee(name = "James", rank = "Junior", age = 28)
  val result = employee.run {
    age = 32
    isHealthCheck() // return 값
  }
  println("건강점진대상: $result")
}

[output]
건강점진대상: true
```

- 수신객체 없이도 동작 가능하지만 이럴땐 내부에 수신객체를 명시

```kotlin
val employee = Employee("James", "Junior", 32)
val isHealthCheck = run {
  employee.age = 30 
  checkHealthCheckTarget(employee)
}
```

## with

- 수신객체에 대한 작업 후 마지막 라인을 return
- run과 완전히 똑같이 동작하지만, with은 수식객체를 파라미터로 사용
- run을 사용하는 것이 깔끔하므로 실제로는 거의 사용 X


- run의 예시와 똑같이 사용하면 다음과 같은 예시

```kotlin
fun main() {
    val employee = Employee(name = "James", rank = "Junior", age = 28)
    val isHealthCheck = with(employee) {
      age = 32
      isHealthCheck() // return 값
    }

    println("isHealthCheck : $isHealthCheck")
```

## let

- 수신객체를 이용해 작업을 한 후 마지막 줄을 return 할 때 사용
- run이나 with과는 수신객체를 접근할 때 it을 사용해야 한다는 점만 다르고 나머지 동작은 동일

```kotlin
public inline fun <T, R> T.let(block: (T) -> R): R
```

- 다음과 같은 경우에 사용
1. null check 후 코드를 실행해야 하는 경우
2. nullable한 수신객체를 다른 타입의 변수로 변환해야 하는경우

- let을 이용해 null을 check를 하려면 아래와 같이 null check 연산자인 '?'와 함께 사용
- 예를 들어 사람이 null이 아닐 때만 일을 시킨다고 하자

```kotlin
fun main() {
    var employee: Employee? = null
    val isWorking = employee?.let { it: Employee ->
        workEmployee(it)
    }
}
```

- employee는 nullable한 객체(Employee?)였다
- ?.let을 사용하면 let block 내부에서는 더이상 nullable하지 않은 it : Employee 이 된 것을 확인 가능
- employee를 통해 일을 시키고 결과값을 return 받았으므로 employee 객체가 다른 타입의 변수로 변환된 것 또한 확인

## also

- apply와 마찬가지로 수신객체 자신을 반환
- 프로퍼티 세팅 뿐만아니라 객체에 대한 추가적인 작업(로깅, 유효성 검사 등)을 한 후 객체를 반환할 때 사용
- 람다식의 입력 파라미터로 also의 수신객체(T)를 지정하기 때문에 내부에서 수신객체를 사용하기 위해서는 it을 사용

```kotlin
public inline fun <T> T.also(block: (T) -> Unit): T
```

- count를 반환받는 함수를 만든 후 해당 count의 숫자를 올리고 싶을 때 다음과 같이 count을 return한 다음 count의 값

```kotlin
var count = 3

fun getAndIncreaseCount() = count.also {
  count++
}

fun main() {
  println("first count ${getAndIncreaseCount()}")
  println("second count ${getAndIncreaseCount()}")
}

[output]
first count 3
second count 4
```

- 주의할 점은 객체를 사용할 때는 객체의 주소값을 return 하는 것이기 때문에 객체의 프로퍼티가 바뀌면 also에서 return하는 객체의 프로퍼티 또한 바뀐다는 점
- 객체의 프로퍼티를 다음과 같이 바꾸어 버릴 경우 바뀐 프로퍼티가 객체의 값이 되버린다

```kotlin
var employee = Employee("James", "Junior", 29)

fun getAndIncreaseAge() = employee.also {
    employee.age = it.age + 1
}

fun main() {
    println("employee ${getAndIncreaseAge()}")
    println("employee ${getAndIncreaseAge()}")
}

[output]
employee Employee(name="James", rank="Junior", age=30)
employee Employee(name="James", rank="Junior", age=31)
```

- 보통 객체에 대해 같은 용도로 사용하고자 할 때에는 copy를 사용
- 바뀌지 않은 객체가 return됨을 보장 가능

```kotlin
var employee = Employee("James", "Junior", 29);

fun getAndIncreaseAge() = employee.also {
  employee = employee.copy(age = it.age + 1)
}

fun main() {
    println("employee ${getAndIncreaseAge()}")
    println("employee ${getAndIncreaseAge()}")
}

[output]
employee Employee(name="James", rank="Junior", age=29)
employee Employee(name="James", rank="Junior", age=30)
```

- 이러한 문제 때문에 also는 거의 사용되지 않고, 사용할 때는 프로퍼티를 바꾸지 않고 동작을 추가적으로 해야하는 경우(로깅 등)에서만 가끔 사용
