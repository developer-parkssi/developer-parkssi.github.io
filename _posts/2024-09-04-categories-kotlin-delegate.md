---
title: "Delegate pattern을 편하게 써보자"
excerpt: "상속보단 composition?"

categories:
  - "Kotlin"
tags:
  - ['Kotlin']

permalink: /categories/dev/kotlin/delegate

toc: true
toc_sticky: true

date: 2024-09-04
last_modified_at: 2024-09-04
---

# Delegate Pattern이란?

- 객체 합성이 상속과 동일하게 코드 재사용을 할 수 있도록 하는 객체 지향 디자인 패턴
- 한 객체가 다른 객체로부터 기능 일부를 넘겨받아 데이터를 제공하거나 특정 작업을 수행 할 수 있게 하는 것
- 보통 상속 대신에 Composition을 사용할 때 활용하면 유용

# Delegation In Kotlin

- Kotlin에서는 Delegate Pattern을 `by`라는 키워드로 제공
- Delegation을 구현하는 데에 있어서 보일러 플레이트 코드를 줄일 수 있다

# Example

- `ElectronicCar`라는 인터페이스에서 전기량을 뜻하는 `battery`, 이를 출력하는 `info()` 메서드 존재

```kotlin
interface ElectronicCar {
    val battery: Int
    fun info()
}
```

- `User`라는 클래스는, `ElectronicCar`을 직접 구현하는 것이 아닌 구현체를 인자로 받아 사용

```kotlin
class User(private val electronicCar: ElectronicCar) : ElectronicCar {
    override val battery: Int = electronicCar.battery
    override fun info() = electronicCar.info()
}

class Tesla : ElectronicCar {
  override val battery: Int = 500

  override fun info() {
    print("사용 가능한 전력양은 500 입니다.")
  }
}

fun main(){
  val user = User(Ezreal())
  user.info()
}
```

- `User`는 Tesla를 상속받지 않더라도 사용 가능
- 하지만 속성, 메서드가 많아질 수록 위임받는게 많아지고, 불필요한 코드를 야기한다

# 코틀린에서 지원하는 키워드 'by'

- 'by' 키워드로 한번에 해결

```kotlin
class User(private val electronicCar: ElectronicCar) : electronicCar by ElectronicCar
```

# 결론

- 상속은 불필요한 코드를 많이 전달받고, 결합도가 높아지기에 'Composition' 사용을 지향하고있다
- 'by'를 쓴다면 위임 클래스의 변경 영향에 부담이 없기에 자주 사용할 것 같다
