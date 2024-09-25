---
title: "왜 등록이 안되지?"
excerpt: "eureka client instance registration"

categories:
  - "Trouble Shooting"
tags:
  - ['Eureka']

permalink: /categories/troubleshooting/eurekaClientRegistration

toc: true
toc_sticky: true

date: 2024-09-25
last_modified_at: 2024-09-25
---

# Problem

- 야심차게 Kotlin Coroutine을 사용해서 만든 오타보정 API 프로젝트를 배포했다
- 모니터링을 했는데 아무런 문제없이 순탄했기에 가슴이 벅차 올랐다
- 문제는 Eureka에 instance에 전부 등록이 안된다는 것...

**Eureka Server 마다 등록이 되어있는 instance가 다르다...**

![img.png](/assets/images/posts_img/troubleshooting/eureka-instance-registration/img.png)

# Cause

- 원인을 찾기 어려운게 한 서버에서만 등록이 안 되는 것이 아니라, 어떤 서버에선 등록이 되고, 또 다른 서버에선 등록이 안 된다;
- 배포된 소스 코드도 동일하고, 다른 서비스의 API와 다른 점을 찾으려고 해도, 같은 온프레미스(On-premise) 서버를 사용하고 있기에 환경도 동일하다

# Solve

- 서버 로그를 보니깐 에러 로그는 아래처럼 나온다

```
was unable to send heartbeat!
com.netflix.discovery.shared.transport.TransportException: Retry limit reached; giving up on completing the request

...

de-registration failedRetry limit reached; giving up on completing the request
com.netflix.discovery.shared.transport.TransportException: Retry limit reached; giving up on completing the request
```

- Eureka Server 쪽으로 무언가 시도를 하다가 실패해서 instance를 등록 못하는 것이라 생각했다!
- 찾아보니깐 아래처럼 설정을 해보란다

```yaml
eureka.client.fetch-registry=false
eureka.client.register-with-eureka=false
```

- **eureka.client.fetch-registry=false** 은 Eureka에 등록되어있는 다른 서비스 목록을 가져오는건데, 배포한 API는 그럴 필요가 없어서 false로 시도해봤다

# Solve

- 설정을 추가하고 배포해보니 client instance 전부 잘 등록된걸 확인했다

![img2.png](/assets/images/posts_img/troubleshooting/eureka-instance-registration/img2.png)

# 결론

- Eureka Server에 Client 등록하는게 자동으로 뚝-딱 되는줄 알았는데 생각보다 복잡했다
- 나중에 Feign, GRPC도 적용해볼 예정인데, 그렇게 되면 다른 Client 서비스 목록도 가져와야 된다고 한다
- 그러면 위에 추가한 설정을 빼야되는데 그때는 어쩌지...? 미래의 나에게 맡긴다..
