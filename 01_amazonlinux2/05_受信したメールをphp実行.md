# 受信したメールをphpスクリプトで実行
## インストール
### php
```
sudo amazon-linux-extras install php7.4
```
### composer download
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '906a84df04cea2aa72f40b5f787e49f22d4c2f19492ac310e8cba5b96ac8b64115ac402c8cd292b8a03482574915d1a8') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php
php -r "unlink('composer-setup.php');"
```

### composer install
```
sudo mv composer.phar /usr/local/bin/composer
```
↓  
```
[ec2-user@ip-172-30-1-28 ~]$ composer -V
Composer version 2.1.14 2021-11-30 10:51:43
[ec2-user@ip-172-30-1-28 ~]$
```

## backup
```
# postfix
sudo cp -p /etc/postfix/main.cf /etc/postfix/main.cf.org.4
sudo cp -p /etc/aliases /etc/aliases.org.4
```

## 受信用仮想ユーザーの作成

### /etc/postfix/virtual-mailbox

#### 形式

```
test04@testmail-20211204.tk testmail-20211204.tk/test04/Maildir/
test05@testmail-20211204.tk testmail-20211204.tk/test05/Maildir/
recv_user01@testmail-20211204.tk testmail-20211204.tk/recv_user01/Maildir/
recv_user02@testmail-20211204.tk testmail-20211204.tk/recv_user02/Maildir/
recv_user03@testmail-20211204.tk testmail-20211204.tk/recv_user03/Maildir/
recv_user04@testmail-20211204.tk testmail-20211204.tk/recv_user04/Maildir/
recv_user05@testmail-20211204.tk testmail-20211204.tk/recv_user05/Maildir/
```

#### ハッシュ化

```
sudo postmap /etc/postfix/virtual-mailbox
```

### /etc/dovecot/passwd

上記で生成したパスワードとメールアカウントを設定

```
test04@testmail-20211204.tk:{PLAIN}password123
test05@testmail-20211204.tk:{PLAIN}password123
recv_user01@testmail-20211204.tk:{PLAIN}password123
recv_user02@testmail-20211204.tk:{PLAIN}password123
recv_user03@testmail-20211204.tk:{PLAIN}password123
recv_user04@testmail-20211204.tk:{PLAIN}password123
recv_user05@testmail-20211204.tk:{PLAIN}password123
```


## postfix
### /etc/postfix/main.cf
#### alias_maps
```
# エイリアス設定
alias_maps = hash:/etc/postfix/aliases
alias_database = hash:/etc/postfix/aliases
allow_mail_to_commands = alias,forward,include
```

#### virtual_alias_maps
```
# バーチャル用ファイルの使用
virtual_alias_maps = pcre:/etc/postfix/virtual.regxp
```

#### #### local_transport
```
# 一部のメールアドレスをローカル転送する
local_transport = local
transport_maps = hash:/etc/postfix/transport
```

### /etc/postfix/virtual.regxp
recvから始まるメールをrecvのローカルアドレスに転送
```
/^recv\_(.*)\@testmail\-20211204\.tk$/   recv
```

### /etc/postfix/transport
以下のメールアドレスをローカルアドレス対応
```
recv@testmail-20211204.tk      local
```

### /etc/postfix/aliases
recvの場合はphpスクリプトを実行
```
recv: "|/usr/bin/php /tmp/hoge.php"
```

### /tmp/hoge.php
テストロジックで確認
```
<?php
file_put_contents("/tmp/aaa.txt", "aaaaaaaa", FILE_APPEND);
```

### hash化
```
sudo postmap /etc/postfix/transport
```

### newaliases
```
sudo newaliases
```

### postfix reload
```
sudo systemctl reload postfix
```
