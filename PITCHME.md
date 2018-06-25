### Tokyo Quantopian User Group Vol 4
shinseitaro

---
### 自己紹介

[しんせいたろう(@shinseitaro)](https://twitter.com/shinseitaro)

個人トレーダー

---
### コミュニティ
  + 主催：Tokyo Quantopian User Group / 月刊フィントーク / モグモグDjango
  + スタッフ：GtugGirls / オプション勉強会 
  
---
### 今日の予定

1. Quantopian Algorithmの書き方
  + 基本編
2. バックテスト結果の見方
  + Build Algorithm
  + Full Backtest
  + Tear Sheet
3. Future Algorithm：
  + クラッシュスプレッド
  + カレンダースプレッド
  + その他
---
### 資料

[http://bit.ly/TQUG-004](http://bit.ly/TQUG-004)


---
### Quantopian Algorithmの書き方 基本編

```python
def initialize(context):
    ## 原油２０１７年１月限 (10/01/2016~12/20/2016)
    context.my_future = future_symbol("CLF17")
    schedule_function(my_rebalance, 
                      date_rule=date_rules.every_day(), 
                      time_rule=time_rules.market_open(hours=1))
```
注意① `def initialize(context)` は，バックテストスタート時にまず実行される関数．アルゴリズムの設定や変数の作成などはココで行う．
+ 銘柄設定
+ スケジュール
+ トレードフラグ等

--- 
### 基本編

注意② **【重要】** グローバル変数は必ず `context` の属性としてつくる

ダメな例

```python
my_future = future_symbol("CLF17") #←ダメゼッタイ！

def initialize(context):
    ## 原油２０１７年１月限 (10/01/2016~12/20/2016)
    schedule_function(my_rebalance, 
                      date_rule=date_rules.every_day(), 
                      time_rule=time_rules.market_open(hours=1))
```  
---
### 基本編

```python
def my_rebalance(context, data):
    cl_price = data.history(context.my_future, 
                            fields ='price', 
                            bar_count = 2, 
                            frequency = '1d')
```
`data.history(symbols, fields, bar_count, frequency)`: 

+ ヒストリカルデータ取得．データは引数によって， `pd.Series` `pd.DataFrame` `pd.Panel`のいずれか 
+ symbols: 銘柄，もしくは銘柄リスト．
+ fields: フィールド，もしくはそのリスト．`price`, `open`, `high`, `low`, `close`, `volume`．複数選択可．
+ bar_count: int. `frequency` で`1d`をした場合は，過去n日分．`1m`をした場合は，過去n分分．
+ frequency:　日足 = `1d`, 分足 = `1m`　のいずれか

---

### 基本編

**注意**
+ data.historyで取得したヒストリカルデータの最新のデータは，データを取ったタイミングのデータ．
+ 2017-03-16 10:30AM に `frequency = '1d'`, `fields ='price'`,`bar_count = 4`, で取得した場合，

時刻|AAAA|BBBB|CCCC
---|---|---|---
2017-03-13 16:00|6.64|10.98|6.94
2017-03-14 16:00|10.45|7.62|8.57
2017-03-15 16:00|9.84|5.96|3.26
2017-03-16 10:30|7.42|2.40|4.26

---

### 基本編

`log.info(msg)` : `error`, `warn`, `debug`

`print`文も使える．

---
### 基本編

先物を一枚注文する時：

+ ロング `order_target(context.my_future, 1)`
+ ショート `order_target(context.my_future, -1)`
+ クローズ `order_target(context.my_future, 0)`

注意

+ https://www.quantopian.com/help#api-order-methods
+ Quantopianでは，`order_optimal_portfolio` が推奨されているが今回は使わない．しかし，将来的には必ずこのオーダー関数を使う必要が出てくるので，気になる方は，[order_optimal_portfolio](https://www.quantopian.com/help#api-order-optimal-portfolio) / [Optimize API](https://www.quantopian.com/help#optimize-api) を参照．

---
### バックテスト結果の見方 Build Algorithm

1. スタート，エンド，想定元本
1. **US Futures** を指定
  ![Screenshot from 2018-05-16 14-30-59.png](https://qiita-image-store.s3.amazonaws.com/0/14019/5cb465ac-b6c9-aa01-a862-dec001a1cb6f.png)
1. Build Algorithm 押下 →　文法エラー等がなければ，結果が出力される

---
### Full Backtest

**Run Full Backtest** 押下


---
### Tear Sheet
--- 
### Future Algorithm クラッシュスプレッド
---
### カレンダースプレッド
--- 
### その他
