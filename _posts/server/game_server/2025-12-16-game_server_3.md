---
title:  "[게임 서버] 3. 소켓 프로그래밍"
excerpt: "3.0"

categories:
  - GameServer
tags:
  - [GameServer]

toc: true
toc_sticky: true
 
date: 2025-12-16
last_modified_at: 2025-12-16
---
# 📦 3. 소켓 프로그래밍
## 🤔 소켓 핸들을 사용하는 경우

- 디스크에 데이터를 쓰거나 읽을 때
- 네트워크로 데이터를 전송하거나 받을 때

## ✅ 특징

- 서버에서 다뤄야 하는 소켓 개수가 많다.
    - TCP로 통신할 때, 클라이언트 개수만큼 소켓이 있어야 한다.
- 파일 핸들하는 동안 스레드가 대기하면 안된다.
    - 애니메이션을 보여주고자 하면, 일시정지된 것처럼 보인다.

## 📌 처리 방식 종류

1. 비동기 입출력
    - 논블로킷 소켓 방식
    - Overlapped I/O 방식
    - epoll 방식
    - I/O Completion Port(IOCP) 방식
2. 동기 입출력
    - 블로킹 소켓