---
layout: 
title: Database migration
tags:
  - system-design
---

migration zezo down time

# Vấn đề 

- có nhiều instance cùng đọc vào db 
- nếu thay đổi code mà không thay đổi db thì chẳng vấn đề gì (nói thế chứ khi update code - khởi chạy lại instance thì vẫn cần chú ý về khoảng thời gian thay đổi giữa 2 version, các req đang phục vụ có được sử lí hết không ...)
- vấn đề xảy ra khi update code cần update cả db 


# Cách làm thông thường 

run-db-migrations && launch-the-app (chạy đồng thời app và tiến trình migration)

Hạn chế:

- app phải đợi migration phần liên quan tới nó xong thì mới được chạy tiếp. nếu không sẽ thành sai xót dữ liệu. app có thể bị treo trong thời gian này.
- migration retries nhiều lần. cần phải có cơ chế dừng lại để sửa lại job migration cho đúng 

# Cách làm đúng 
Chạy db migration xong mới chạy app v2

Vấn đề :
- backward incompatibility: vì phải chạy xong db migarion (db2) thì mới update app lên v2 nên db sau khi migration (db) phải tương thích ngược với app v1. nếu không app v1 có thể bị crash
- heavy operation: quá trình migrate có thể thực hiện những tác vụ nặng và lock db. điều này có thể làm ảnh hưởng đến app v1 đang chạy 


# Usecase


<details markdown="1">
<summary>Add new column. non null</summary>

thêm cột user_avatar

Trình tự: 

- thêm cột cho db. nullable 
- sửa cột đó, add thêm giá trị
- sửa cột thành nonnull
- update app v2 


Vấn đề: 
- khi bản v1 vẫn đang chạy thì nó không biết tới cột mới thêm rồi user_avatar. nếu vẫn insert vào thì sẽ lỗi do cột đó là nonnull. 
- nó sẽ đúng tới khi update dc app lên v2 

Cách làm: chia làm 2 giai đoạn
- db add thêm cột, thêm giá trị. app thêm chức năng get 
- db make cột đó là nonnull. app thêm chức năng put


</details>


<details markdown="1">
<summary>Remove column</summary>

xóa cột user_avatar


Vấn đề: 
- xóa cột thì lập tức app lỗi ngay

Cách làm: cũng chia làm 2 giai đoạn
- db cột make nullable. bỏ chức năng liên quan cột đó trên app
- xóa cột


</details>

<details markdown="1">
<summary>Rename or change type column</summary>

Cách làm: chia làm 4 giai đoạn
- thêm cột_mới. app ghi đồng thời vào cột_cũ, cột_mới (cột_mới theo format mới - chứ k phải 2 cột có giá trị như nhau), vẫn đọc tại cột_cũ
- sync dữ liệu từ cột cũ sang mới. app chuyển sang đọc tại cột mới. vẫn ghi vào 2 cột cũ và mới (vì trong quá trình deploy có nhiều instance -> vẫn đảm bảo backward compatibility với các instance v1). 
- cho cột cũ là nullable. app ngừng đọc vào cột cũ
- xóa cột cũ


</details>

Trên là các usecase cho việc backward compatibility. với heavy operations ta có thể dùng modern DB engine, migration khi có ít trafic, chia nhỏ migration để chạy 

# Ref 

[teamplify](https://teamplify.com/blog/zero-downtime-DB-migrations/)




