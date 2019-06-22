---
layout: post
title: "Hướng dẫn sử dụng giao diện Cerebro để quản lý Elasticsearch Cluster"
date: 2019-06-22 12:00:00.000000000 +07:00
author: "Thang Le"
type: post
published: true
status: publish
categories: 
- Guide
tags:
- cerebro
- elastic
- elasticsearch
- docker
- docker-compose
excerpt: "Sử dụng Cerebro để quản lý Elasticsearch Cluster"

---

## Elasticsearch là gì

Elasticsearch là một trong ba phần mềm được phát triển bởi [Elastic](https://www.elastic.co/) phục vụ bài toán thu thập, lưu trữ, phân tích dữ liệu (thường được dùng trong các hệ thống Logging). Ba phần mềm bao gồm:

* [Logstash](https://www.elastic.co/products/logstash): Thu thập, xử lý dữ liệu
* [Elastic Search](https://www.elastic.co/products/elasticsearch): Lưu trữ, tìm kiếm dữ liệu
* [Kibana](https://www.elastic.co/products/kibana): Cung cấp giao diện tương tác với Elastic Search, tìm kiếm, mô hình hóa dữ liệu một cách trực quan

Elasticsearch là một database hiện đại, có search engine mạnh mẽ, thân thiện với nhà phát triển phần mềm, với cấu hình tối thiểu và được quản lý thủ công.
Elasticsearch cung cấp một hệ thống API đầy đủ được gọi là elasticsearch APIs để nhà phát triển phần mềm có thể dễ dàng tương tác với ES cluster để tìm kiếm thông tin, cũng như quản lý ES cluster.

Dưới đây là công cụ Cerebro, một GUI tương tác với elasticsearch APIs mà tôi ưa thích để có cái nhìn tổng quan về ES cluster.

## Cài đặt

Mã nguồn của Cerebro [Github](https://github.com/lmenezes/cerebro).

Chạy script dưới đây để cài đặt và sử dụng Cerebro:

```bash
    # Java 1.8 or newer is required. brew cask install java
    # Download the latest tarball from https://github.com/lmenezes/cerebro/releases/latest
    wget https://github.com/lmenezes/cerebro/releases/download/v0.8.1/cerebro-0.8.1.tgz
    tar zxvf cerebro-0.8.1.tgz
    cd cerebro-0.8.1
    ./bin/cerebro
    # Open cerebro with http://localhost:9000
    # If you're using docker version, access ES via host.docker.internal:9200
```

## Cài đặt bằng docker-compose

Tôi tạo file docker-compose.yml

```bash
version: '2.1'

services:
    cerebro
```

## Sử dụng Cerebro

Tôi thường sử dụng Cerebro để quan sát sự phân bổ Index Shards giữa các node trong cluster. Nó rất rõ ràng và trực quan.

```
CHÈN ẢNH!
```

Ngoài ra, Cerebro cũng cung cấp đầy đủ giao diện để tương tác với các APIs khác của Elasticsearch như Cluster settings, Nodes settings, Index settings, Index mappings, Index templates, Aliases, cat APIs...

Trên thanh menu chính của Cerebro, từ trái qua phải:

* Đầu tiên là `overview`, ở tab này sẽ có danh sách các nodes và các index. Đây là trang chính của Cerebro và thường được sử dụng để quan sát sự phân bổ của Index Shards giữa các node. Ngoài ra chúng ta cũng có thể sử dụng nó để xem cài đặt của nodes, indices, mappings của indices... hay các thao tác cơ bản như là tìm kiếm index, xóa index, di chuyển shards giữa các node...
* Tab thứ hai là `nodes`, ở tab này chúng ta sẽ thấy được danh sách các nodes cùng thông tin của nodes là loại node (gồm có: data, master, ingest, coordinate), heap, disk, uptime.
* Tiếp theo là `rest` (RESTful APIs), Cerebro cung cấp cho người dùng một giao diện đơn giản để tương tác trực tiếp với Elasticsearch APIs. Danh sách các APIs được Elasticsearch hộ trở ở đây: [Github](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs.html).
* Cuối cùng là `more`. Tab này cung cấp thêm cho người dùng một số giao diện để tạo index mới, xem/sửa cài đặt cluster, aliases, analysis, thêm/sửa/xóa index templates, repositories, snapshot, cat apis.

```
PHẦN NÀY BỔ SUNG SAU.
```