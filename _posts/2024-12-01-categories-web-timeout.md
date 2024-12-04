---
title: "Read Timeout Connection Timeout"
excerpt: "이 둘 차이는 뭘까?"

categories:
  - "WEB"
tags:
  - ["WEB", "Timeout"]

permalink: /categories/web/read-connection-timeout

toc: true
toc_sticky: true

date: 2024-12-01
last_modified_at: 2024-12-01
---

# Read Timeout, Connection Timeout

- API 요청, DB 접근 등의 설정을 할 때 항상 나오는 Timeout
- 보통 넉넉히 잡고 잘 돌아가면 넘어갔는데 정확히 개념을 알고 싶어졌다

![img.png](/assets/images/posts_img/dev/web/timeout/img.png)

## Connection Timeout

- TCP 3 way handshake에서 발생하며 예상한 시간까지 제대로 연결을 못해 발생하는 timeout
- 대부분의 문제가 방화벽 설정이라고 한다. 그 놈의 방화벽...

## Read Timeout

- 연결을 완료한 후 요청을 보냈는데 예상 응답시간을 넘어도 응답이 돌아오지 않아 발생하는 timeout
- 가장 흔하게 신경 쓰게될 값이다. 이를 얼마나 설정하냐에 따라 retry를 몇번하고, delay를 얼마나 걸지 생각하기 때문

# 결론

- 그냥 막연하게만 생각했던 개념들을 다시 고찰할 수 있는 것 같다



