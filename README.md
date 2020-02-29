# Brutus

リバーシとチェスの中間のようなゲーム
<https://www.asatamin.com>

## GUI play

```bash:
python -m game.gui
```

## CUI play

```bash:
python -m game.play
```

## test

```bash:
python -m unittest
```

## Start learning (QLearning)

```bash:
python -m agent.model
```

## Generate histories of games(AlphaZero)

```bash:
python -m agent.mcts_self_play
```

## Learn from histories (AlphaZero)

```bash:
python -m agent.mcts_learn
```

## Repeatedly generate histories and learn (AlphaZero)

```bash:
python -m worker.self_play_and_learn
```


## 強化学習について(案)

まず、基準として次のような入出力のモデルを作りたい。

### 盤面データの読み込みの方法

入力データは2 \* 7 \* 5の配列(今は`arr`と呼ぶことにする)で

- `arr[0]`は黒(キング以外)の位置
- `arr[1]`は白(キング以外)の位置

という感じにしたい。

### 次の一手の出力の方法

長さ315の1次元配列であって、次の7 \* 5 \* 9の多次元配列`next`を`flatten`したものにしたい。

- 各要素は次の一手の有効さを表す確率とする。
- `next[i, j, drc]`は、位置`[i, j]`のコマを`DIRECTIONS[drc]`の方向に動かす確率とする。ただし、前に2コマすすめる手は`DIRECTIONS[8]`とする。
ただし、位置`[i, j]`は`GameState`を基準とし、`drc`は`Drc`の要素で方向を表す(どちらも`game/game_state.py`を参照のこと)。

動かし方としては、出力結果の手のうち、有効なものを確定的、あるいは確率的に選択する(自己対戦時は、毎回同じ手にならないよう確率的な方がいいだろう)。

### モデル選択と学習方法

まずはAlpha GOのように、CNNとかで、モデルを作ってみるのがいいのかなあと。ただし、あんまり大きなモデルにはしないほうがよさげ。

学習については、はじめのうちはランダムな相手(`game/game_state.py`に作成済み、勝利手を優先的に打つことも可能)と戦わせて、勝てるようになってきたら、自分と戦わせるようかな？

その報酬については、勝利に対して点を与える感じかな、、？なんかもっと細かく点を与えたほうがいいのか、、？

あとは、盤面に対し勝つ確率を予測する価値ネットワークとかモンテカルロ木探索とかやってみたいけど、とりま後回し？

### 保存の方法について

手分けして強化学習の方法を探るに当たって、学習のパラメータをjson、モデルのconfigをjson, 重みをh5形式で保存していきたいと思います。

すでに`agent/model.py`に実装されているものは、保存するように設計されているので、
できるかぎり真似する感じ(もっといい方法があれば教えてください!)でやって見てもらえると助かります。
今回は他の人のコピペで済むKerasではじめ、実装しましたが、pytorch等も使っていただいて構いません。同様の形式で何をやって何をやっていないかがわかるようにしていただきたいです。

## Gitの使い方について

Gitに慣れてない方も参加されると思います。Gitができる方、挑戦したい方は積極的に使っていただいて、そうでない方はSlack, Google Driveでコード、データを共有していただければと思います。

Gitですが、みなさんの様子を見て、共同作業にするか、フェッチしてもらって適宜プルリクを送ってもらうか、考えます

## ちょっと学習させてみて考えたこと

まず、報酬の与え方ですが、毎回の手でコマ数に対し与えるのと、終盤力をつけるために相手が近くにいるときは守る行動を取らせるとか、両方の与え方で学習させる必要があると感じました。

また、なかなか王手のときに守る手というのは学習が難しいようなので、そこに特殊な報酬を与える必要があると感じました(昔からあるゲーム木探索では怒らないんですが)。
