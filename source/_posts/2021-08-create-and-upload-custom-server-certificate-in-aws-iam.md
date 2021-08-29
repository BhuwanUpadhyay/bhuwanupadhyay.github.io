---
title: Create and upload custom server certificate in AWS IAM
cover: /gallery/media/fsqs-tips-tricks-notes-.png
date: 2021-08-10T10:08:22.921Z
categories:
  - AWS
  - SSL
tags:
  - aws
  - iam
  - server-certificate
  - ssl
---
How to create SSL custom certificate with you domain and upload as server certificate in AWS IAM? Read here...

<!-- more -->

1. Create setup directory

   ```
   mkdir -p ~/customcerts
   cd ~/customcerts
   rm -rf *
   ```
2. Setup variables

   ```
   SUBJECT="/C=CN/ST=GD/L=SZ/O=Acme, Inc."
   DOMAIN_SUFFIX=example.com
   CERTIFICATE_NAME=custom-loadbalancer-cert
   ```
3. Generate client key & certificate

   ```
   openssl genrsa -out ca.key 2048
   openssl req -new -x509 -days 365 -key ca.key -subj "$SUBJECT/CN=Acme Root CA" -out ca.crt
   ```
4. Generate server key & certificate

   ```
   openssl req -newkey rsa:2048 -nodes -keyout server.key -subj "$SUBJECT/CN=*.$DOMAIN_SUFFIX" -out server.csr
   openssl x509 -req -extfile <(printf "subjectAltName=DNS:*.$DOMAIN_SUFFIX,DNS:$DOMAIN_SUFFIX,DNS:www.$DOMAIN_SUFFIX") -days 365 -in server.csr -CA ca.crt -CAkey ca.key -CAcreateserial -out server.crt
   ```
5. Delete IAM server certificate if exists

   ```
   aws iam delete-server-certificate --server-certificate-name $CERTIFICATE_NAME
   ```
6. Upload IAM server certificate

   ```
   aws iam upload-server-certificate \
       --server-certificate-name $CERTIFICATE_NAME \
       --certificate-body file://server.crt \
       --private-key file://server.key
   ```