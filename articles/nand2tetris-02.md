---
title: "Nand2Tetris読書会（2章）"
emoji: "📚"
type: "idea"
topics: ["読書会", "勉強会"]
published: true
---

# 概要

[Nand2Tetris読書会](https://zenn.dev/tomom1_s/articles/nand2tetris-00) の第2回を開催しました。
今回取り上げるのは、2章『ブール算術』です。

# 内容

数を表現して算術演算を行う論理ゲートについて学習する。 2章では、1章で扱った論理ゲートから算術論理演算器（ALU）を作成する。
ALU回路は、コンピュータで行われる全ての算術演算と論理演算を行う。

デジタルコンピュータで実行される命令の多くは、2進数の加算に置き換えて考えることができる。

2進数加算では、ふたつの数を右（最下位ビット）から左（最上位ビット）へ桁ごとに足し合わせる。
最上位ビット同士を足し合わせたときにキャリービットが1となると、「オーバーフロー」となる。

n桁の2進数では $2^n$ 通りのビット配列が考えられ、これらが表す領域を二等分して正の数と負の数を表す。
符号付き整数を表現するために現在一般的に用いられているのは、**2の補数表現**（**基数の補数**）である。 
n桁の2進数 $x$ について $x+(-x)=2^n$ が成り立つ。

2の補数表現の性質:

- 最大値は $2^{n-1}-1$、最小値は $-2^{n-1}$。
- 正の数の最上位ビットは0、負の数の最上位ビットは1。
- $x$ の全ビットを反転させて1を足すと $-x$ になる。

2の補数で表した符号付き整数の和は、正の数と同様に計算できる。
したがって、ビット単位の加算ができれば正負にかかわらずに加算処理ができる。

2章で扱う加算器は以下の4種類。

- 半加算器: 2つのビットの和を計算
- 全加算器: 3つのビットの和を計算
- 加算器: 2つのnビットの和を計算
- インクリメンタ: 入力された数字に1を加算

ここまでで扱った加算器回路の仕様はどのコンピュータでも共通しているが、ALUの仕様はコンピュータプラットフォームによって異なる。
本章では [Hack](https://ja.wikipedia.org/wiki/Hack_(プログラミング言語)) 専用の回路を考える。

ALUの簡単な仕様は以下の通り。

- 入力: 16ビットデータ2つと、どの関数を実行するかを指定する6ビットの**制御ビット**
- 出力: 16ビットデータ1つ

## 予習メモ

### 算術演算と論理演算

- 算術演算: 数を扱う計算のこと。整数の加減乗除や、べき乗、商・余りの計算など。
- 論理演算: ブール関数を評価する演算のこと。1章で作った論理ゲートはこの演算をするゲートである。

### 1の補数

2の補数ではなく、[1の補数](https://ja.wikipedia.org/wiki/符号付数値表現#1の補数) で符号付き2進数を表現することもできる。
こちらは元の数の全ての桁のビットを反転させて、負の数を得る。
したがって、0を表現するビット配列は2通り存在し、表現できる範囲は $-2^{n-1}+1$ から $2^{n-1}-1$ となる。
3桁の2進数の場合、000と111はいずれも0を表し（+0と-0）、表現できるのは-3~3の範囲の数字となる。
1の補数で表された2つの数の和を計算する場合、二進数の加算をした結果、そのキャリーを最下位ビットに戻して加算する。

なぜ1の補数ではなく2の補数が使われているかについては、ここのページで書かれている回答が分かりやすかった。
https://stackoverflow.com/questions/1049722/what-is-2s-complement

他のページもいくつか見たところ、主に2つの理由があるようだった。
- 1の補数表現では、0が2つの形式で表現される。
- 2の補数表現を使った場合に比べて、加算処理が複雑。2の補数表現では、単純な加算だけで済ませられる。

## ディスカッションメモ

### 1の補数で0が2つの形式で表現されるデメリットは？

0という数に2パターンの表現方法があると「0と比較する」処理が少し面倒になる。
すなわち、「2パターン(今回は000と111)のどちらかと一致するならば0である」という処理をしないといけない。

### 小数はどう表現する？

コンピュータで小数を表現する方法はいくつかある。一般的に使われているのは浮動小数点数による表現。

- [固定小数点数](https://ja.wikipedia.org/wiki/固定小数点数): 小数点以上・以下の桁数を固定した表現。二進数で表現した場合の小数点以下のビットが、$2^{-1}$, $2^{-2}$, ...に対応する。ただし、十進数で表現できる数を正確に表現できない場合もある。
- [浮動小数点数](https://ja.wikipedia.org/wiki/浮動小数点数): 符号部・仮数部(整数部分が1桁の小数)・指数部(桁数を表す)を組み合わせた表現。

### インクリメンタ(1を加算)があるとなぜ便利？

プログラムカウンタの値の加算に使える。プログラムは、基本的にCPUの番地に収められた命令を順番に実行していくため、プログラムカウンタの値を1ずつ増やしていけばそれぞれの番地に収められた命令を正しい順序で実行できる。

プログラムカウンタについてはここの説明が分かりやすいと思った。

https://toshiba.semicon-storage.com/jp/semiconductor/knowledge/e-learning/micro-intro/chapter4/cpu-configuration-program-counter.html

### 一般的なALUに入っていそうな計算は？

本書で扱うALUでは限定的な計算のみが入っているが、一般的なコンピュータのALUでは別の演算処理もできるはず。その例として考えられるのは以下のような演算である。

- ビットシフト
- 乗算・除算など加算以外の算術演算
- 浮動小数点数の計算

古典的なALUの例として、[74181](https://ja.wikipedia.org/wiki/74181) がある。

### 論理設計における「効率さ」と「簡潔さ」が達成できたか、どう確認するとよい？

読書会の参加者が演習問題を解くときにやっていること:
- GitHubの他の人の解答を参考にする。
- 前に登場したものを利用しているか確認する。

### 図2-6(ALUの真理値表)で、64行から18行だけ取り上げたのはなぜ？

この本で扱うALUが実現する計算は、64パターンのうち18パターンとする。
18個以外の他のビット組み合わせになったときは気にしない、ということ。
18パターンに分けるだけなら5ビットあれば十分に表現できるが、この後の章の演習では処理の組み合わせを表す今回のビット組み合わせが扱いやすい。

# 感想

1の補数と2の補数の辺りは大学生のときに勉強したはずなのですが、そのときはなぜ2の補数の方が優れているのかあまりきちんと理解できていなかったなあと思いました。
当時はレポートやバイト、サークルなどでいっぱいいっぱいでしたね…

読書会では、コンピュータサイエンスをしっかり勉強された方が"あえて"出してくれた質問で理解が深まったり、演習問題の解答の違いを見比べて新たな発見があったりしました。

# 最後に

『[Nand2Tetris読書会始めました](https://zenn.dev/tomom1_s/articles/nand2tetris-00)』の記事でも紹介していますが、読み進めているのはこちらの本です。
https://www.oreilly.co.jp/books/9784873117126/

初学者なりに書籍やその他に調べた内容をまとめていますが、理解が足りておらず間違ったことを書いているかもしれません。
そのような箇所を見つけた場合はコメントなどで指摘していただけると助かります。

次の記事は [こちら](https://zenn.dev/tomom1_s/articles/nand2tetris-03)。
