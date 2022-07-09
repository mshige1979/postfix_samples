# DKIM設定
## インストール
### opendkim
```
sudo yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
sudo yum install opendkim
```

## キー生成
### ディレクトリ作成
```
sudo mkdir /etc/opendkim/keys/example.com
```

### キー生成
```
sudo opendkim-genkey -D /etc/opendkim/keys/example.com/ -d example.com -s default
sudo cat /etc/opendkim/keys/example.com/default.txt
----
default._domainkey	IN	TXT	( "v=DKIM1; k=rsa; "
	  "p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCn1m55gXIjjWgsGPtMhxA7hpboYMq2CHPjTppKwbgk/sbeJLpnssUk96WN6/ZcR45jEsHBTk5BYG+3XBcrmeEAx3KM+LxL/kEtZ7JpI55zBiC4FUOAX5ZkJC+uaPUBIMYq1XV1ntK+IYVWOINkdZ6ajj+BKtwxnTF7X6AlW6l0swIDAQAB" )  ; ----- DKIM key default for example.com----
```

### 権限
```
sudo chown -R opendkim:opendkim /etc/opendkim/keys
```

### 設定ファイル更新1
```
sudo vi /etc/opendkim.conf

* Mode
---
#Mode   v
Mode    sv
---

* KeyFile
----
#KeyFile        /etc/opendkim/keys/default.private
----

* KeyTable
----
# KeyTable      /etc/opendkim/KeyTable
KeyTable       refile:/etc/opendkim/KeyTable
----

* SigningTable
----
# SigningTable  refile:/etc/opendkim/SigningTable
SigningTable    refile:/etc/opendkim/SigningTable
----

* ExternalIgnoreList
----
# ExternalIgnoreList    refile:/etc/opendkim/TrustedHosts
ExternalIgnoreList     refile:/etc/opendkim/TrustedHosts
----

* InternalHosts
----
# InternalHosts refile:/etc/opendkim/TrustedHosts
InternalHosts  refile:/etc/opendkim/TrustedHosts
----

```

### 設定ファイル更新2
```
sudo vi /etc/opendkim/KeyTable

----
#default._domainkey.example.com example.com:default:/etc/opendkim/keys/default.private
default._domainkey.example.com example.com:default:/etc/opendkim/keys/example.com/default.private
----
```

### 設定ファイル更新3
```
sudo vi /etc/opendkim/SigningTable
----
*@example.com default._domainkey.example.com
----
```

## サービス起動
```
sudo systemctl start opendkim.service
sudo systemctl enable opendkim.service
sudo systemctl status opendkim.service
```

## postfix変更
### /etc/postfix/main.cf
```
sudo vi /etc/postfix/main.cf
----
# DKIM (追記)
smtpd_milters     =    inet:127.0.0.1:8891
non_smtpd_milters = inet:localhost:8891
milter_default_action = accept
----
※末尾に追加
```

### 再起動
```
sudo systemctl restart postfix
```

## route53にレコード追加
### DKIM
```
Name: default._domainkey
Type: TXT
Value: "v=DKIM1; k=rsa; p=MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCn1m55gXIjjWgsGPtMhxA7hpboYMq2CHPjTppKwbgk/sbeJLpnssUk96WN6/ZcR45jEsHBTk5BYG+3XBcrmeEAx3KM+LxL/kEtZ7JpI55zBiC4FUOAX5ZkJC+uaPUBIMYq1XV1ntK+IYVWOINkdZ6ajj+BKtwxnTF7X6AlW6l0swIDAQAB"
```

### DMARC
```
Name: _dmarc
Type: TXT
Value: "v=DMARC1; p=none; rua=mailto:info@example.com; ruf=mailto:info@example.com"
```
