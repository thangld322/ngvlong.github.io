---
layout: post
title: "Thu thập Log với Filebeat"
date: 2019-05-10 12:00:00.000000000 +07:00
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
- filebeat
- beats
excerpt: "Thu thập Log của các ứng dụng với Filebeat."
---

Ở [bài trước](../tutorials/elk-in-15-minutes.html) tôi đã trình bày cách dựng 1 hệ thống ELK có phần thu thập log thông qua giao thức Syslog. Bài này tôi trình bày thêm 1 cách thu thập log các ứng dụng ghi log ra dạng file bằng filebeat.

# Giới thiệu Filebeat

[Filebeat](https://www.elastic.co/products/beats/filebeat) là một phần mềm mã nguồn mở trong họ [Beats](https://www.elastic.co/products/beats/) được phát triển bởi [Elastic](https://www.elastic.co/). Filebeat được sử dụng làm Agent thu thập Log, mặc dù hỗ trợ rất nhiều đầu vào khác nhau (Syslog, Redis, Log ...) nhưng Filebeat được sử dụng chủ yếu trong việc thu thập log của các ứng dụng, hệ thống dưới dạng file.

# Cài đặt

Tôi dùng docker trong hệ thống của mình, nếu bạn không thích dùng docker thì có thể tham khảo cách cài đặt [ở đây](https://www.elastic.co/guide/en/beats/filebeat/current/filebeat-installation.html).
Nếu dùng docker như tôi, hãy chạy lệnh sau để tải filebeat về:

```bash
docker pull docker.elastic.co/beats/filebeat:5.6.16
```

Bây giờ tôi sẽ tạo file docker-compose-filebeat.yml để chạy filebeat.

```bash
version: '2.1'

services:
    filebeat:
        extends:
            file: common.yml
            service: common
        container_name: filebeat
        image: docker.elastic.co/beats/filebeat:5.6.16
        volumes:
            - ./etc/filebeat/filebeat.yml:/usr/share/filebeat/filebeat.yml
            - /var/log/:/opt/filebeat/log/
        network_mode: host
        user: root
```

Tôi mount thư mục /var/log vào thư mục /opt/filebeat/log để lấy toàn bộ log trong /var/log trên máy. Để chắc chắn có quyền đọc tôi chay filebeat trong docker với user root. Với ứng dụng của bạn, bạn nên thay đổi 2 tham số này.

Sau đây là file config (được mount vào /usr/share/filebeat/filebeat.yml trong container) để filebeat đọc các file trong /opt/filebeat/log/ (tương đương với /var/log trên host) sau đó đẩy về logstash.

```bash
filebeat.prospectors:

- input_type: log
  paths:
    - /opt/filebeat/log/*.log

output.logstash:
  hosts: ["localhost:5044"]
```

Để có thể nhận được log này và đẩy vào Elastic Search, tôi cấu hình thêm 1 input trong logstash:

```bash
input {
    beats {
        port => 5044
    }
}
```

Vậy là phần cấu hình đã xong. Bây giờ tôi chạy ```docker-compose up -d``` để chạy lại hệ thống ELK đã dựng trước, sau đó chạy ```docker-compose -f docker-compose-filebeat.yml up -d``` để filebeat đẩy log về.

Vào lại Kibana, kết quả log đẩy về đã hiển thị trên màn hình

![Kibana filebeat]( {{site.url}}/assets/img/2019/05/12/kibana_filebeat.png) 

# Kết luận

Tôi đã trình bày xong cách sử dụng filebeat lấy log dạng file của 1 ứng dụng đẩy về hệ thống ELK. Mã nguồn bạn có thể lấy tại [Github](https://github.com/ngvlongit1/elk) của tôi.