---
title: "Redis 스냅샷을 찍어보자"
excerpt: "Redis Backup"

categories:
  - "Cache"
tags:
  - ['Cache', 'Redis']

permalink: /categories/dev/cache/redis-backup

toc: true
toc_sticky: true

date: 2024-11-20
last_modified_at: 2024-11-20
---

# Issue

## Problem

- 기존에 운영하던 Redis Cluter 구조는 다음과 같다

```shell
Master - Slave
Master2 - Slave2
Master3 - Slave3
```

- Redis Cluster Node 중 1개를 재시작하면 Cluster에서 빠지거나 데이터가 유실됐다
- 캐싱된 데이터가 없으면 에러가 발생하는 API들이 존재했기에 관련 Batch들을 전부 다시 돌려야했다
- 기존 Backup 데이터가 없기 때문이라 판단하여 RDB를 추가하기로 했다

## Why RDB?

- RDB, AOF 방식 중에 고민이 됐는데 스냅샷 방식인 RDB를 선택했다
- 우리가 해결하고 싶었던 것은 운영에서 무중단으로 재기동했을 때 정상 동작이었다
- 이는 RDB만으로 충분하고, 유지보수하기에도 편할 것이라 생각했다
- 그런데 이미 `shutdown SAVE`를 하면 스냅샷 저장이 되는데 운영에 적용이 안되고 있는지 의문이었다

# Progress

## 설정 파일 수정

- redis.conf

```shell

# 900초 동안 명령어 1개 이상 들어오면 스냅샷 생성
save 900 1
save 300 10
save 60 10000

# 스냅샷 Backup 파일 이름
dbfilename dump.rdb
# 스냅샷 Backup 파일 저장 경로
# 기존에는 'dir ./'로 되어있다
dir /data01/sw/redis/redis-5.0.5/data

# 스냅샷을 저장하다가 에러가 났을 때 write 명령어를 멈출 것이냐?
# yes해서 운영중에 에러나면 아작나니 당연히 'no'로 하는 것이 좋다
stop-writes-on-bgsave-error no
```

## 실행 명령어

- Redis 종료

```shell
$ src/redis-cli -a search shutdown SAVE
```

- Redis 가동

```shell
$ src/redis-server conf/redis.conf
```

- cluster node 추가

```shell
$ src/redis-cli --cluster add-node <slave_ip>:<slave_port> <master_ip>:<master_port> --cluster-slave --cluster-master-id <master_node_id>
```

## 확인 명령어

### Cluster 관계 확인

- Node 하나씩 껐켰하고 Cluster에 제대로 들어갔는지 확인할 때 사용

```shell
127.0.0.1:6379> info replication

# Replication
role:slave
master_host: 마스터노드 Host
master_port:6379
master_link_status:up
master_last_io_seconds_ago:0
master_sync_in_progress:0
slave_repl_offset:84296568300763
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:b00db58b6dcb79e99d8d3b96d4908722d77ab843
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:84296568300763
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:84296567252188
repl_backlog_histlen:1048576

# master_link_status: 제대로 올라왔으면 up, 아니면 down
# master_sync_in_progress: 마스터 노드와 sync 완료면 0, 진행중이면 1
```

### Master - Slave 관계 바꾸기

- Slave 노드는 그냥 껐켰해도 기존 Cluster 관계에 맞게 들어간다
- Master 노드는 꺼지면 Slave 노드가 Master가 되버리기 때문에, 기존대로 Master로 올라오려고 하면 적용이 안 된다
- 나같은 경우는 아래 명령어로 Master-Slave를 일단 바꾼 다음에 Slave로 올리고, 기존 관계로 다시 돌렸다

```shell
127.0.0.1:6379> cluster failover
```

## Backup 재실행 시 유의사항

- Backup 데이터인 `dump.rdb`만 필요하면 된다고 생각했는데 이것만 있으면 Node가 기존 Cluster 관계 참조를 못한다
- Redis는 설정 파일 `dir 경로` 값 경로에 `dump.rdb`뿐만 아니라 `node-6379.conf`도 저장한다
- `node-6379.conf`를 실제로 열어보면 Cluster 구조가 나와있다

```shell

b5877acfcbbc1e7f79932c1aabd8a9cc6f666c27 Master:6379@16379 master - 0 1731785121680 10 connected 0-5460
2655d2bc99d1500a38cfbd2d441d3824f5e22f31 Master3:6379@16379 master - 0 1731785121000 12 connected 10923-16383
fb1c0e97d0e5bfbfa1589dff2f1bcd5145eff54f Slave:6379@16379 slave b5877acfcbbc1e7f79932c1aabd8a9cc6f666c27 0 1731785120577 10 connected
619bd9dc59da5c6861cb8dcbe74e778478177d51 Slave2:6379@16379 myself,slave 7b3b5482ce2900fa13ebff4617697ed17045e0b2 0 1731785120000 8 connected
c805aa4ea13504b3247fcdca37b926b1da953dfb Slave3:6379@16379 slave 2655d2bc99d1500a38cfbd2d441d3824f5e22f31 0 1731785122618 12 connected
7b3b5482ce2900fa13ebff4617697ed17045e0b2 Master2:6379@16379 master - 0 1731785120678 11 connected 5461-10922
vars currentEpoch 12 lastVoteEpoch 0
```

- Cluster Node 하나씩 재시작하면서 적용 중이었는데 자꾸 Cluster가 깨져서 당황했다ㅜ
- 그러다가 해당 파일까지 같이 복붙하니 해-결

# 결론

## 기존에 Backup 적용이 안 된 이유
- 기본으로 설정되어있는 RDB 적용이 왜 안되는지 알게됐다
- 백업 경로 주소가 Default `dir ./` 상대 경로로 되어있어서, 명령어를 실행한 시점 그 경로에 파일이 저장된다
- 절대 경로가 아니기 때문에 그때마다 달라지는 것

## 무중단 운영 가능
- 이제 기존 데이터와 Cluster 구조를 유지가능 하기 때문에 무중단 운영이 가능할 것 같다
- Redis 작업 할 때마다 벌벌 떨었는데 이젠 안 그래도 되어서 기분 좋다
