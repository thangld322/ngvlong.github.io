---
layout: post
title: "Tránh việc đầy ổ cứng khi chạy Docker"
date: 2019-05-25 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- docker
- logging
excerpt: "Tôi chạy ứng dụng bằng docker container và phát hiện ra rằng lâu lâu ổ cứng lại bị đầy, điều mà trước khi chạy trong docker tôi chưa bao giờ gặp. Vì quả thật, ứng dụng của tôi có ghi gì ra đĩa đâu"
---

Tôi chạy ứng dụng bằng docker container và phát hiện ra rằng lâu lâu ổ cứng lại bị đầy, điều mà trước khi chạy trong docker tôi chưa bao giờ gặp. Vì quả thật, ứng dụng của tôi có ghi gì ra đĩa đâu. Sau một quá trình xem xét cẩn thận, tôi rút ra kết luận là do việc ghi log của Docker. Tôi đã giải quyết bằng cách thêm đoạn cấu hình sau vào phần cấu hình của service trong file ```docker-compose.yml```

```bash
        logging:
            driver: json-file
            options:
                max-size: "10m"
                max-file: "10"
```

Đoạn cấu hình trên nghĩa là tôi chỉ cho phép container đó ghi log ra tối đa 10 file, mỗi file tối đa 10MB.
