---
layout: post
title: Null Index
tags:
  - sql
---

bạn bảo null thì chẳng ảnh hưởng gì, tôi bảo là chưa chắc

# Lí thuyết 

- câu hỏi là tìm kiếm trên 1 cột có giá trị null thì nó có ăn index hay không 
- các phụ thuộc : lượng null so với không null, tìm giá trị null hay giá trị khác null
- các kết quả dựa trên EXPLAIN (thực tế nó chạy khác thì khóc). mysql version Ver 8.0.36-0ubuntu0.20.04.1 for Linux on x86_64, db engine: innoDb

# Thực nghiệm 


<details>
<summary>Khởi tạo data base</summary>

<div class="highlight"><pre class="highlight">
<code>
DROP DATABASE if exists index_db; 
CREATE DATABASE index_db;
use index_db;

CREATE TABLE example_table (
    id INT PRIMARY KEY,
    name VARCHAR(50),
    age INT,
    salary DECIMAL(10,2)
);

</code>
</pre></div>

</details>

### Kịch bản 1

<details markdown="0">
<summary>insert giá trị null << không null (990001 not null và 1 null)  </summary>


```
INSERT INTO example_table (id, salary) VALUES(6,  null);

DELIMITER //
CREATE PROCEDURE insert_records()
BEGIN
    DECLARE i INT DEFAULT 10000;
    WHILE i <= 1000000 DO
        INSERT INTO example_table (id, salary) VALUES (i, i * 10);
        SET i = i + 1;
    END WHILE;
END //
DELIMITER ;
CALL insert_records();

select count(*) from example_table where salary is not null;
990001

select count(*) from example_table where salary is  null;
1

```

</details>

<details>
<summary>thực thi is null và not null thì đều trả kết quả explain như nhau (đang k có index)</summary>

```
EXPLAIN SELECT * FROM example_table WHERE salary is null;
```

| id | select_type | table          | partitions | type | possible_keys | key | key_len | ref | rows   | filtered | Extra           |
|----|-------------|----------------|------------|------|---------------|-----|---------|-----|--------|----------|-----------------|
| 1  | SIMPLE      | example_table  |            | ALL  |               |     |         |     | 988948 | 10.00    | Using where     |


</details>


<details>
<summary>tiến hành thêm index </summary>

```
ALTER TABLE example_table ADD INDEX idx_salary(salary);

# check xem đã đánh xong chưa 

SELECT * FROM information_schema.processlist;

# check các index hiện có 
show index from example_table;
```

</details>

<details>
<summary>kiểm tra index . có ăn index khi select * where is null, hoặc = const  </summary>

```
EXPLAIN SELECT * FROM example_table WHERE salary is null;
```

| id | select_type | table         | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra                        |
|----|-------------|---------------|------------|------|---------------|------------|---------|-------|------|----------|-----------------------------|
| 1  | SIMPLE      | example_table |            | **ref**  | idx_salary    | **idx_salary** | 6       | **const** | 1    | 100.00   | **Using index condition**       |


```
EXPLAIN SELECT * FROM example_table WHERE salary is not null;
```
| id | select_type | table         | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra                        |
|----|-------------|---------------|------------|------|---------------|------------|---------|-------|------|----------|-----------------------------|
| 1  | SIMPLE      | example_table |            | **ALL**  | idx_salary    |  |      |  | 988948   | 50.00   | **Using where**        |

```
EXPLAIN select * from example_table where salary = 101000;
```

| id | select_type | table         | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra                        |
|----|-------------|---------------|------------|------|---------------|------------|---------|-------|------|----------|-----------------------------|
| 1  | SIMPLE      | example_table |            | **ref**  | idx_salary    | **idx_salary** | 6       | **const** | 1    | 100.00   |      |

</details>


<details>
<summary>kiểm tra index với select count(*) where is null, hoặc = const  </summary>


```
EXPLAIN select count(*) from example_table where salary is  null;

```

| id | select_type | table         | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra                        |
|----|-------------|---------------|------------|------|---------------|------------|---------|-------|------|----------|-----------------------------|
| 1  | SIMPLE      | example_table |            | **ref**  | idx_salary    | idx_salary | 6       | **const** | 1    | 100.00   | Using where; Using index       |


```
EXPLAIN select count(*) from example_table where salary is not null;

```

| id | select_type | table         | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra                        |
|----|-------------|---------------|------------|------|---------------|------------|---------|-------|------|----------|-----------------------------|
| 1  | SIMPLE      | example_table |            | **range**  | idx_salary    | idx_salary | 6       | | 494474    | 100.00   | Using where; Using index     |


```
EXPLAIN select count(*) from example_table where salary = 101000;
```

| id | select_type | table         | partitions | type | possible_keys | key        | key_len | ref   | rows | filtered | Extra                        |
|----|-------------|---------------|------------|------|---------------|------------|---------|-------|------|----------|-----------------------------|
| 1  | SIMPLE      | example_table |            | **ref**  | idx_salary    | **idx_salary** | 6       | **const** | 1    | 100.00   |   Using index   |



</details>



### kịch bản 2

















