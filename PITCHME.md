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
  + 先物編
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
注意：
1. `def initialize(context)` は，バックテストスタート時にまず実行される関数．アルゴリズムの設定や変数の作成などはココで行う．
  + 銘柄設定
  + スケジュール
  + トレードフラグ等
  
1. <font color=red>【重要】</font>グローバル変数は必ず `context` の属性としてつくる
  + ダメな例
```python

my_future = future_symbol("CLF17")
def initialize(context):
    ## 原油２０１７年１月限 (10/01/2016~12/20/2016)
    schedule_function(my_rebalance, 
                      date_rule=date_rules.every_day(), 
                      time_rule=time_rules.market_open(hours=1))
```  


---
### 先物編
---
### バックテスト結果の見方 Build Algorithm
---
### Full Backtest
---
### Tear Sheet
--- 
### Future Algorithm クラッシュスプレッド
---
### カレンダースプレッド
--- 
### その他
