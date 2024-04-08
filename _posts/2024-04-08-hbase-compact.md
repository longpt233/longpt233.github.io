---
layout: post
title: Hbase Compaction
tags:
  - hadoop
---

hbase architecture

# Compaction


# Usecase

1. load theo colum family 

- việc chia thành 2 columfamily tương đương với việc tạo 2 bảng có chung key và 1 colum family
- Column families are about performance not schema.
-> cần tính toán hợp lí giữa schema (các cột đó phải cùng 1 table) - performace (nếu bắt buộc cùng 1 table thì có cần tách ra 2 colum để tăng hiệu năng khi chỉ dùng 1 colum hay không)
- Since each column family represents a separate Store on RegionServer, accessing multiple stores takes more time. You can limit your scan to specific column families, use addFamily on your scan object.

