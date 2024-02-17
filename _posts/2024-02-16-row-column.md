---
layout: post
title: Rows vs Columnar Databases
tags:
  - bigdata
---

lí do sinh ra colume db: kích thước dữ liệu

## 2 cách lưu trữ dữ liệu khác nhau

- lưu trữ dưới dạng row: 

```
001: BMW, 840i, 2020, 335
002: Porsche, 911, 2019, 443
003: Mercedes-Benz, G-Wagon, 2019, 577 
```

- lưu trữ dưới dạng column: 

```
BMW:001, Porsche:002, Mercedes-Benz:003
840i:001, 911:002, G-Wagon:003
2020:001, 2019:002, 2019:003
335:001, 443:002, 577:003
```

## nhận xét


|row| column| 
|-|-|
|khi cần tìm thông tin của xe 003 thì cần duyệt các dòng tới khi gặp row có id = 003 thì lấy ra thông tin| khi cần tìm thông tin của xe 003 ví dụ như năm sản xuất. ta tìm tới hàng ```năm sản xuất```, sau đó duyệt hàng đó cho tới khi gặp giá trị 003 thì lấy được thông tin năm sản xuất của xe 003 |
|khi đọc thì các thông tin của xe 003 là các ô nhớ liên tiếp nhau -> locality. đọc thông tin của 1 bản ghi nhẹ hơn||
|khi tính toán trên cột (lấy tất cả các giá trị trên cột ví dụ như tính trung bình) đồng nghĩa phải load cả row để lấy ra cột đó (mỗi cột là một giá trị của row) | khắc phục được hạn chế này bằng cách chỉ lấy ra giữ liệu của cột đó để tính toán(thay vì toàn bộ row)|

## size compression 

- một ưu điểm khác của colume database là khả năng compression do khái niệm data alignment (các ngôn ngữ như C++, golang đều có khái niệm này)
- 1 word là một đơn vị nhỏ nhất để load lên mem xử lí, ví dụ os 32-bit 1 word = 32bit = 4 byte
- thứ tự sắp xếp của các field trong một obj sẽ liên quan tới sự sắp xếp các field trong mem 
- nếu một thuộc tính có dung lượng lớn hơn 1 word thì nó sẽ được chuyển sang word tiếp theo. cơ chế là sẽ k load dữ liệu trên 2 word

```
struct Option1  
{ 
    bool item1; // 1 byte
    int item2;  // 4 bytes
    int item3;  // 4 bytes
    bool item4; // 1 bytes
}; 

word 1: 1byte(bool của item 1) + 3byte(padding-null)
word 2: 4byte(int của item 2)
word 2: 4byte(int của item 3)
word 4: 1byte(bool của item 4) + 3byte(padding-null)

```

- nếu sắp xếp lại thứ tự: tiết kiệm được 4 byte

```
struct Option1  
{ 
    bool item1; // 1 byte
    bool item4; // 1 byte
    int item2;  // 4 bytes
    int item3;  // 4 bytes
}; 

word 1: 1byte(bool của item 1) + 1byte(bool của item 1) + 2byte(padding-null)
word 2: 4byte(int của item 3)
word 2: 4byte(int của item 4)
word 4: free

```

- với row db, giả sử 1 row có dung lượng là lẻ số word thì nó sẽ bị padding. nhiều row dẫn đến dư thừa dữ liệu. tuy nhiên với cloumn db, vì các giá trị của 1 cột có cùng kiểu dữ liệu nên sẽ k tốn không gian dư. 
- hiểu đơn giản là lưu trữ theo cột -> cùng kiểu dữ liệu nên dễ compress hơn. nếu lưu theo hàng, các kiểu dữ liệu khác nhau sẽ khó nén hơn
