---
layout: post
title: Distributed transaction
tags:
  - micro-service
---

transaction micro service

# Khái niệm

The dual write problem

![](../images/2024-02-27%2014-50-35.png)

Trong một transaction boundary. có thể xảy ra các trường hợp kiểu như 

- local-commit-then-publish approach: A commit rồi gửi yêu cầu sang B (qua queue - async). nhưng B chưa commit thì app lỗi
- publish-then-local-commit: gửi sang B trước, rồi A commit. sẽ lỗi hoặc có vấn đề về thời gian khi B ghi trước khi A commit. đây gọi là dual write problem (chưa hiểu chỗ này lắm )


# Cách giải quyết 


|Tên| khái niệm| hạn chế | ví dụ|
|-|-|-|-|
|modular monolith| dùng chung một runtime, chung db | k scale được, high coupling| |
|two-phase commit (2PC)| cho một thằng đứng giữa. thực hiện một thao tác atomic [prepare, synchronous blocking, commit] | k scale, block | Message brokers such as Apache ActiveMQ|
|Orchestration| dùng saga Orchestrator. async commit | single point failure | saga |
|Choreography| dùng các topic khác nhau để các service tự giao tiếp với nhau (có cả rollback)| khó implement| even sourcing, saga|
|Parallel pipelines| null | null |null | 





# Ref 

[developers.redhat](https://developers.redhat.com/articles/2021/09/21/distributed-transaction-patterns-microservices-compared#)

[medium](https://medium.com/cloud-native-daily/microservices-patterns-part-04-saga-pattern-a7f85d8d4aa3)


