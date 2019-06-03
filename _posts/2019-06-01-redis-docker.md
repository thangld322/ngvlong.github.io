---
layout: post
title: "Một số chú ý khi sử dụng Redis làm Message Queue"
date: 2019-06-01 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Docker
tags:
- docker
- logging
excerpt: "Một vài kinh nghiệm của tôi trong quá trình sử dụng Redis làm Message Queue. Hi vọng sẽ giúp ích cho bạn"
---

Trong quá trình sử dụng Redis làm Message Queue, thi thoảng hệ thống có phát sinh một vài lỗi, và tôi đã chỉnh sửa một số cấu hình để hệ thống chạy được cho các lần tiếp theo.

## Redis là gì

Ahihi, Redis là một hệ thống lưu trữ dạng Key-Value trên Memory, thường được sử dụng làm Message Queue để làm hàng chờ cho các thành phần khác xử lý. Chi tiết thêm bạn có thể xem [ở đây](https://redis.io/)

## Tôi cấu hình Redis như thế nào

Như thường lệ tôi vẫn sẽ dùng docker, docker-compose thôi. Và đây là file docker-compose.yml của tôi:

```bash
version: '2.1'
services:
    redis:
        container_name: redis
        image: redis:5.0.5
        command: redis-server --requirepass Meomeo@123 --save "" --maxclients 10000 --timeout 300
        ports:
            - "6379:6379"
        restart: always
        ulimits:
            memlock:
                soft: -1
                hard: -1
            nproc: 65535
            nofile:
                soft: 20000
                hard: 20000
        logging:
            driver: json-file
            options:
                max-size: "5m"
                max-file: "2"
        mem_limit: 512m
        swap_limit: 512m
```

Trong cấu hình trên, có 1 số chú ý như sau:

* maxclients: Số client tối đa được kết nối vào redis của bạn. Bạn có thể cấu hình số này to lên (nhưng hãy để ý đến tham số max_open_file trên server)

* timeout: Số giây mà nếu trong khoảng thời gian đó client không có hành động gì thì nó sẽ không được giữ kết nối với redis server. Nếu không cấu hình tham số này, mặc định client sẽ giữ kết nối với server và khi mà số client vượt quá maxclients thì bạn sẽ gặp lỗi ```ERR max number of clients reached```

* mem_litmit: Một tham số quan trọng nữa là RAM, bạn phải giới hạn RAM cho Redis đề phòng ứng dụng của bạn không xử lý kịp. Tuy nhiên nếu cấu hình tham số này, nếu ứng dụng không xử lý kịp thì phần ghi vào sẽ bị lỗi ```OOM command not allowed when used memory > 'maxmemory'```. Để tránh trường hợp này thì có nhiều cách, cách tốt nhất là đảm bảo ứng dụng của bạn xử lý không để tồn queue trong redis (đương nhiên). Tuy nhiên, nếu việc đó không quá quan trọng thì tôi cấu hình giới hạn RAM bằng docker-compose như trên. Nếu đầy RAM, Redis sẽ khởi động lại và mọi thứ lại như mới.

* Còn một lỗi hay gặp nữa là khi bạn có tuỳ chọn ```save```. Thường thì liên quan đến việc không có quyền ghi vào file dump.rdb. Tôi dùng redis trên làm queue trên mem và dữ liệu cũng không quan trọng lắm nên tôi không sử dụng tuỳ chọn lưu xuống đĩa làm gì cả.
