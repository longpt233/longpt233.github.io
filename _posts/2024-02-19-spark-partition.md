---
layout: post
title: Spark Partition Turning
tags:
  - spark
---

spark faster

# Kiến thức Cơ bản
 
Full data -> các partition -> các partition này sẽ được thực thi trên các executor

Wide transformation: cần dữ liệu từ các partition khác nhau để hoàn thành task. ví dụ: reduceByKey, groupByKey, and join. wide transfomation need "shuffle" which is basically transferring data between the different partitions. this is expensive operation. 

Adaptive query execution(spark 3.) with realtime optimizing, can change number of partition in runtime, depend on dataset size (adaptive)



<details markdown="1">
<summary>Config example</summary>

config cơ bản khi read file: 

```
spark.sql.files.maxPartitionBytes: max bytes trong 1 partition, default 128MB
spark.sql.files.minPartitionNum: default spark.default.parallelism là số core của job
```

config khi bật adaptive 

```
spark.sql.adapative.enabled must be true 
spark.sql.adaptive.coalescePartitions.enabled must be true 

spark.sql.adaptive.advisoryPartitionSizeInBytes
spark.sql.adaptive.coalescePartitions.initialPartitionNum
spark.sql.adaptive.coalescePartitions.parallelismFirst
```

nếu không bật, mặc định số partition là 
```
spark.sql.shuffle.partitions: default 200. 
```

</details>

### Điều chỉnh số partition 

- df.repartition(10): round robin. vẫn tính là shuffle. coalesce  thì không shuffle, chỉ merge các partition có sẵn, nên nhẹ hơn repartition, đổi lại thì dữ liệu gộp lại có thể không đồng đều giữa các partition.
- df.repartiton(10, 'class_id') : hash partition. nếu số lượng partition lớn hơn không gian mà hàm hash trả về -> null partition.
- df.repartitionByRange(10, 'grade'): range partition. giống hash, nhưng nó dùng sampling để estimate range nên có thể inconsistent. 

### Tối ưu partition

- Available resources in your cluster: 3x number of core
- data looks (sizing, cardinality): bình thường thì chỉ cần round robin thôi là đủ.
- nhiều trường hợp cần phải xử lí phức tạp hơn khi các partition theo key là không đều (vì groupby các thứ đều cần chia lại theo key). nếu các key phân bố không đều thì có thể chia key làm 2 phần để tính sau đó gộp lại. 

# Usecase

Tối ưu số partition

```
spark = SparkSession.builder.getOrCreate()

df = spark.read.option('header', 'true').csv('./example_data/dataset_1.csv')
df = df.withColumn('amount', F.col('amount').cast('int'))

# tính tổng amount theo business
df = df.groupBy('business').agg(F.sum('amount').alias('total_amount'))

# tính trung bình amount
df_avg = df.select(F.avg('total_amount').alias('avg_amount'))

# join cross vào để tí tính xem total/ trung bình theo từng business
df = df.crossJoin(df_avg)
df = df.withColumn('compared_to_avg', F.round(F.col('total_amount') / F.col('avg_amount'), 3))
df.write.mode('overwrite').partitionBy('compared_to_avg').csv('.output_data/')
```

|stage  | default | optimize |
|-|-|-|
|(sau khi) load dữ liệu, cast| 192 task = 192 partition = 24G/ spark.sql.files.maxPartitionBytes (128Mb)| 24G/ số core *3 (16 *3) = 500Mb = 54 partition (có thể không phải là 48 vì đó là config max)|
|**time**| 32s|24s|
|(sau khi) groupBy | 11 partition = size kết quả của groupBy / spark.sql.adaptive.coalescePartitions.minPartitionSize (1Mb) | 1 task, vì set parallelismFirst false|
|**time**| 0.4s|0.2s|
|(sau khi) tính trung bình| 1 partition| 1 partition |
|**time** |4ms|2ms |
|(sau khi) join lại + ghi xuống| 11 partition| thêm 1 task repartition+  chia làm 24 partiton sẽ tối ưu hiệu năng tính toán và ghi.
|**time**| 15s |0.1s + 4s|


cấu hình tối ưu 

```
spark_conf.set('spark.sql.adaptive.coalescePartitions.initialPartitionNum', 24)   

# To decrease the number of partitions resulting from shuffle operations
spark_conf.set('spark.sql.adaptive.coalescePartitions.parallelismFirst', 'false')

spark_conf.set('spark.sql.files.minPartitionNum', 1)
spark_conf.set('spark.sql.files.maxPartitionBytes', '500mb')

# partition trước khi write theo partitionBy
df = df.repartition(24, 'compared_to_avg')
```

# Spark flat map

Sau khi flatMap thì số lượng partition khác nhưng mà số hàng sẽ tăng lên. nên repartition sau khi flatMap 

# Spark spills 

If spark.shuffle.spill is true(which is the default). tránh OOM vì đôi khi reduce task(groupBy) quá lớn -> spill: lưu data xuống disk. điều này vô tình chung làm tăng áp lực lên bộ nhớ, I/O disk, GC.

Log

```
INFO ExternalSorter: Task 1 force spilling in-memory map to disk it will release 232.1 MB memory
```

Cách xử lí: 
- select ít data thôi, hoặc tăng số partition lên
- tăng shuffle buffer: ```spark.executor.memory```, hoặc nếu không đủ mem để tăng thì tăng size ```spark.shuffle.file.buffer```, để hạn chế  chàn buffer, giảm I/O 


# Spark unbalance partition

Nếu chia partition không tốt có thể dẫn đến oom, thời gian các task chạy không đều (khác biệt lớn giữa min task time và max task time). thực ra đây không phải vấn đề của spark mà là về phân bố dữ liệu.

Often data is partitioned based on a key, such as day of week, country, etc. If the values for that key are not evenly distributed, then one partition will contain more data than another.

Cách giải quyết 
- Broadcast the smaller dataframe if possible  ```df = transactions.join(broadcast(countries), 'country')```
- Redistribute,  or simply increasing the number of partitions
- Thêm salt, join trên 2 cột. sau đó xóa cột đó đi
- Differential replication
- Iterative broadcast join


# Một số tối ưu khác 

Qua mỗi bước cần check xem đang có bn partition, nếu ít quá phải tiến hành repartition lại 

PartitionBy(col). thì khi load lên, nếu where(col=val), thì nó chỉ cần load đúng cái file đó lên mà không load hết tất cả. Vấn đề: ghi bằng 1 cụm lớn đọc bằng 1 cụm nhỏ. Mặc khác PartitionBy cũng là một phép gom nhóm nhưng nó k yêu cầu shuffle, đổi lại, nó có thể sinh ra số lượng file rất lớn (gây **small file problems** của HDFS)



# Ref

[salesforce](https://engineering.salesforce.com/how-to-optimize-your-apache-spark-application-with-partitions-257f2c1bb414/)

[luminousmen](https://luminousmen.com/post/spark-tips-partition-tuning)

[tối ưu partition tools](https://sparkconfigoptimizer.com/)

[stackoverflow](https://stackoverflow.com/questions/24622108/apache-spark-the-number-of-cores-vs-the-number-of-executors)

[cloudera](https://blog.cloudera.com/how-to-tune-your-apache-spark-jobs-part-1/)

[chú ý khi lưu, đọc hdfs: size, số lượng parttion, partitionBy](https://towardsdatascience.com/optimizing-output-file-size-in-apache-spark-5ce28784934c)








