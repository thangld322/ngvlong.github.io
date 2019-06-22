---
layout: post
title: "Sử dụng Kafka Reassign Partitions"
date: 2019-06-13 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- kafka
- zookeeper
- docker
excerpt: "Để đáp ứng được lượng dữ liệu đầu vào ngày một nhiều, tôi phải thêm một vài Node Kafka vào Cluster hiện tại, khi đó tôi sử dụng Kafka Reassign Partitions để thêm các Node vào mà không ảnh hưởng đến hệ thống đang chạy"
---

## Giới thiệu

Hệ thống của tôi đang chạy ổn định, tuy nhiên ngày càng nhiều dữ liệu được đẩy vào, Cluster Kafka hiện tại có vẻ không còn đáp ứng được tốc độ đọc/ghi dữ liệu của Producers/Consumers. Để giải quyết việc này, tôi thêm các Node Kafka vào Cluster hiện tại, vấn đề đặt ra là làm thế nào để sau khi thêm việc đọc/ghi được san tải sang các Node mới này? Có một cách là xoá hết các topic đi rồi tạo lại nhưng nó sẽ làm mất dữ liệu hiện thời, các phần chưa được xử lý sẽ mất. Có một cách hay hơn để giải quyết việc này mà không ảnh hưởng gì đến hệ thống đang chạy là sử dụng Kafka Reassign Partitions.

## Thực hiện

Giả sử, tôi đang có 1 Cluster với chỉ 1 Node Kafka có Broker_id là 1 và tôi vừa thêm 2 Node có Broker_id là 2,3. Bây giờ tôi sẽ cần mở rộng topic ```logs``` từ 1 Node sang cả 3 Node (Tất nhiên việc này sẽ chỉ có ý nghĩa nếu số partition của topic ```logs``` không ít hơn 3).
Đầu tiên tôi tạo file ```topics-to-move.json``` với nội dung như sau:

```json
{"topics":
    [
        {"topic": "logs"}
    ],
     "version":1
}
```

Tôi sử dụng ```docker exec``` vào trong container kafka, trong đó không có vim, nano nên để tạo file tôi hay dùng đoạn lệnh tương tự như sau:

```bash
cat <<EOF >> topics-to-move.json
{"topics":
    [
        {"topic": "logs"}
    ],
     "version":1
}
EOF
```

> Nếu bạn muốn mở rộng nhiều topic 1 lúc thì bạn có thể thêm vào mảng topics trong file

Sau khi có file ```topics-to-move.json```, tôi chạy lệnh

```bash
/opt/kafka/bin/kafka-reassign-partitions.sh --zookeeper zoo1:2181 --topics-to-move-json-file topics-to-move.json --broker-list "1,2,3" --generate
```

> Mục tiêu của lệnh này là tạo ra file đề xuất đặt các partition vào các tất cả các brocker trong cluster.

Kết quả của lệnh này như sau:

```bash
Current partition replica assignment
{"version":1,"partitions":[{"topic":"logs","partition":2,"replicas":[1],"log_dirs":["any"]},{"topic":"logs","partition":1,"replicas":[1],"log_dirs":["any"]},{"topic":"logs","partition":0,"replicas":[1],"log_dirs":["any"]}]}

Proposed partition reassignment configuration
{"version":1,"partitions":[{"topic":"logs","partition":2,"replicas":[3],"log_dirs":["any"]},{"topic":"logs","partition":1,"replicas":[2],"log_dirs":["any"]},{"topic":"logs","partition":0,"replicas":[1],"log_dirs":["any"]}]}
```

Lấy thông tin sau phần ```Proposed partition reassignment configuration``` cho vào file ```reassign.json```.

```bash
cat <<EOF >> reassign.json
{
  "version": 1,
  "partitions": [
    {
      "topic": "logs",
      "partition": 2,
      "replicas": [
        3
      ],
      "log_dirs": [
        "any"
      ]
    },
    {
      "topic": "logs",
      "partition": 1,
      "replicas": [
        2
      ],
      "log_dirs": [
        "any"
      ]
    },
    {
      "topic": "logs",
      "partition": 0,
      "replicas": [
        1
      ],
      "log_dirs": [
        "any"
      ]
    }
  ]
}
EOF
```

Sau đó thực hiện lệnh

```bash
/opt/kafka/bin/kafka-reassign-partitions.sh --zookeeper zoo1:2181 --reassignment-json-file reassign.json --execute
```

Các bạn có thể kiểm tra kết quả thực hiện bằng lệnh

```bash
/opt/kafka/bin/kafka-reassign-partitions.sh --zookeeper zoo1:2181 --reassignment-json-file reassign.json --verify
```

> Khi đang thực hiện chuyển các partition từ Node này sang Node khác, CPU và Ổ cứng của bạn sẽ phải hoạt động nhiều hơn. Hãy giám sát chặt chẽ hệ thống cho đến khi việc chuyển các partition được thực hiện xong.

## Kết luận

Trên đây, tôi trình bày cách để mở rộng Cluster Kafka, chuyển các partition của các topic hiện có sang các Node mới được thêm vào. Nếu 1 ngày bạn lại muốn giảm số Broker của Cluster Kafka đi thì bạn cũng làm tương tự nhé.
