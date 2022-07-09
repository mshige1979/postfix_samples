# no-reply設定

## エイリアス関連定義差替え
### /etc/postfix/main.cf

#### alias_maps
```
#alias_maps = hash:/etc/aliases
alias_maps = hash:/etc/postfix/aliases
```

#### alias_database
```
#alias_database = hash:/etc/aliases
alias_database = hash:/etc/postfix/aliases
```

#### allow_mail_to_commands
```
allow_mail_to_commands = alias,forward,include
```

## バーチャル設定
### /etc/postfix/main.cf
#### virtual_alias_maps
```
# バーチャル用ファイルの使用
virtual_alias_maps = pcre:/etc/postfix/virtual.regxp
```

#### local_transport
```
# 一部のメールアドレスをローカル転送する
local_transport = local
transport_maps = hash:/etc/postfix/transport
```

#### default_privs
```
# スクリプト実行ユーザー指定
default_privs = vmail
```

## バーチャルエイリアス
### /etc/postfix/virtual.regxp
```
/^no\-reply@example.com$/   devnull
```

## エイリアス設定
### /etc/postfix/aliases
```
devnull: /dev/null
```

## ローカルへ配送
### /etc/postfix/transport
```
devnull@example.com      local
```

## 再起動
```
sudo postmap /etc/postfix/transport
sudo newaliases
sudo systemctl reload postfix
```
