---
title: "에러 그만 먹어라"
excerpt: "e.printStackTrace()의 위험함"

categories:
  - "Trouble Shooting"
tags:
  - ['Trouble Shooting']

permalink: /categories/troubleshooting/printStackTrace

toc: true
toc_sticky: true

date: 2024-08-07
last_modified_at: 2024-08-07
---
# 누가 자꾸 에러 먹냐
## "동의어 부스팅이 안 먹어요ㅜㅜ"
- 검색어의 동의어에 동일한 랭킹 부스팅이 적용이 안 된다는 보고를 받았다
- 동의어 부스팅을 거는 배치가 있길래 들여다보기로 했다
- Airflow 배치 로그를 보려고 했는데 로깅이 엄청 많은지 무한 로딩이 걸리더라

![img](/assets/images/posts_img/troubleshooting/print-stack-trace/img.png)

- 할 수 없이 배치 코드를 디버깅을 통해 하나하나 따라가보기로 했다
- 원인은 어떠한 클래스로 매핑을 하는데, 생성자가 없어 값들을 못 넣어 터지고 있었다
- 하지만 그게 중요한게 아니다. 터졌으면 우리가 진작에 알아야하는데 그걸 몰랐던 것
- 몰랐던 이유는 요놈이다 ㅡㅡ 로깅만 하고 넘어가기에 배치가 실패로 안 뜨고 성공으로 끝난다

```java
try {
  return objectMapper.readValue(value, 터지는클래스.class);
} catch (IOException e) {
  // 안 배부르냐
  e.printStackTrace();
}
```

- 학창시절 코드 자동완성으로 신명나게 쓰던 'e.printStackTrace()', 오랜만에 보니 반가웠다
## 그럼 다른 곳도 있냐?
- 해당 메소드를 사용하는 배치만 위의 것 포함해서 37개였다...
- 노가다로 하기엔 빡세니 intellij Replace로 한번에 바꿔줬다
![img2](/assets/images/posts_img/troubleshooting/print-stack-trace/img2.png)

- 처음에는 그냥 배포하고 배치 에러 알람 울리면, 담당자가 원인 찾아내서 고치려고 했다
- 근데 이번 장애 담당이 나여서 솔직히 앞날이 두려웠다.. 일단 PR로 올리고 배포하기전에 각자 한번 체크해보기로...
- 아래 커밋 로그를 보면 적용된 갯수가 ㄷㄷ 저게 다 터진다고 생각하면 으악
![img3](/assets/images/posts_img/troubleshooting/print-stack-trace/img3.png)

# printStackTrace() 한번도 안 쓴 사람이 있으면 내게 돌을 던져라
## 알았으면 됐다..
- 예전 학창 시절도 생각나고, 처음에 개발 시작할 때 누구나 해왔던 실수니 웃으며 넘어갔다
- 오히려 이런 레거시를 못 고친 우리가 문제...

# 결론
- 여윽시 에러는 붙잡는 순간 탈이 생긴다
- 지금 발생하는 에러 잡기 귀찮아서 로깅만 하고 넘어가는 습관은 버리는게 좋은 것 같다
- 배포한 다음에 대응 생각하니 무섭다


