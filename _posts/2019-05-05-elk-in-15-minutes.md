---
layout: post
title: "Xây dựng hệ thống ELK trong 15 phút"
date: 2019-05-05 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- elk
- logstash
- elastic
- elasticsearch
- kibana
excerpt: "Xây dựng hệ thống Elastic Search, Logstash và Kibana sử dụng Docker, Docker-compose."

---

Sau đây tôi trình bày một cách đơn giản để dựng 1 hệ thống ELK. Tất cả mã nguồn của ví dụ, bạn có thể tải trên [Github](https://github.com/ngvlongit1/elk của tôi.

# ELK là gì?

ELK là tập hợp 3 phần mềm được phát triển bởi [Elastic](https://www.elastic.co/) phục vụ bài toán thu thập, lưu trữ, phân tích dữ liệu (thường được dùng trong các hệ thống Logging). Ba phần mềm bao gồm:

* [Logstash](https://www.elastic.co/products/logstash): Thu thập, xử lý dữ liệu
* [Elastic Search](https://www.elastic.co/products/elasticsearch): Lưu trữ, tìm kiếm dữ liệu
* [Kibana](https://www.elastic.co/products/kibana): Cung cấp giao diện tương tác với Elastic Search, tìm kiếm, mô hình hóa dữ liệu một cách trực quan

# Chuẩn bị

Tôi dựng hệ thống này trên [Debian 9](https://www.debian.org/distrib/). Các ứng dụng chạy dưới dạng [Docker Container](https://www.docker.com/) bằng [Docker Compose](https://docs.docker.com/compose/)

Chạy script dưới đây để cài đặt các thành phần cần thiết:
```bash
apt-get update
apt-get install -y \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg2 \
    software-properties-common
curl -fsSL https://download.docker.com/linux/debian/gpg | apt-key add -
add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/debian \
   $(lsb_release -cs) \
   stable"
apt-get update
apt-get install -y docker-ce docker-ce-cli containerd.io
curl -L "https://github.com/docker/compose/releases/download/1.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
```

Tôi sử dụng Logstash, Elastic Search, Kibana phiên bản 5.6. Đoạn lệnh sau giúp tôi tải các image đó về:
```bash
docker pull logstash:5.6
docker pull elasticsearch:5.6
docker pull kibana:5.6
```

# Cài đặt

Tôi tạo file docker-compose.yml
```bash
version: '2.1'

services:
    logstash:
        extends:
            file: common.yml
            service: common
        container_name: logstash
        image: logstash:5.6
        volumes:
            - ./etc/logstash/conf.d:/etc/logstash/conf.d
        command: -f /etc/logstash/conf.d -r
        environment:
            LS_JAVA_OPTS: "-Xmx256m -Xms256m"
            XPACK_MONITORING_ENABLED: "false"
        ports:
            - '514:10514/udp'
        mem_limit: 512m

    elasticsearch:
        extends:
            file: common.yml
            service: common
        container_name: elasticsearch
        image: elasticsearch:5.6
        volumes:
            - ./data:/usr/share/elasticsearch/data
        environment:
            ES_JAVA_OPTS: "-Xms512m -Xmx512m"
        ports:
            - '9200:9200'
        mem_limit: 1g

    kibana:
        extends:
            file: common.yml
            service: common
        container_name: kibana
        image: kibana:5.6
        environment:
            ELASTICSEARCH_HOSTS: http://elasticsearch:9200
            XPACK_MONITORING_ENABLED: "false"
        ports:
            - "80:5601"    
```

Một số cấu hình chung, tôi để trong file common.yml
```bash
version: '2.1'
services:
    common:
        restart: always
        ulimits:
            memlock:
                soft: -1
                hard: -1
        logging:
            driver: json-file
            options:
                max-size: "5m"
                max-file: "2"
```

Chạy lệnh ```docker-compose up -d ``` để bắt đầu các dịch vụ. 

Bây giờ, tôi sẽ cấu hình đẩy log vào logstash. Để đơn giản tôi đẩy luôn log của server debian vào logstash qua giao thức syslog.
```bash
echo *.info @127.0.0.1 >> /etc/rsyslog.conf
service rsyslog restart
```

Truy cập vào địa chỉ http://127.0.0.1 để bắt đầu sử dụng Kibana cho việc tìm kiếm, phân tích log.
Trước tiên, tôi tạo ```index pattern``` bằng cách vào phần Management và nhấn Create:

![Kibana]( {{site.url}}/assets/img/2019/05/05/kibana_management.png) 

Sau đó, cấu hình để index pattern vừa rồi thành index mặc định

![Set as default index]( {{site.url}}/assets/img/2019/05/05/kibana_set_default_index.png) 

Xong rồi, bây giờ vào phần Discover chúng ta sẽ thấy dữ liệu được hiển thị

![Set as default index]( {{site.url}}/assets/img/2019/05/05/kibana_discover.png) 





