## rubyによるアクセス
rubyでaerospikeにアクセスしてみる。

#### install
まずgemをインストール

```
gem install aerospike
```

#### レコード作成(ruby code)
```ruby
require 'rubygems'
require 'aerospike'
include Aerospike
client = Client.new('127.0.0.1', 5000)

key = Key.new('bar', 'user', '0000000000609787')
bin1 = Bin.new('customer_code','0000000000609787')
bin2 = Bin.new('user_no',1)
bin3 = Bin.new('status','ok')

client.put(key,[bin1,bin2,bin3])
client.close
```
aqlで確認
```
aql> select * from bar.user
+--------------------+--------------------+---------+--------+
| key                | customer_code      | user_no | status |
+--------------------+--------------------+---------+--------+
| "0000000000609787" | "0000000000609787" | 1       | "ok"   |
+--------------------+--------------------+---------+--------+
1 row in set (0.038 secs)
```

#### rubyで読む
```ruby
require 'rubygems'
require 'aerospike'
include Aerospike
client = Client.new('127.0.0.1', 5000)

key = Key.new('bar', 'user', '0000000000609787')
record = client.get(key)
puts record.bins

puts client.exists(key)
client.close
```
実行結果
```
{"customer_code"=>"0000000000609787", "user_no"=>1, "status"=>"ok"}
true
```
#### レコードの削除
```ruby
require 'rubygems'
require 'aerospike'
include Aerospike
client = Client.new('127.0.0.1', 5000)

key = Key.new('bar', 'user', '0000000000609787')
record = client.get(key)
puts record.bins

client.delete(key)
puts client.exists(key)

client.close
```
実行結果
```
{"customer_code"=>"0000000000609787", "user_no"=>1, "status"=>"ok"}
false
```
#### データの書き換え
```ruby
require 'rubygems'
require 'aerospike'
include Aerospike
client = Client.new('127.0.0.1', 5000)

key = Key.new('bar', 'user', '0000000000609787')
bin1 = Bin.new('customer_code','0000000000609787')
bin2 = Bin.new('user_no',1)
bin3 = Bin.new('status','ok')
client.put(key,[bin1,bin2,bin3])

bin4 = Bin.new('status','ng')
client.put(key,[bin1,bin2,bin4])
client.close
```
結果
```
aql> select * from bar.user
+--------------------+--------------------+---------+--------+
| key                | customer_code      | user_no | status |
+--------------------+--------------------+---------+--------+
| "0000000000609787" | "0000000000609787" | 1       | "ng"   |
+--------------------+--------------------+---------+--------+
1 row in set (0.035 secs)
```
#### JSONデータ形式で直接書きこむ
```ruby
require 'rubygems'
require 'aerospike'
include Aerospike
client = Client.new('127.0.0.1', 5000)

key = Key.new('bar', 'user', '0000000000609692')
hm = {'customer_code'=>'0000000000609692','user_no'=>2,'status'=>'ok'}

client.put(key,hm)

client.close
```
結果
```
aql> select * from bar.user
+--------------------+--------------------+---------+--------+
| key                | customer_code      | user_no | status |
+--------------------+--------------------+---------+--------+
| "0000000000609787" | "0000000000609787" | 1       | "ng"   |
| "0000000000609692" | "0000000000609692" | 2       | "ok"   |
+--------------------+--------------------+---------+--------+
2 rows in set (0.038 secs)
```
次回*fluentd*と*aerospike*を試す
