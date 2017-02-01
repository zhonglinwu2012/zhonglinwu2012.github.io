---
# LOGとaerospikeのテスト
---

### fluentdとaerospikeを組みあわせる

*fluentd* のログデータをaerospikeに投入する場合を考える。

#### fluentdのinstallと設定
以下のリンクを参照
http://docs.fluentd.org/articles/install-by-rpm#step-1-install-from-rpm-repository

```bash
curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent2.sh | sh
```

#### fluent-plugin-aerospikeのinstall

```
sudo /usr/lib64/fluent/ruby/bin/gem install fluent-plugin-aerospike
```

#### 設定ファイルを編集
設定ファイルはdefaultは/etc/td-agent以下にある。

```bash
[root@localhost td-agent]# pwd
/etc/td-agent
[root@localhost td-agent]# ls -1
logrotate.d
prelink.conf.d
td-agent-activities.conf
td-agent.conf
td-agent.conf.tmpl
```

以下は設定の一例

```
[root@localhost td-agent]# cat td-agent.conf

<source>
  type multiprocess
  <process>
    cmdline -c /etc/td-agent/td-agent-activities.conf --log /var/log/td-agent/td-agent-activities.log
    sleep_before_start 1s
    sleep_before_shutdown 5s
  </process>
</source>

[root@localhost td-agent]# cat td-agent-activities.conf 

<source>
  type tail
  path /var/www/app/shared/log/act.log
  pos_file /var/log/td-agent/app-log_activities.log.pos
  tag activities
  format json
</source>
<match activities>
  type rewrite_tag_filter
  rewriterule1 account ^(.*)$ activities.$1
</match>
<match activities.*>
  type aerospike
  address 127.0.0.1:5000
  namespace bar
  set acti
</match>
```

/var/www/app/shared/log/act.logは以下のようなログ形式とする

```
{"ts":1435025600603,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025679302,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025694660,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025729249,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025731348,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025750194,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025771624,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025775009,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025777724,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025778852,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025780438,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025856827,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025880810,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435025982539,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435048801611,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435060089216,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1435060218954,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"detail","act_params":[{"commodityCode":"CC55463"}]}
{"ts":1446781223481,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"bdr_view","act_params":[{"r_id":"64"}]}
{"ts":1446781224465,"account":"ac_123_55","vt":"2e7RHi.kSTLPuk","cid":"12345678","ctype":"1","act_type":"bdr_view","act_params":[{"r_id":"64"}]}
```

#### fluentdの動作確認
fluentdの起動と終了はそれぞれ以下のコマンドで実現する

```
/etc/init.d/td-agent start
```

```
/etc/init.d/td-agent stop
```

fluentdを起動するしてから、ログが変更になると、aerospikeにレコードが書き込まれる

```sql
aql> select * from bar.acti
+---------------------------------------------------+---------------+-------------+------------------+------------+-------+------------+-------------------------------+
| key                                               | ts            | account     | vt               | cid        | ctype | act_type   | act_params                    |
+---------------------------------------------------+---------------+-------------+------------------+------------+-------+------------+-------------------------------+
| "1453689499-3381d84f-4795-4b2d-a0f9-636e78e2a049" | 1435025982539 | "ac_123_55" | "2e7RHi.kSTLPuk" | "12345678" | "1"   | "detail"   | [{"commodityCode":"CC55463"}] |
| "1453689441-52c594e5-36af-42be-843c-baf8f40864e0" | 1435025694660 | "ac_123_55" | "2e7RHi.kSTLPuk" | "12345678" | "1"   | "detail"   | [{"commodityCode":"CC55463"}] |
| "1453689441-3005ed82-162f-4d9f-9ea8-e7fe85ab41e4" | 1447070740725 | "ac_123_55" | "2e7RHi.kSTLPuk" | "12345678" | "1"   | "bdr_view" | [{"bdr_r_id":"64"}]           |
| "1453689441-22193e19-5959-4ad1-aa75-42b02eae6140" | 1435025982539 | "ac_123_55" | "2e7RHi.kSTLPuk" | "12345678" | "1"   | "detail"   | [{"commodityCode":"CC55463"}] |
| "1453689499-3cb6eaae-ac2c-4a3b-8dd3-169981fbac36" | 1435025856827 | "ac_123_55" | "2e7RHi.kSTLPuk" | "12345678" | "1"   | "detail"   | [{"commodityCode":"CC55463"}] |
| "1453689499-f1451be5-6176-4184-bf6b-b7e03e414bea" | 1435025775009 | "ac_123_55" | "2e7RHi.kSTLPuk" | "12345678" | "1"   | "detail"   | [{"commodityCode":"CC55463"}] |
...
```

#### ついでにfluentdから別のhttp serverにpostし直すことも試した。
gemのinstall

```
/usr/lib64/fluent/ruby/bin/gem install fluent-plugin-out-http
```

設定ファイル td-agent-activities.conf:

```
<source>
  type tail
  path /var/www/mario-app/shared/log/act.log
  pos_file /var/log/td-agent/mario-app-log_activities.log.pos
  tag activities
  format json
</source>
<match *>
  type http
  endpoint_url http://127.0.0.1:4567/
  http_method get
  raise_on_error false
</match>
```

受け取るhttp serverの実装はとにかく手間を省くため、sinatraで実装

```
require 'sinatra'

get "/" do 
  File.open("log","a") do |f|
    f.puts(params)
  end
  params.to_s
end

post "/" do
  File.open("log","a") do |f|
    f.puts(params)
  end
  params.to_s
end
```

http serverを起動

```
[vagrant@localhost aerospike]$ ruby myapp.rb
[2016-01-25 06:06:29] INFO  WEBrick 1.3.1
[2016-01-25 06:06:29] INFO  ruby 2.1.2 (2014-05-08) [x86_64-linux]
== Sinatra (v1.4.6) has taken the stage on 4567 for development with backup from WEBrick
[2016-01-25 06:06:29] INFO  WEBrick::HTTPServer#start: pid=8359 port=4567
```

前記のact.logを変更すると、logファイルが出力される。
結果のlogの中身の例

```javascript
{"ts"=>"1435025600603", "account"=>"ac_123_55", "vt"=>"2e7RHi.kSTLPuk", "cid"=>"12345678", "ctype"=>"1", "act_type"=>"detail", "act_params"=>"{\"commodityCode\"=>\"CC55463\"}"}
{"ts"=>"1435025679302", "account"=>"ac_123_55", "vt"=>"2e7RHi.kSTLPuk", "cid"=>"12345678", "ctype"=>"1", "act_type"=>"detail", "act_params"=>"{\"commodityCode\"=>\"CC55464\"}"}
{"ts"=>"1435025679412", "account"=>"ac_123_55", "vt"=>"2e7RHi.kSTLPuk", "cid"=>"12345678", "ctype"=>"1", "act_type"=>"detail", "act_params"=>"{\"commodityCode\"=>\"NE663BU102XX\"}"}
...
```
