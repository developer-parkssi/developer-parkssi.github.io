---
title: "Max Open Files Limit 미적용"
excerpt: "ulimit 수치 오류"

categories:
  - "Trouble Shooting"
tags:
  - ['Trouble Shooting']

permalink: /categories/troubleshooting/openFilesLimit

toc: true
toc_sticky: true

date: 2024-08-28
last_modified_at: 2024-08-28
---

# 30%나 초과했다고?

- 어느날 Slack Alarm으로 어떤 서비스의 process open files가 전체 대비 30%를 초과했다는 메시지가 왔다

![img.png](/assets/images/posts_img/troubleshooting/max-open-files-limit/img.png)

- 해당 모니터링에 사용하는 PromQL은 다음과 같다

```shell
(process_files_open_files/ process_files_max_files) * 100
```

- `process open files`란 프로세스가 작업을 하는데 열어놓은 파일의 수   
  ex) f.write, f.read일 때 사용하는 파일
- 과도하게 열고 닫거나, 열고서 안 닫으면 무한정으로 수가 늘어나 최대치에 이르렀을 때 서비스가 내려가버린다
- 위에서 process_files_max_files를 넉넉히 65000개 정도로 설정했는데, 30%라면 최소한 현재 열려있는 파일 개수가 18000개 정도는 돼야한다-
- 이 정도면 서비스 셧다운이 될 정도의 수치... 뭔가 이상하다...

# 잘못된 설정값

- process_files_max_files만 찍어봤을 때 아래의 수치가 나왔다

![img2.png](/assets/images/posts_img/troubleshooting/max-open-files-limit/img2.png)

- default 값인 4096로 출력이 되는거보면, 서버 설정이 제대로 안 먹혔다고 생각!
- 서버에서 max user process, open files 수치를 확인해봤다

```shell
ulimit -a

[output]
...
open files                      (-n) 65536
max user processes              (-u) 65536
```

- 예상대로 잘 설정되어있다... 그렇다면 다른 곳에서 설정을 해야되는거 아닐까?
- 찾아보니 'service'로 등록해서 실행한 것들은 해당 쪽 설정을 바꿔야 한다고 한다 [참조](https://www.jacobbaek.com/1338)

# Service 설정 변경

## 1. 설정 파일 수정
- 위의 '참조'에서는 service 실행 파일을 수정했는데 나는 default 설정을 아예 바꿔버렸다

```shell
</etc/systemd/system.conf>
[Manager]
...
DefaultLimitNOFILE=65536


</etc/systemd/system.conf>
[Manager]
...
DefaultLimitNOFILE=65536
```

## 2. 설정 적용
- **systemctl daemon-reexec**

## 3. 서비스 재실행
- **systemctl restart xxx.service**

# 결과

## process_files_max_files

- 기존 default 값 4096 -> 65536으로 잘 수정됐다

![img3.png](/assets/images/posts_img/troubleshooting/max-open-files-limit/img3.png)

## 최대치 대비 현재 process open files

- 30% 이상까지 치솟았던 수치들도 안정화됐다

![img4.png](/assets/images/posts_img/troubleshooting/max-open-files-limit/img4.png)

# 결론

- process마다 open files 수치가 max를 넘어가게 되면 서비스가 중지되기 때문에 조기에 발견된게 다행이었다고 생각한다
- service 별로 설정하게끔 만든건 실행되는 인스턴스 별로 custom하게 적용시키게끔 하는 의도이지 않나싶다
