---
layout: post
title: "Phân tích log iptables với plugin logstash-filter-kv "
date: 2019-05-20 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Logstash
tags:
- logstash
- filter
- plugin
excerpt: "Khi cài đặt một hệ thống, đặc biệt các hệ thống ngoài Internet, tôi luôn phải cấu hình iptables cẩn thận để hạn chế các kết nối ngoài ý muốn. Đôi khi xem log này, tôi thấy có rất nhiều kết nối bị chặn nhưng để phân tích thì thật khó khăn. Vì vậy, tôi cấu hình đẩy log này về Logstash."
---

Bài này, tôi trình bày về việc phân tích log iptables sử dụng plugin logstash-filter-kv.

## Giới thiệu Plugin logstash-filter-kv

Đây là plugin để parse các message có dạng key=value. Chi tiết về cách dùng, các tham số bạn có thể tham khảo thêm [ở đây](https://www.elastic.co/guide/en/logstash/current/plugins-filters-kv.html#plugins-filters-kv)

## Cấu hình đẩy log iptables về Logstash

Trước tiên, tôi thêm luật iptables để ghi log các kết nối sẽ bị DROP

```bash
iptables -A INPUT -j LOG --log-level info --log-prefix "CHAIN=INPUT STATUS=DROP "
iptables -A INPUT -j DROP
```

Bạn cũng có thể cấu hình tương tự với các chain OUTPUT, FORWARD ... để lấy log tương ứng
> Mặc định, log sẽ được ghi ra file, việc này có thể gây ra tình trạng đầy ổ cứng Server của bạn.

Bây giờ gõ lệnh ```dmesg -T```, tôi thấy đã có log DROP gói tin

![Iptables DROP]( {{site.url}}/assets/img/2019/05/20/Iptables_DROP.png)

Để log này đẩy về Logstash, tôi chạy lệnh sau

```bash
echo kern.info @127.0.0.1 >> /etc/rsyslog.conf
service rsyslog restart
```

> Bạn có thể thay 127.0.0.1 bằng địa chỉ Server Logstash dùng để nhận log của bạn. Với cấu hình trên, log sẽ được đẩy về 127.0.0.1:514/UDP

## Sử dụng Kv trong Logstash

Thêm đoạn cấu hình vào phần filter trong file cấu hình logstash
```bash
    kv {
        
    }
```

Truy cập vào Kibana, ta sẽ thấy log được phân tích ra các trường với giá trị tương ứng.

![Iptables Logs]( {{site.url}}/assets/img/2019/05/20/Iptables_Logs.png)

Xem chi tiết 1 bản ghi, kết quả như sau

![Iptables Detail]( {{site.url}}/assets/img/2019/05/20/Iptables_Json.png)







