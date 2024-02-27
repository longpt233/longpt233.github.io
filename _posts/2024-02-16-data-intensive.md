---
layout: post
title: Data Intensive Application
tags:
  - bigdata
---

sách nhập môn cho bigdata ?

chương 3: lưu trữ và truy vấn

log structured và page-oriented storage engine

## cấu trúc dữ liệu của db

- ghi vào cuối file, khi tìm sẽ phải duyệt toàn bộ O(n)
- để tăng tốc độ tìm kiếm thì sử dụng **kiểu dữ liệu** index, tuy nhiên sẽ làm giảm hiệu năng truy vấn dữ liệu
- việc sử dụng index là do người dùng cần nhắc sử dụng dựa trên query pattern, tải hệ thống
    
## hash index
- map key-offset. khi tìm giá trị key -> thì tìm tới offset của key đó trong file
- lưu toàn bộ trong mem -> tràn -> segment, compact
- segment: chia dữ liệu thành các file nhỏ, khi đầy việc ghi sẽ ghi tiếp vào segment mới. trên 1 segment có thể có nhiều giá trị của cùng 1 key (thời gian update khác nhau), vì vậy mới cần quá trình compact và merge lại để tăng tốc độ tìm kiếm
- compact: loại bỏ các key trùng và giữ lại key có giá trị mới nhất. diễn ra trên từng segment.
- merge: merge các segment lại với nhau để tăng tốc độ tìm kiếm (giam số lượng segment)
- chú ý
    - khi xóa thì cần đánh dấu là delete, sau đó khi merge sẽ xóa đi là được
    - crash recovery: đọc lại toàn bộ segment file -> snapshot định kì
    - concurency control: chỉ để 1 luồng ghi duy nhất
- ưu/ nhược điểm:
    - ưu điểm: nhanh hơn random- write (over-write).
    - nhược điểm: out mem, range query cần đọc nhiều

## SSTable và LMS-Tree
- sorted string table: sắp xếp các key
- việc ghi sẽ chậm hơn do dữ liệu cần được sắp xếp
- sparse index inmemory: vì dữ liệu được sort nên chỉ cần lưu index thưa với các dải key khác nhau (ví dụ trong index chỉ cần có giá trị a, c thì nếu tìm b thì ta chỉ cần quét giá trị nằm giữa a và c là được)


# Ref 

[ngtung medium](https://ngtung.medium.com/l%C6%B0%E1%BB%A3c-d%E1%BB%8Bch-designing-data-intensive-applications-ph%E1%BA%A7n-i-ch%C6%B0%C6%A1ng-1-8d54dc7ce70b)