# Tokyo Quantopian User Group Vol 4
shinseitaro

---
## 自己紹介

[しんせいたろう(@shinseitaro)](https://twitter.com/shinseitaro)

#### 個人トレーダー
+ VIF ETF など

#### コミュニティ
+ 主催：Tokyo Quantopian User Group / 月刊フィントーク / モグモグDjango
+ スタッフ：GtugGirls / オプション勉強会

---
## 今日の予定

1. Quantopian Algorithmの書き方
  + 基本編
    + `initialize`
    + `context`
    + `schedule`
    + `future_symbol`
    + `continuous_future`
    + `data.current`
    + `data.history`
    + contract
    + `log`
    + `order_target`

2. バックテスト結果の見方
  + Build Algorithm
  + Full Backtest
  + Tear Sheet
3. Future Algorithm：
  + クラッシュスプレッド
  + カレンダースプレッド
  + その他
---
## 資料

[http://bit.ly/TQUG-004](http://bit.ly/TQUG-004)

---
## Quantopian Algorithmの書き方 基本編

[ここにClone先のリンクを張る．]()

Quantopianで利用できる先物とそのヒストリカルデータは，
[Continuous Future Data Lifespans](https://goo.gl/KbFxjx) で確認して下さい．



---
### def initialize(context)

バックテストスタート時に最初に実行される関数

全体の設定を行う．
+ 銘柄設定
+ スケジュール設定
+ アルゴリズム全体で使う変数作成
+ その他

---
**【重要】** グローバル変数は必ず `context` の属性としてつくる

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
### スケジューラー

```schedule_function()```

必ず，`initialize` 内で定義．引数説明は最後のDoc参照

---

`future_symbol` と `continuous_future`

評価したい先物のオブジェクトを作ると，そのオブジェクトを利用して，簡単に先物価格を取得したり，注文したりできる．

`future_symbol`: 一代足先物
`continuous_future`: つなぎ足先物

---

### 一代足

```python
    ## 原油２０１７年１月限 (10/01/2016~12/20/2016)
    context.my_future = future_symbol("CLF17")
    ```

---

### つなぎ足

```python
    # 原油期近つなぎ足
    context.my_future = continuous_future('CL',
                                          offset=0, # 限月．期近＝0，2番限＝1，3番限=2，・・・
                                          roll='calendar', # ロールするタイミング
                                          adjustment='mul' # アジャスト方法
                                         )
```

このオブジェクトを使って，ロール日などを気にしなくても，ほしい先物データにアクセスできる．

---
### ヒストリカルデータ

```python
    cl_price = data.history(context.my_future,
                            fields ='price',
                            bar_count = 2,
                            frequency = '1d')
```

第一引数に渡した先物オブジェクトのヒストリカルデータを取得．

ユーザーは，自分でつなぎ足をつくることなく，この方法でヒストリカルデータを取得できる．


---
### current contract

今日現在の先物のコントラクトを取得する

```python
cl_contract = data.current(context.my_future, 'contract')
```

このコントラクトを使って，現在価格取得や注文を行う．

---
注意：

+ ヒストリカルデータを取得する時→ `continuous_future` オブジェクト
+ 現在価格取得や注文を行う時→ `data.current` に `contract` を渡して，コントラクトオブジェクトを作り，このオブジェクトを該当の関数に渡す．


---
### 現在価格取得

```python
 cl_contract = data.current(context.my_future, 'contract')
 current_price = data.current(cl_contract, 'price')
```


---
### 注文

先物を一枚注文する時：

+ ロング `order_target(context.my_future, 1)`
+ ショート `order_target(context.my_future, -1)`
+ クローズ `order_target(context.my_future, 0)`

---
注意

+ https://www.quantopian.com/help#api-order-methods
+ Quantopianでは，`order_optimal_portfolio` が推奨されているが今回は使わない．しかし，将来的には必ずこのオーダー関数を使う必要が出てくるので，気になる方は，[order_optimal_portfolio](https://www.quantopian.com/help#api-order-optimal-portfolio) / [Optimize API](https://www.quantopian.com/help#optimize-api) を参照．

---
### ロギング

`log.info(msg)` : `error`, `warn`, `debug`

`print`文も使える．


---
## バックテスト結果の見方 Build Algorithm

1. スタート，エンド，想定元本
1. **US Futures** を指定
  ![Screenshot from 2018-05-16 14-30-59.png](https://qiita-image-store.s3.amazonaws.com/0/14019/5cb465ac-b6c9-aa01-a862-dec001a1cb6f.png)
1. Build Algorithm 押下 →　文法エラー等がなければ，結果が出力される

---
## Full Backtest

**Run Full Backtest** 押下


---
## Tear Sheet
---
## Future Algorithm クラッシュスプレッド
---
## カレンダースプレッド

---
## DOCS

`schedule_function(func,
                   date_rule=date_rule,
                   time_rule=time_rule,
                   calendar=calendar,
                   half_days=True)`

func: 指定した時間に実行したい関数名
date_rule: 日付の指定．
+ `date_rules.every_day()`：毎日（デフォルト）
+ `date_rules.week_start(days_offset=0)`：週初めからdays_offset日分オフセットした日．
+ `date_rules.week_end(days_offset=0)`：週終わりからdays_offset日分巻き戻した日．
+ `date_rules.month_start(days_offset=0)`：月初めからdays_offset日分オフセットした日．
+ `date_rules.month_end(days_offset=0)` ：月終わりからdays_offset日分巻き戻した日．

time_rule：時間の指定．
+ `time_rules.market_open(hours=0, minutes=1)`（デフォルト）：取引開始時から〜時間〜分後
+ `time_rules.market_close(hours=0, minutes=1)`：取引終了時から〜時間〜分前

calendar：カレンダーの指定．デフォルトは，バックテスト時に選んだカレンダー．
+ `calendars.US_EQUITIES`
+ `calendars.US_FUTURES`

half_days：マーケットが半日しか開いていない日（例；クリスマスの次の日）にスケジュールを実行するかどうか．デフォルトはTrue．

【メモ】： 先物の場合：market open 6:30AM ET 〜 market close 5PM ET.
[schedulefunction](https://www.quantopian.com/help#ide-schedulefunction)参照

---

`data.history(symbols, fields, bar_count, frequency)`:

+ ヒストリカルデータ取得．データは引数によって， `pd.Series` `pd.DataFrame` `pd.Panel`のいずれか
+ symbols: 銘柄，もしくは銘柄リスト．
+ fields: フィールド，もしくはそのリスト．`price`, `open`, `high`, `low`, `close`, `volume`．複数選択可．
+ bar_count: int. `frequency` で`1d`をした場合は，過去n日分．`1m`をした場合は，過去n分分．
+ frequency:　日足 = `1d`, 分足 = `1m`　のいずれか

---
**注意**
+ data.historyで取得したヒストリカルデータの最新のデータは，データを取ったタイミングのデータ．
+ 2017-03-16 10:30AM に `frequency = '1d'`, `fields ='price'`,`bar_count = 4`, で取得した場合，

時刻|AAAA|BBBB|CCCC
---|---|---|---
2017-03-13 **16:00**|6.64|10.98|6.94
2017-03-14 **16:00**|10.45|7.62|8.57
2017-03-15 **16:00**|9.84|5.96|3.26
2017-03-16 **10:30**|7.42|2.40|4.26


---

`future_symbol(symbol)`

+ 一代足先物オブジェクトを返す
+ symbol: 文字列．

symbolの書き方

+ Symbol+月コード+年コード
+ 例 CLF160 CL=CrudeOil F=January 16=2016

![](https://www.quantopian.com/assets/futures_getting_started1_l2_screenshot1-0bc90a54a7e14712bce267e0a7b0bb6e06de3c5ddf302c8102274f687628f68a.png)

---

`continuous_future(root_symbol, offset=0, roll='volume', adjustment='mul')` [参照](https://www.quantopian.com/help#quantopian_research_experimental_continuous_future)

+ つなぎ足オブジェクトを返す．（このオブジェクトは．`specifier` と呼ばれている）
+ root_symbol: str. 原資産シンボル．利用できる先物は[こちら](https://www.quantopian.com/help#available-futures)
+ offset: int. 限月．期近＝0，2番限＝1，3番限=2，・・・
+ roll: ロールのタイミングを指定．
    + 'volume' (default) : 次の限月の取引量が期近の取引量を超えた時ロールする．これは，マーケットのトレーダーが次の限月に移ったというアイデアがベースになっている．もし，auto_close_dateまでにこの現象が起きなければ，auto_close_dateがロール日になる．また，ロールはauto_close_dateの７営業日前まで起きない．
    + 'calendar' : コントラクトがauto_close_dateに達した時にロールする

+ adjustment: アジャスト方法
    + 'mul' (default): 連続したコントラクトの価格比を価格シリーズに掛けたデータ．
    + 'add': 連続したコントラクトの価格差を加算したデータ
    + None: アジャスト無し



---

`data.current(assets, fields)`  [参照](https://www.quantopian.com/help#api-data-current)

+ assets の現在価格や，コントラクトを返す．価格は，実際の取引価格．
+ assets: `continuous_future` オブジェクト．もしくはそのリスト．
+ fields: 'price', 'last_traded', 'open', 'high', 'low', 'close', 'volume', 'contract' のいずれか．もしくは複数入ったリスト．






