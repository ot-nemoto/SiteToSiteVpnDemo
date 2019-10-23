# SiteToSiteVpnDemo

## 概要

## 構成

## 事前準備

- デモで使うVyOSのイメージはFree Trial(30days)のある有償版を利用し、Subscribeする必要があります
  - [VsOS Marketplace page](https://aws.amazon.com/marketplace/pp/B07N3X1P1T?qid=1555959590559&sr=0-1&ref_=srh_res_product_title)
    - Continue to Subscribe
    - Accept Terms

- インスタンス接続用鍵
  - 既にある場合は作成しなくても問題ありません
  - スタック作成時に、デフォルトで **SiteToSiteVpnDemo** というキー名を指定しているので、作成済みのキー名（`KeyName`）を指定して下さい。

```sh
aws ec2 create-key-pair \
    --key-name SiteToSiteVpnDemo \
    --query 'KeyMaterial' \
    --region ap-northeast-1 \
    --output text > SiteToSiteVpnDemo.pem

chmod 600 SiteToSiteVpnDemo.pem
ls -l SiteToSiteVpnDemo.pem
  # -rw------- 1 ec2-user ec2-user 1671 Oct 21 08:29 SiteToSiteVpnDemo.pem
```

## デプロイ

**Properties**

|Name|Type|Default|Description|
|--|--|--|--|
|AMIId|String|ami-0ff21806645c5e492|Amazon Linux 2 AMI (HVM), SSD Volume Type|
|VyOSAMIId|String|ami-059473b296f19f858|VyOS (HVM) **1.2.3**|
|VyOSInstanceType|String|m3.medium|VyOSのインスタンスタイプ|
|KeyName|AWS::EC2::KeyPair::KeyName|SiteToSiteVpnDemo|キーペア名|
|LocalPublicIp|String|0.0.0.0/0|自身の環境のパブリックIPアドレス|

```sh
aws cloudformation create-stack \
    --stack-name site-to-site-vpn-demo \
    --capabilities CAPABILITY_IAM \
    --template-body file://template.yaml
```

デフォルトでは、SecurityGroupで **0.0.0.0/0** を許可していしまうので、自身の環境のパブリックIPアドレスを指定し、スタックを作成することを推奨（e.g. EC2インスタンスから作業しているケース）

```sh
PUBLIC_IP=$(curl -s http://169.254.169.254/latest/meta-data/public-ipv4)
echo ${PUBLIC_IP}
  # (e.g.) 13.231.159.206

aws cloudformation create-stack \
    --stack-name site-to-site-vpn-demo \
    --capabilities CAPABILITY_IAM \
    --parameters ParameterKey=LocalPublicIp,ParameterValue=${PUBLIC_IP}/32 \
    --template-body file://template.yaml
```

## VPN ConnectionからVyOSの設定をダウンロード

VPN ConnectionのURLは以下のコマンドから確認できます

```sh
aws cloudformation describe-stacks \
    --stack-name site-to-site-vpn-demo \
    --query 'Stacks[].Outputs[?OutputKey==`VpnConnectionUrl`].OutputValue' \
    --output text
  # (e.g.) https://ap-northeast-1.console.aws.amazon.com/vpc/home?region=ap-northeast-1#VpnConnections:search=vpn-020e8e85dbef567f7;sort=VpnConnectionId
```

**Download Configuraiton** > **Download**

![Download Configuration](https://github.com/ot-nemoto/SiteToSiteVpnDemo/blob/images/download_configuration.png)

## ダウンロードした設定を修正

- **local-address** がVyOSのパブリックIPアドレスで定義されているので、ローカルIPアドレスに変更する

  - VyOSのパブリックIPアドレスは以下のコマンドから確認できます

    ```sh
    VyOS_Public_IP=$(aws cloudformation describe-stacks \
        --stack-name site-to-site-vpn-demo \
        --query 'Stacks[].Outputs[?OutputKey==`VyOSPublicIp`].OutputValue' \
        --output text)
    echo ${VyOS_Public_IP}
      # (e.g.) 54.250.169.14
    ```

  - VyOSのプライベートIPアドレスは以下のコマンドから確認できます

    ```sh
    VyOS_Private_IP=$(aws cloudformation describe-stacks \
        --stack-name site-to-site-vpn-demo \
        --query 'Stacks[].Outputs[?OutputKey==`VyOSPrivateIp`].OutputValue' \
        --output text)
    echo ${VyOS_Private_IP}
      # (e.g.) 10.39.0.30
    ```

**VPN_ID**.txt

```
set vpn ipsec site-to-site peer 13.113.120.252 local-address '54.250.169.14'
set vpn ipsec site-to-site peer 52.193.148.209 local-address '54.250.169.14'
↓
set vpn ipsec site-to-site peer 13.113.120.252 local-address '10.39.0.30'
set vpn ipsec site-to-site peer 52.193.148.209 local-address '10.39.0.30'
```

## VyOSにダウンロードした設定を反映

VyOSにログイン

```sh
ssh -i SiteToSiteVpnDemo.pem vyos@${VyOS_Public_IP}
```

設定を反映

```sh
configure
  # [edit]

/* 設定を貼付け */

commit
  # [edit]
save
  # Saving configuration to '/config/config.boot'...
  # Done
  # [edit]
exit
  # exit
```

## 動作確認

**AWS Site**

```sh
AwsSite_Public_IP=$(aws cloudformation describe-stacks \
    --stack-name site-to-site-vpn-demo \
    --query 'Stacks[].Outputs[?OutputKey==`AwsSiteInstancePublicIp`].OutputValue' \
    --output text)
echo ${AwsSite_Public_IP}
  # (e.g.) 54.64.55.194

ssh -i SiteToSiteVpnDemo.pem ec2-user@${AwsSite_Public_IP}
```

**Customer Site**

```sh
Customer_Public_IP=$(aws cloudformation describe-stacks \
    --stack-name site-to-site-vpn-demo \
    --query 'Stacks[].Outputs[?OutputKey==`CustomerInstancePublicIp`].OutputValue' \
    --output text)
echo ${Customer_Public_IP}
  # (e.g.) 54.199.216.242

ssh -i SiteToSiteVpnDemo.pem ec2-user@${Customer_Public_IP}
```
