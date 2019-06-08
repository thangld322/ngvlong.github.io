---
layout: post
title: "Dựng và sử dụng Kafka Cluster"
date: 2019-06-06 12:00:00.000000000 +07:00
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
excerpt: "Bài này tôi trình bày về việc dựng Kafka Cluster và một số lệnh thường dùng trong quá trình sử dụng Kafka"
---

## Kafka là gì

Kafka là một nền tảng Stream dữ liệu phân tán. Mọi chi tiết các bạn có thể xem cụ thể [ở đây](https://kafka.apache.org/).
Tôi thường sử dụng Kafka làm Message Queue cho một số hệ thống lớn, nhiều ứng dụng ghi vào, nhiều ứng dụng khác nhau đọc ra. Điểm mạnh của Kafka là scale dễ dàng.

## Chạy Cluster Kafka với Docker

Vẫn như thường lệ tôi vẫn sẽ dùng docker, docker-compose thôi. Và đây là file docker-compose.yml của tôi:

```bash
version: '2.1'
services:
    zoo1:
        image: zookeeper:3.4.14
        restart: always
        hostname: zoo1
        container_name: zoo1
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=0.0.0.0:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888

    zoo2:
        image: zookeeper:3.4.14
        restart: always
        hostname: zoo2
        container_name: zoo2
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=0.0.0.0:2888:3888 server.3=zoo3:2888:3888

    zoo3:
        image: zookeeper:3.4.14
        restart: always
        hostname: zoo3
        container_name: zoo3
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=0.0.0.0:2888:3888

    kafka01:
        image: wurstmeister/kafka:2.11-1.1.1
        hostname: kafka01
        container_name: kafka01
        environment:
            KAFKA_BROKER_ID: 1
            KAFKA_ADVERTISED_HOST_NAME: kafka01
            KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
            KAFKA_NUM_PARTITIONS: 3
            KAFKA_LOG_RETENTION_HOURS: 1

    kafka02:
        image: wurstmeister/kafka:2.11-1.1.1
        hostname: kafka02
        container_name: kafka02
        environment:
            KAFKA_BROKER_ID: 2
            KAFKA_ADVERTISED_HOST_NAME: kafka02
            KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
            KAFKA_NUM_PARTITIONS: 3
            KAFKA_LOG_RETENTION_HOURS: 1

    kafka03:
        image: wurstmeister/kafka:2.11-1.1.1
        hostname: kafka03
        container_name: kafka03
        environment:
            KAFKA_BROKER_ID: 3
            KAFKA_ADVERTISED_HOST_NAME: kafka03
            KAFKA_ZOOKEEPER_CONNECT: zoo1:2181,zoo2:2181,zoo3:2181
            KAFKA_NUM_PARTITIONS: 3
            KAFKA_LOG_RETENTION_HOURS: 1
```

Ở đây tôi tạo 1 cluster 3 Node Kafka, 1 cluster 3 Node [Zookeeper](https://zookeeper.apache.org/). Chắc bạn sẽ thắc mắc Zookeeper dùng để làm gì. Zookeeper là 1 dịch vụ trung tâm cho việc duy trì thông tin cấu hình, đặt tên, cung cấp sự đồng bộ phân tán, cung cấp dịch vụ nhóm. Zookeeper là một [mã nguồn mở](https://github.com/apache/zookeeper) viết bằng Java.
Bây giờ, tôi chạy Cluster này lên bằng lệnh ```docker-compose up -d```

> Trong các bài viết trước, tôi thường dùng lệnh ```docker pull``` để tải image về. Thực ra thì lý do là tôi cẩn thận tải trước cho yên tâm, chứ bình thường khi chạy ```docker-compose up``` nếu image đó chưa có nó sẽ được tải về. Vừa rồi, nhà mạng Viettel mới tăng gấp đôi băng thông (đương nhiên là giá không đổi) cho tôi, giờ đây tôi được sử dụng một đường truyền có băng thông siêu to, khổng lồ rồi nên việc này cũng không cần thiết nữa.

## Một số khái niệm cơ bản khi sử dụng Kafka

Khi sử dụng Kafka, có 1 số khái niệm cơ bản cần biết như sau:

* Topic: Có thể hiểu là một nơi trung gian lưu trữ thông điệp, ứng dụng của bạn có thể đọc/ghi từ đây

* Partition: Mỗi Topic có thể được chia thành nhiều partition (như ví dụ trên của tôi là 3). Mỗi Partition lưu trữ các thông điệp theo một thứ tự không đổi.

* Producer: Ứng dụng ghi dữ liệu và Kafka

* Consumer: Ứng dụng đọc dữ liệu từ kafka. Nếu bạn có nhiều Consumer cùng đọc vào 1 topic trong kafka, thì chúng sẽ phân biệt với nhau bởi group_id. Các ứng dụng khác nhau phải có group_id khác nhau, ngược lại các Consumer của cùng 1 ứng dụng (xử lý song song, phân tán) thì phải có group_id giống nhau.

## Một số câu lệnh thường dùng

Dưới đây là 1 số lệnh tôi hay phải dùng khi làm việc với kafka. Vì tôi sử dụng Docker rồi nên các lệnh dưới được hiểu là chạy trong Container, tức là trước khi gõ các lệnh bên dưới tôi phải gõ lệnh ```docker exec -it kafka01 bash``` (có thể thay kafka01 bằng container name trên hệ thống của bạn)

### Tạo 1 topic

```/opt/kafka/bin/kafka-topics.sh --zookeeper zoo1:2181 --create --topic topic_test --partitions 6 --replication-factor 2```
> Câu lệnh trên sẽ tạo topic topic_test với 6 partitions và có 2 bản sao. Chi tiết thêm về các tham số cài đặt của topic bạn có thể xem thêm [ở đây](https://kafka.apache.org/documentation/#topicconfigs)

### Liệt kê danh sách các topic

```/opt/kafka/bin/kafka-topics.sh --zookeeper zoo1:2181 --list```

### Xem thông tin chi tiết 1 topic

```/opt/kafka/bin/kafka-topics.sh --zookeeper zoo1:2181 --describe --topic <topic>```

### Xoá 1 topic

```/opt/kafka/bin/kafka-topics.sh --zookeeper zoo1:2181 --delete --topic <topic>```
> Việc xoá topic sẽ chỉ được thực hiện nếu trong kafka có cấu hình ```delete.topic.enable=true```

### Thay đổi số Partitions của Topic

```/opt/kafka/bin/kafka-topics.sh --zookeeper zoo1:2181 --topic <topic> --partitions 30```

> Có 1 lưu ý là bạn không thể giảm số partition của 1 topic trong kafka. Nên thường thì tôi sẽ cấu hình số partition mặc định ít rồi sau đó dùng lệnh trên để tăng lên nếu cần.

### Xem danh sách các group consumer

```/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server kafka01:9092 --list```

### Xem chi tiết 1 group

```/opt/kafka/bin/kafka-consumer-groups.sh --bootstrap-server kafka01:9092 --describe --group <group>```

> Lệnh này rất hữu ích trong việc xem việc xử lý của Consumer có kịp so với tốc độ ghi vào của Producer không thông quá giá trị LAG.
