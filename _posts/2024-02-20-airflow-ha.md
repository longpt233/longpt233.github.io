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

# Flower

# Mô hình ha

# Cài airflow

<details markdown="1">
<summary>Cài airflow từ pip</summary>

```
sudo apt update
sudo apt install python3-pip -y

export AIRFLOW_HOME=~/airflow

AIRFLOW_VERSION=2.0.1
PYTHON_VERSION="$(python3 --version | cut -d " " -f 2 | cut -d "." -f 1-2)"
CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

pip3 install "apache-airflow==${AIRFLOW_VERSION}" --constraint "${CONSTRAINT_URL}"

hoặc 
python3.8 -m pip install --upgrade pip
python3.8 -m pip install apache-airflow==2.5.1

```

</details>


```

python3 -m airflow db init 
# lúc này mới tạo ra cái thưu mục AIRFLOW_HOME
# mặc định sql_alchemy_conn = sqlite:////home/airflow/airflow2/airflow.db


CREATE DATABASE airflow_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'airflow_user' IDENTIFIED BY 'airflow_pass';
GRANT ALL PRIVILEGES ON airflow_db.* TO 'airflow_user';

mysql+mysqldb://airflow_db:airflow_pass@192.168.56.1:3306/airflow_db
mysql+mysqlconnector://airflow_db:airflow_pass@192.168.56.1:3306/airflow_db
```


# note 
- xcom (cross-communications) để truyền thông tin giữa các task trong dag
- get_pty=True để kill được task trên ui
- do_xcom_push=False để tránh trường hợp xcom lưu output> 65kb dẫn đến task fail(sử  dụng print hoặc return function)



# Ref 

[lotus doc - hiephm](https://lotus.vn/w/blog/gioi-thieu-ve-airflow-va-trien-khai-kien-truc-ha-348074727123714048.htm)

[git  ha](https://github.com/teamclairvoyant/airflow-scheduler-failover-controller)

[doc](https://airflow.apache.org/docs/apache-airflow/stable/start.html)



