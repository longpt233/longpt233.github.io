---
layout: post
title: Các chiến lược cache
tags:
  - system-design
---

chính xác là các chiến lược update cache

# Overview
 
- cache aside
- write through
- write behind...

# So sánh

|chiến lược| cách thực hiện | ưu | nhược | 
|-|-|-|-|
|Cache aside| đọc trong cache, nếu không có đọc trong db rồi update vào cache| lazy loading | cache miss chậm (do phải update), dữ liệu cũ (cần set ttl hoặc update dữ liệu - write-through), node chết (ban đầu cache miss hết)
|Write-through| đọc ghi gì cũng qua cache trước, rồi mới update vào db chính (sync) | giải quyết vấn đề có thể đọc vào dữ liệu cũ của cache aside  | data có thể không được đọc bao giờ(cần set ttl)
|Write-behind| như write through, nhưng update vào db chính là async | giải quyết vấn đề write heavy |data lost(vì là async)


# Vấn đề của cache
- cache invalid
- cần code thêm, đôi khi là cũng phức tạp

# Ref 

[system-design-primer](https://github.com/donnemartin/system-design-primer?tab=readme-ov-file#when-to-update-the-cache)




