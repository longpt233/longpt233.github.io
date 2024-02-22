---
layout: post
title: Airflow HA
tags:
  - bigdata
---

airflow và ứng dụng

# Khái niệm 

- cho phép xây dựng flow các task
- cung cấp ui thuận tiện cho việc theo dõi và quản lí tập trung các task
- hạn chế: không sử dụng để truyền dữ liệu lớn giữa các task. không sử dụng với các task vô hạn (như streaming)

# Các thành phần của airflow

- scheduler
- executor -> worker 
- ngoài ra: metadata db, dag dir, web server

# Scheduler


# Executor

các loại executor 

- local: local executor, sequential executor 
- remote: celery executor, kubernetes executor 

nên dùng celery executor vì nó có thể scale được số lượng các worker thông qua celery backend (rabbitMQ, redis). executor như kiểu một cách thức để task được giao từ scheduler tới worker (đứng giữa nơi lập lịch và nơi thực thi task)

![](../images/2024-02-22%2021-44-01.png)

# Mô hình ha

# note 
- xcom (cross-communications) để truyền thông tin giữa các task trong dag
- get_pty=True để kill được task trên ui
- do_xcom_push=False để tránh trường hợp xcom lưu output> 65kb dẫn đến task fail(sử  dụng print hoặc return function)



# Ref 

[lotus doc - hiephm](https://lotus.vn/w/blog/gioi-thieu-ve-airflow-va-trien-khai-kien-truc-ha-348074727123714048.htm)




