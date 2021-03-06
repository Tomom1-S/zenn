---
title: "Nand2Tetris読書会（6章）"
emoji: "📚"
type: "idea"
topics: ["読書会", "勉強会"]
published: true
---

# 概要

[Nand2Tetris読書会](https://zenn.dev/tomom1_s/articles/nand2tetris-00) を開催しています。
今回取り上げるのは、6章『アセンブラ』です。

前の記事は [こちら](https://zenn.dev/tomom1_s/articles/nand2tetris-05)。

# 内容

まずは前の章の復習として…

- アセンブラ: アセンブリから機械語(バイナリ)に変換するプログラム
- アセンブリ言語: 機械語の命令コードの組み合わせ

アセンブラを書くときの大きな問題は、アセンブリプログラムでシンボルでメモリアドレスを参照することで生じる。
このとき、ユーザーの定義したシンボルを物理メモリアドレスに変換できないといけないため、シンボルテーブルが利用される。

アセンブラとアセンブリ言語の関係はこんなイメージ。
```
LOAD R3, 7  ← アセンブリ言語(≒シンボル)
   ↓
[アセンブラ]
   ↓
1100....11  ← バイナリ
```

アセンブラによるアセンブリ言語→バイナリの変換は、以下のようなステップで実施される。
後ほど、より詳細な手順が説明される。
1. 各アセンブリコマンドを基本要素に分解する(LOAD, R3, 7 とか？)
1. 各要素をそれぞれバイナリコードに置換する
1. 置換結果をつなぎ合わせて、ハードウェアが実行するバイナリ命令を作成する

シンボルを使うと、アドレス値を直接指定する代わりに別の文字列でアドレス値を指定できる。
シンボルが利用されるのは、一般的に以下の2つの場合。
- 変数: 変数に自動で割り当てられたメモリアドレスをシンボルで指す。
- ラベル: プログラム中の位置をシンボルで指す。

シンボルは、あらかじめ(アセンブリ言語の仕様で?)決められたものの他に、ユーザーが定義する任意のシンボルもある。
ユーザー定義のシンボルと実際のアドレス値を紐づけるためには、**シンボル解決** が必要である。
1. シンボルテーブルを作る  
   新しいシンボルが出てきたらシンボルテーブルにシンボルとそのシンボルのメモリアドレスを追加する。
1. プログラムを変換する
   1で作ったシンボルテーブルを参照しながら、プログラム内のシンボルをアドレス値に置き換えていく。

アセンブラは、アセンブリコマンドを入力として受け取り、バイナリ命令を生成して出力する変換プログラムである。
生成されたコードは、コンピュータのメモリに読み込まれてハードウェアが実行する。

アセンブラでやる処理: 
1. アセンブリコマンドを構文解析して分割
1. 分割した領域ごとに、機械語のビットを生成
1. シンボルを直接メモリアドレスを指定するための数字に置き換え(=シンボル解決)
1. 領域ごとのバイナリコードを組み合わせて、完全な機械語命令を作成

アセンブラの実装では、以下の4つのモジュールを実装していく。
- Parserモジュール: アセンブリコマンドを基本要素(フィールド、シンボル)に分解。
  - 入力コードへのアクセスのカプセル化
  - アセンブリコマンドを読んでパース
  - コマンドの要素へ簡単にアクセスできるルーティンを提供
  - 空白文字とコメントを削除
- Codeモジュール: アセンブリのニーモニックをバイナリに変換。
- SymbolTableモジュール: シンボルを実際のアドレスへと解決。
- メインプログラム: 変換処理の全てを実行。

シンボルを含むプログラムをアセンブラで変換したいときは…

1. 初期化: 定義済みシンボルと対応するRAMアドレスを含むシンボルテーブルを用意
1. ラベルとROMアドレスの対応付け
   1. プログラムを1行ずつ読んで、現在の命令が読み込まれるROMアドレスの番号を保持
   1. ラベルシンボルが出てきたら、その次のコマンドが格納されるROMアドレスに対応付ける
1. シンボルをパースする
   1. A命令のシンボルに出くわしたら、シンボルテーブルで同名のシンボルを探す
   1. シンボルテーブルに目的のシンボルが見つかれば対応する数値に置き換えて終了
   1. テーブルにシンボルが見つからなければ、シンボルとRAMアドレスのペアをシンボルテーブルに追加

マクロコマンドとは、一連の機械語命令に名前をつけたものである。
マクロコマンドを使うと、よく行う操作を単純化できる。

## 予習メモ

### 「高水準プログラムのコード中にアセンブリ言語によるコードを埋め込む」とは？

インラインアセンブラと呼ばれるもので、最適化やシステムコールのために利用される。

https://ja.wikipedia.org/wiki/インラインアセンブラ

C言語の場合は、「ここからアセンブリ言語を記述しますよ」というキーワードを使って、インラインアセンブラを実現している。
ブロックを指定すれば複数行に渡る命令も書ける。

https://codezine.jp/article/detail/393

## ディスカッションメモ

### 任意の言語でアセンブリ言語を機械語に変換するアセンブラを作ったが、アセンブラを機械語に翻訳するためのアセンブラも必要になるのでは？

これはブートストラップ問題と呼ばれる問題である。
典型例として、ある言語のコンパイラをその言語で作成した場合に、そのコンパイラのコンパイルをどうやったのかというものがある。

https://ja.wikipedia.org/wiki/ブートストラップ問題

この問題はブートストラップ法と呼ばれる方法で実現されている。
最初のごく簡単な単位のコンパイラのソースを人手で「ハンドコンパイル」すれば、それを使って少し複雑なコンパイラを書ける。
それを繰り返していくと、徐々に機能が追加されたコンパイラが得られるというものである。
こちらの記事に書かれていた説明が分かりやすかった。

https://note.com/ruiu/n/n44e161a0c243

### インタープリタ方式では、未知のシンボルが出てきたらエラーになってしまう？

- [コンパイラ方式](https://ja.wikipedia.org/wiki/コンパイラ): ソースプログラムを一度機械語に翻訳して、その機械語のプログラムを実行する方式。実行速度は速いが、コンパイルした機械語のプログラムが環境によっては動かないことがある。  
  [例] C, Goなど
- [インタープリタ方式](https://ja.wikipedia.org/wiki/インタプリタ): ソースプログラムを1行ずつ読み込んで、機械語に翻訳・実行する方式。作成したソースを直ちに実行できるが、翻訳しながら実行するため実行速度はコンパイラ方式に劣る。  
  [例] Ruby, Pythonなど

インタープリタ方式の中には、都度の変換で実行速度が遅くなるという欠点を解消するため、一度全部ファイルを中間言語に変換してそれを1行ずつ実行することで高速化を図るものもある。
そのようなインタープリタの場合、一度ファイルの中身を全て見ているのでシンボル解決ができなくなることはない。
一方、完全に1行ずつ変換・実行をする [REPL](https://ja.wikipedia.org/wiki/REPL) では、未定義シンボルでエラーが発生する。

# 感想

今まではコマンドの力を借りて何気なくやっていたコンパイルの裏で、こんなことが起こっていたのかと気づける章でした。

演習のコードはTypeScriptで書いてみました。
あまり書き慣れていない言語なので、ファイルの読み込みだけでもかなり時間がかかってしまったし、おそらく最善の方法にはなっていない気がしますが、慣れていないなりにいろいろと試行錯誤するのは楽しかったです。
コマンドタイプの判別に正規表現を活用したのですが、なかなかうまくいかずに試行錯誤していました。

# 最後に

『[Nand2Tetris読書会始めました](https://zenn.dev/tomom1_s/articles/nand2tetris-00)』の記事でも紹介していますが、読み進めているのはこちらの本です。
https://www.oreilly.co.jp/books/9784873117126/

初学者なりに書籍やその他に調べた内容をまとめていますが、理解が足りておらず間違ったことを書いているかもしれません。
そのような箇所を見つけた場合はコメントなどで指摘していただけると助かります。

次の記事は [こちら](https://zenn.dev/tomom1_s/articles/nand2tetris-07)。
