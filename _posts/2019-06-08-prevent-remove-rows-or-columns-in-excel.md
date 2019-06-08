---
layout: post
title: "Ngăn chặn người khác xoá hàng/cột trong Excel"
date: 2019-06-08 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Excel
tags:
- excel
- office
excerpt: "Đã bao giờ bạn gửi một biểu mẫu Excel cho đồng nghiệp và bảo họ điền thông tin vào đó, những chỗ không có thông tin thì để trống nhưng đến lúc nhận lại thì đồng nghiệp tốt bụng của bạn đã xoá tất cả những hàng/cột mà họ nghĩ là thừa đó. Bài này, tôi giới thiệu cách để bạn hạn chế việc đó"
---

## Tình huống giả định

Bạn đang có 1 bảng tính Excel cần tổng hợp kết quả, phải gửi đi cho nhiều người điền thông tin. Mong muốn của bạn là họ chỉ điền thông tin của họ vào đó, những dòng/cột không có thì để nguyên (để bạn paste vào và dùng công thức cho dễ chẳng hạn). Tuy nhiên, khi nhận lại kết quả thì những phần thừa (theo họ) đã được làm sạch. Ahihi, cuộc sống mà.

## Cách giải quyết

Tôi thực hiện việc không cho người khác xoá 1 dòng bất kỳ trong vùng có dữ liệu. Với cột bạn làm tương tự.

Chi tiết về cách làm và kết quả mời bạn xem hình dưới đây:

![Excel]( {{site.url}}/assets/img/2019/06/08/Excel.gif)

Chắc chắn khi đồng nghiệp của bạn xoá 1 dòng bất kì trong bảng dữ liệu của bạn sẽ bất ngờ mà kêu lên "Bí ẩn .... (à mà thôi)"
