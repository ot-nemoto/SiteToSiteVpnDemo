# SiteToSiteVpnDemo

## 概要

## 構成

## 事前準備

- デモで使うVyOSのイメージはFree Trial(30days)のある有償版を利用し、Subscribeする必要があります。
- [VsOS Marketplace page](https://aws.amazon.com/marketplace/pp/B07N3X1P1T?qid=1555959590559&sr=0-1&ref_=srh_res_product_title)
  - Continue to Subscribe
  - Accept Terms

## デプロイ

```sh
aws cloudformation create-stack \
    --stack-name site-to-site-vpn-demo \
    --capabilities CAPABILITY_IAM \
    --template-body file://template.yaml
```
