1. chết executor 

- lỗi trên driver

```
ERROR YarnScheduler: Lost executor 1 on hd3728:
Container exited with a non-zero exit code 52. Error file: prelaunch.err.  

```

- lỗi trên executor

```
java.lang.OutOfMemoryError: unable to create new native thread
```

- theo dõi trên checkmk: lượng thread tăng và giảm khi có dead executor 

- lí do: khơỉ tạo connect es không đóng dẫn tới mỗi lần chạy đều mở nhiều connnection

- chú ý: có thể chết cả node manager

2. show log executor






