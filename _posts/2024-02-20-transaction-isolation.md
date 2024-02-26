---
layout: post
title: Transaction Isolation
tags:
  - sql
---

consitency in acid 

# Read problem

Các kịch bản transaction T

|Read problem| Khái niệm | Ví dụ|
|-|-|-|
|Dirty read| đọc vào data chưa được commit| T1 sửa, T2 đọc, T1 roolback. T2 sẽ đọc được giá trị chưa từng tồn tại|
| Non repeatable read| đọc 1 giá trị 2 lần trong 1 transaction nhưng ra 2 giá trị khác nhau| T1 đọc, T2 sửa, T1 đọc. T1 đọc lần 2  ra giá trị khác lần 1 (T2 không thêm xóa)| 
| Phantom read| xử lí trên một tập dữ liệu mà có thằng khác ghi vào tập dữ liệu đó| T1 đọc, trong khi đó T2 thêm, xóa các bản ghi mà T1 đã đọc.|

# Khóa 

Các cấp độ khóa: Database - Table - Page - Row - Field

Các loại khóa: Binary, Shared / Exclusive Lock. 

# Isolation levels

sắp xếp theo tăng dần isolation. các level thấp hơn thì concurency nhanh hơn nhưng đổi lại là data có thể inconsistencies. 

|level| Khái niệm | lock |
|-|-|-|
|Read Uncommitted| bị dirty read: T1 có thể đọc T2 chưa commit.  | không có lock giữa các T |
|Read Committed| bị Non repeatable read: T1 luôn read được dữ liệu mới nhất nếu T2 đã commit.  | T giữ R,W lock với row đấy, các T khác không đọc, sửa, xóa được|
|Repeatable Read| vẫn bị Phantom read | T giữ R,W lock với cả các row liên quan |
|Serializable | giải quyết hết các vấn đề read |


# Q

Nếu 2 connection vào db cùng update vào một bản ghi thì sao ? Tuỳ db postgres là read committed (chủ yếu là read commited). mysql thì cao hơn repeatable read


# Ref 

[geeksforgeeks](https://www.geeksforgeeks.org/transaction-isolation-levels-dbms/)

[viblo](https://viblo.asia/p/transaction-o-muc-do-co-lap-isolation-level-1ZnbRlWNv2Xo)




