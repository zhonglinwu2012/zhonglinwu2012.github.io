---
layout: post
title:  aerospikeのinstall
---

## install

http://www.aerospike.com/ を開く

Try Communication Edition をクリック

Set up an Aerospike Serverをクリック

Linuxを選択してクリック

Continueをクリック

Choose Your Linuxをクリック

Install On Redhatを選択

画面の指示通りを実行する

注意：aerospikeは3000と3001を使うが、開発環境ではport衝突する場合、それを修正

```
sudo vi /etc/aerospike/aerospike.conf
```

その中のport 3000とport 3001を別の番号に書き換える、例えば5000,5001

## 開発ツール関係

コマンドラインではaqlとascliがあります、
それぞれsqlライクとcliのコマンドです

ruby用はgem install aerospikeでgem installできる
https://github.com/aerospike/aerospike-client-ruby
↑のexamples以下に例があります

embulk のinstallについては https://github.com/embulk/embulk#linux--mac--bsd を参照

aerospikeのpluginは embulk gem install embulk-output-aerospike でinstall

https://github.com/tkrs/embulk-output-aerospike

パラメータなどは↑を参照

## embulkでデータロード

mysqlのDBからデータをaerospikeにロードしてみる。
一応成功しているように見えるが、結果がおかしい、全部最後のデータになってしまいます。

DBの内容：

|user_id|user_no|customer_status|
|---|---|---|
|0000000000609692|1|ng|
|0000000000609777|2|ng|
|0000000000609787|3|ok|

config.yml:

```yaml
in:
  type: mysql
  host: localhost
  user: root
  password: root
  database: test
  query: "select user_id, user_no, customer_status as status from users limit 3"
out:
  type: aerospike
  hosts:
  - {name: '127.0.0.1', port: 5000}
  command: put
  namespace: bar
  set_name: users
  key_name: user_id
  client_policy:
    max_retries: 3
  write_policy:
    generation: 0
    expiration: 64000
    send_key: true
```

embulkを実行する

```
embulk run config.yml
```

aqlで結果を確認すると

|user_id|user_no|customer_status|
|---|---|---|
|0000000000609692|3|ok|
|0000000000609777|3|ok|
|0000000000609787|3|ok|


次回 rubyとaerospikeで試す
