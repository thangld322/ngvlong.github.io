---
layout: post
title: "Cấu hình HTTPS cho Nginx"
date: 2019-05-12 12:00:00.000000000 +07:00
author: "Long Tom"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- https
- http
- openssl
- nginx
excerpt: "Sử dụng OpenSSL tạo cert để cấu hình HTTPS cho Nginx."
---

Sau đây tôi giới thiệu cách sử dụng OpenSSL tự tạo cert để cấu hình HTTPS cho Nginx.

## Cài đặt

Như thường lệ, tôi vẫn sử dụng Debian làm hệ điều hành cho hệ thống của mình, Nginx thì tôi sử dụng Docker. Trước tiên tôi tải image nginx về bằng lệnh

```bash
docker pull nginx:1.14.0
```

Tạo file config cho nginx (Nội dung file bạn có thể tham khảo [ở đây](https://mozilla.github.io/server-side-tls/ssl-config-generator/))

```bash
server {
    listen 80 default_server;
    listen [::]:80 default_server;
    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /path/to/dhparam.pem;

    # intermediate configuration. tweak to your needs.
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256:ECDHE-ECDSA-AES128-SHA:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA:ECDHE-ECDSA-AES256-SHA384:ECDHE-ECDSA-AES256-SHA:ECDHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA256:DHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA256:DHE-RSA-AES256-SHA:ECDHE-ECDSA-DES-CBC3-SHA:ECDHE-RSA-DES-CBC3-SHA:EDH-RSA-DES-CBC3-SHA:AES128-GCM-SHA256:AES256-GCM-SHA384:AES128-SHA256:AES256-SHA256:AES128-SHA:AES256-SHA:DES-CBC3-SHA:!DSS';
    ssl_prefer_server_ciphers on;

    # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
    add_header Strict-Transport-Security max-age=15768000;

    # OCSP Stapling ---
    # fetch OCSP records from URL in ssl_certificate and cache them
    ssl_stapling on;
    ssl_stapling_verify on;

    ## verify chain of trust of OCSP response using Root CA and Intermediate certs
    # ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;
    server_name ngvlong.com www.ngvlong.com;
    gzip on;
}
```

Ở đây ta thấy có ssl_certificate, ssl_certificate_key, ssl_dhparam, ssl_trusted_certificate cần phải cấu hình. Tôi sử dụng cert tự sinh nên bỏ qua ssl_trusted_certificate và chỉ cấu hình cho 3 file còn lại.

Để sinh cặp cert và key cho server tôi dùng OpenSSL. Mặc dù có thể chạy lệnh

```bash
openssl req -newkey rsa:2048 -nodes -keyout ngvlong.com.key -x509 -days 365 -out ngvlong.com.crt
```

rồi khi terminal hiện lên mấy câu hỏi, chỉ việc trả lời hoặc Enter là xong, nhưng tính tôi không thích bị máy hỏi nhiều nên tôi chạy lệnh

```bash
openssl req -batch -newkey rsa:2048 -nodes -keyout ngvlong.com.key -x509 -days 365 -out ngvlong.com.crt -subj '/CN=ngvlong.com/O=Long Tom/C=VN'
```

Sau khi có cặp cert và key, tôi cần tạo file ssl_dhparam. Sử dụng lệnh sau để tạo nhanh file này

```bash
openssl dhparam -dsaparam -out ngvlong.com.pem 4096
```

Nếu bạn tìm kiếm trên mạng cách tạo file dhparam mà sau đó chạy nửa giờ chưa xong thì hãy xem lại lệnh nhé. Nhớ thêm tham số -dsaparam vào. :D

Vậy là đã có đầy đủ 3 file. Ta sửa file config nginx thành đường dẫn của 3 file trên (nhớ mount 3 file này vào đúng đường dẫn trong container của bạn)

```bash
ssl_certificate /opt/nginx/certs/ngvlong.com.crt;
ssl_certificate_key /opt/nginx/certs/ngvlong.com.key;
ssl_dhparam /opt/nginx/certs/ngvlong.com.pem;
```

Bây giờ, tôi tạo file docker-compose để chạy nginx:

```bash
version: '2.1'

services:
    nginx:
        container_name: nginx
        image: nginx:1.14.0
        volumes:
            - ./etc/certs:/opt/nginx/certs
            - ./etc/conf:/etc/nginx/
        network_mode: host
        user: root
```

Sau khi cấu hình DNS (hoặc sửa file hosts), truy cập vào <http://ngvlong.com> ta sẽ được chuyển về <https://ngvlong.com>. Tất nhiên cert tự sinh nên trình duyệt sẽ cảnh báo.

![Kibana filebeat]( {{site.url}}/assets/img/2019/05/12/https.png)

## Kết luận

Như vậy, tôi đã cấu hình xong https cho nginx với cert tự tạo. Để trình duyệt không cảnh báo (hiển thị màu xanh trông chuyên nghiệp) thì bạn nên mua cert cho website của mình, hoặc cũng có thể chọn giải pháp dùng cert miễn phí từ [Comodo](https://ssl.comodo.com/free-ssl-certificate.php), [SSL For Free](https://www.sslforfree.com/) hoặc [Let's Encrypt](https://letsencrypt.org/).
