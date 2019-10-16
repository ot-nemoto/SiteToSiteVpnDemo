# SiteToSiteVpnDemo

## 概要

## 構成

## 事前準備

- [VsOS Marketplace page](https://aws.amazon.com/marketplace/pp/B07N3X1P1T?qid=1555959590559&sr=0-1&ref_=srh_res_product_title)
  - Continue to Subscribe
  - Accept Terms


```sh
aws ec2 create-key-pair \
    --key-name tokyo-site \
    --query 'KeyMaterial' \
    --region ap-northeast-1 \
    --output text > tokyo-site.pem
```

```sh
aws ec2 create-key-pair \
    --key-name seoul-site \
    --query 'KeyMaterial' \
    --region ap-northeast-2 \
    --output text > seoul-site.pem
```

## デプロイ

```sh
aws cloudformation create-stack \
    --stack-name site-to-site-vpn-demo \
    --capabilities CAPABILITY_IAM \
    --template-body file://template.yaml
```
