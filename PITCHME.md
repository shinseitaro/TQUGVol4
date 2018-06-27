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

## quantopianとは

Frequently Asked Questions - FAQ - https://www.quantopian.com/faq


---
## Quantopian Algorithmの書き方 基本編

[ここにClone先のリンクを張る．]()

Quantopianで利用できる先物とそのヒストリカルデータは，
[Continuous Future Data Lifespans](https://goo.gl/KbFxjx)
[available-futures](https://www.quantopian.com/help#available-futures)
で確認して下さい．




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
### 注文

ポートフォリオに対して何％ホールドするか：

`opt.TargetWeights` 説明書く

```python

target_weight = {}
target_weight[sym1] = -0.1
target_weight[sym2] = 0.1

order_optimal_portfolio(
    opt.TargetWeights(target_weight),
    constraints=[]
        )
```

---

### 注意

+ https://www.quantopian.com/help#api-order-methods
+ Quantopianでは，`order_optimal_portfolio` が推奨されている．
+ [order_optimal_portfolio](https://www.quantopian.com/help#api-order-optimal-portfolio) / [Optimize API](https://www.quantopian.com/help#optimize-api) を参照．

---
### ロギング

`log.info(msg)` : `error`, `warn`, `debug`

`print`文も使える．


---
## バックテスト結果の見方 Build Algorithm

1. スタート，エンド，初期設定資金額
1. **US Futures** を指定
  ![Screenshot from 2018-05-16 14-30-59.png](https://qiita-image-store.s3.amazonaws.com/0/14019/5cb465ac-b6c9-aa01-a862-dec001a1cb6f.png)
1. Build Algorithm 押下 →　文法エラー等がなければ，結果が出力される

---
## Full Backtest

**Run Full Backtest** 押下

+ 結果はQuantopianが[Quantopian Contest](https://www.quantopian.com/contest)用に開発したもの．
+ リターンやリスク等をオサレに表示してくれる．
+ 詳しくは，こちら→ [Improved Backtest Analysis](https://www.quantopian.com/posts/improved-backtest-analysis)

---
## Full Backtest 詳細

### Risk
+ Leverage 資本金に対してポジションの額（毎日ベース） End-of-day gross leverage. A measure comparing position value to capital base.
+ Turnover アセットに対して投資資金額の比率．（63日移動平均） Turnover represents the rate at which assets are being bought and sold within the portfolio. The value displayed is the rolling 63-day mean turnover.
+ Beta To SPY 自分のポートフォリオのリターンとSPYのリターンの相関関係（6ヶ月移動BETA)．6-month rolling beta to SPY. Trailing beta measures the correlation between the portfolio's overall return stream and the returns of SPY.
+ Position Concentration ポートフォリオの中で1番比率の高い投資対象がポートフォリオの中でどのくらいの比率になっているか．（毎日ベース） End-of-day position concentration. The percentage of the algorithm's portfolio invested in its most-concentrated asset.
+ Net Dollar Exposure ポートフォリオ中のロングとショートのポジション比率

### Performance
+ Total Return バックテストスタートからエンドまでのリターン The total percentage return of the portfolio from the start to the end of the backtest.
+ Sharp リターンを標準偏差で割ったもの（6ヶ月移動）
+ Max Drawdown The largest peak-to-trough drop in the portfolio's history.
+ Volatility

### Activity
+ position 毎日のポジション
+ transaction トレードログ
+ Logs 取引中に出したログ
+ Code この full backtest で使用したコードのスナップショット．

### Notebook
+ jupyter notebook
+ 最初のセルを実行すると，一分くらいでTear Sheet が作成される
+ tear sheet を見て，アルゴリズムの改良を考える
+ 作った tear sheet は https://www.quantopian.com/research にストラテジー名と実行時刻とハッシュキーで格納されるので，いつでも確認できる．

---
## Future Algorithm クラッシュスプレッド

+ [Crack spread-Oil01.ipynb](https://github.com/drillan/quantopian/blob/master/driller/Crack_spread-Oil01.ipynb) の説明
+ https://www.quantopian.com/algorithms/5b3335b53ef854003ea5f25c
+ 'CL'(Light Sweet Crude Oil), 'HO'(NY Harbor USLD Futures (Heating Oil)), 'XB'(RBOB Gasoline Futures), 'NG'(Natural Gas)のヒストリカルデータを取得
+ 全ペアを作成して，限月毎に割合を算出

### 特徴を見てみる

+ offset 4 (5限月) の HO/XB や CL/XB あたりが何かアヤシイ
+ とくに HO/XB は1の周りをウロウロしているのでコードも簡単に書けそうな予感．
+ （コード説明，ビルドテスト）


```python
"""
Tokyo Quantopian User Group handson Vol4
strategy4.py
アルゴリズムその4：クラックスプレッド
https://github.com/drillan/quantopian/blob/master/driller/Crack_spread-Oil01.ipynb

2015/1/1−2018/6/1

"""
import pandas as pd
from quantopian.algorithm import order_optimal_portfolio
import quantopian.optimize as opt
from zipline.utils.calendars import get_calendar

def initialize(context):
    sym1 = 'HO'
    sym2 = 'XB'

    context.sym1 = continuous_future(sym1, roll='calendar', offset=3)
    context.sym2 = continuous_future(sym2, roll='calendar', offset=3)

    # 標準偏差をはかる日数
    context.std_term = 20
    # ホールドする日数（アルゴリズムでは使っていない，カウントだけして，利用出来るようにしておく）
    context.holding_days = 0
    # 満期日までの残存期間を指定（これもアルゴリズムでは使っていない）
    context.n_days_before_expired = 5
    # フラグ
    context.long_spread = False
    context.short_spread = False

    schedule_function(my_rebalance)
    schedule_function(my_record)

def get_ratio(context, data):
    hist = data.history([context.sym1, context.sym2],
                        fields ='price',
                        bar_count = context.std_term,
                        frequency = '1d')

    ratio = hist[context.sym1] / hist[context.sym2]
    std = ratio.std()
    return ratio[-1] + std

def is_near_expiration(contract, n):
    # .expiration_date このコントラクトの満期日を取得
    return (contract.expiration_date - get_datetime()).days < n

def get_my_position(cpp):
    l = list()
    for k, v in cpp.iteritems():
        d = {'symbol': k.symbol,
             'amount': v.amount,
             'average value': v.cost_basis,
             'last_sale_price': v.last_sale_price,
             'current value': v.last_sale_price*v.amount*k.multiplier,
             'multiplier':k.multiplier,
             'PL': (v.last_sale_price/v.cost_basis-1)*v.amount*k.multiplier,
            # 'exp date': k.expiration_date.strftime("%Y%m%d")
            }
        l.append(d)
    df = pd.DataFrame(l)
    df = df.set_index('symbol')
    log.info(df)
    return df

def my_rebalance(context, data):

    # もしポジションを持っている場合は，ログ出力．（ここはアルゴリズムには不要）
    cpp = context.portfolio.positions
    if cpp:
        df = get_my_position(cpp)
        log.info('PL: {}'.format(df['PL'].sum()))
    else:
        log.info("No position")

    # target_weight
    target_weight = {}

    # 今日のコントラクトを取得
    contract_sym1 = data.current(context.sym1, 'contract')
    contract_sym2 = data.current(context.sym2, 'contract')
    # 今日のRatioを取得
    context.ratio = get_ratio(context, data)

    # ポジションクローズ１
    if (context.ratio < 1.0) and context.long_spread:
        target_weight[contract_sym1] = 0
        target_weight[contract_sym2] = 0
        context.holding_days = 0
        context.long_spread = False

    # ポジションクローズ2
    elif (context.ratio > 1) and context.short_spread:
        target_weight[contract_sym1] = 0
        target_weight[contract_sym2] = 0
        context.holding_days = 0
        context.short_spread = False

    # ショートポジション注文
    elif context.ratio > 1.1:
        target_weight[contract_sym1] = -0.5
        target_weight[contract_sym2] = 0.5
        context.long_spread = True
        context.holding_days = context.holding_days + 1

    # elif context.ratio < 0.96:
    #     target_weight[contract_sym1] = 0.5
    #     target_weight[contract_sym2] = -0.5
    #     context.short_spread = True
    #     context.holding_days = context.holding_days + 1

    if target_weight:
        order_optimal_portfolio(
            opt.TargetWeights(target_weight),
            constraints=[]
        )

def my_record(context, data):
    record(ratio=context.ratio)

```


---
## カレンダースプレッド

### 特徴を見てみる


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



---
参照

[Quantopian Futures API Tutorial](https://www.quantopian.com/posts/quantopian-futures-api-tutorial)
[Continuous Future Data Lifespans](https://goo.gl/KbFxjx)
[available futures](https://www.quantopian.com/help#available-futures)






