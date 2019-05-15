---
layout: post
title: "Hướng dẫn xác thực giữa Filebeat và Logstash với SSL certificates"
date: 2019-05-12 12:00:00.000000000 +07:00
author: "Thang Le"
type: post
published: true
status: publish
categories: 
- Tutorials
tags:
- elk
- logstash
- filebeat
- ssl
- logstash-input-plugin
excerpt: "Xây dựng hệ thống Elastic Search, Logstash và Kibana sử dụng Docker, Docker-compose."

---

Nếu bạn muốn có một logstash instance từ xa có thể kết nối qua internet, bạn cần đảm bảo chỉ những client được phép mới có thể kết nối. Vì giao thức lumberjack không dựa trên HTTP, bạn không thể quay lại proxy thông qua nginx với http xác thực cơ bản và SSL được cấu hình. Thay vào đó, bạn cần thiết lập xác thực của riêng mình dựa trên SSL certificates. Tuy nhiên, [tài liệu chính thức của logstash](https://www.elastic.co/guide/en/beats/filebeat/current/configuring-ssl-logstash.html) không đề cập chi tiết đến việc tạo ra SSL certificates. Do đó, tôi cung cấp hướng dẫn từng bước này để bạn đọc có thể tự tạo một bộ SSL certificates cho riêng mình.

## Cấu hình Logstash input beat (cần 3 files ca.crt,server.crt và server.key).
    input {
        beats {
            port => 5044
            ssl => true
            ssl_certificate_authorities => ["/etc/ca.crt"]
            ssl_certificate => "/etc/server.crt"
            ssl_key => "/etc/server.key"
            ssl_verify_mode => "force_peer"
        }
    }

## Cấu hình Filebeat output (cần 3 files ca.crt, client.crt và client.key).
    output.logstash:
        hosts: ["logs.mycompany.com:5044"]
        ssl.certificate_authorities: ["/etc/ca.crt"]
        ssl.certificate: "/etc/client.crt"
        ssl.key: "/etc/client.key"
        ssl.supported_protocols: "TLSv1.2"

## Bước 1: CA cert (tạo files ca.key và ca.crt)
>
> openssl genrsa -out ca.key 2048
>
> openssl req -x509 -new -nodes -key ca.key -sha256 -days 3650 -out ca.crt

## Bước 2: Logstash server (tạo file server.key và server.crt)
File server.conf

    [req]
    distinguished_name = req_distinguished_name
    req_extensions = v3_req
    prompt = no

    [req_distinguished_name]
    countryName                     = XX
    stateOrProvinceName             = XXXXXX
    localityName                    = XXXXXX
    postalCode                      = XXXXXX
    organizationName                = XXXXXX
    organizationalUnitName          = XXXXXX
    commonName                      = XXXXXX
    emailAddress                    = XXXXXX

    [v3_req]
    keyUsage = keyEncipherment, dataEncipherment
    extendedKeyUsage = serverAuth
    subjectAltName = @alt_names

    [alt_names]
    DNS.1 = DOMAIN_1
    DNS.2 = DOMAIN_2
    DNS.3 = DOMAIN_3
    DNS.4 = DOMAIN_4

File server.key

    openssl genrsa -out server.key 2048

File server.crt

    openssl req -sha512 -new -key server.key -out server.csr -config server.conf
    echo "C2E9862A0DA8E970" > serial
    openssl x509 -days 3650 -req -sha512 -in server.csr -CAserial serial -CA ca.crt -CAkey ca.key -out server.crt -extensions v3_req -extfile server.conf
    mv server.key server.key.pem && openssl pkcs8 -in server.key.pem -topk8 -nocrypt -out server.key

## Bước 3: Filebeat client (tạo file client.key và client.crt)
File client.conf

    [req]
    distinguished_name = req_distinguished_name
    req_extensions = v3_req
    prompt = no
    
    [req_distinguished_name]
    countryName                     = XX
    stateOrProvinceName             = XXXXXX
    localityName                    = XXXXXX
    postalCode                      = XXXXXX
    organizationName                = XXXXXX
    organizationalUnitName          = XXXXXX
    commonName                      = XXXXXX
    emailAddress                    = XXXXXX

    [ usr_cert ]
    # Extensions for server certificates (`man x509v3_config`).
    basicConstraints = CA:FALSE
    nsCertType = client, server
    nsComment = "OpenSSL FileBeat Server / Client Certificate"
    subjectKeyIdentifier = hash
    authorityKeyIdentifier = keyid,issuer:always
    keyUsage = critical, digitalSignature, keyEncipherment, keyAgreement, nonRepudiation
    extendedKeyUsage = serverAuth, clientAuth

    [v3_req]
    keyUsage = keyEncipherment, dataEncipherment
    extendedKeyUsage = serverAuth, clientAuth

File client.key

    openssl genrsa -out client.key 2048

File client.crt

    openssl req -sha512 -new -key client.key -out client.csr -config client.conf
    openssl x509 -days 3650 -req -sha512 -in client.csr -CAserial serial -CA ca.crt -CAkey ca.key -out client.crt -extensions v3_req -extensions usr_cert  -extfile client.conf


