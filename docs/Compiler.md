# コンパイル

C++のコンパイラーとしては、GCC(GNU Compiler Collection)と Clang(クラン)がある
GCC を使ってコンパイルする場合は以下のコマンドを実行する

```shell
g++ その他のオプション -o 出力するファイル名 ソースファイル名
```

コンパイルしたコードはファイル名をコマンド実行すると実行できる

```shell
./出力したファイル名
```

コンパイラオプション

- `-std=`：C++の規格を指定（例：`-std=c++17`）
- `-Wall`：コンパイラの便利な警告メッセージをほとんど全てを有効にする
- `pendantic-errors`：C++の規格を厳格に守る。規格に違反しているコードはコンパイルエラー扱いになる

```shell
g++ -std=c++17 -Wall --pedantic-errors -o 出力ファイル名 入力ファイル名
```

# ヘッダーファイルの省略

通常はファイル冒頭に`#include`を記載してライブラリを読み込む
`#include <iostream>`こう書いた場合、ヘッダーファイル iostream を取り込む

毎回、`#include`を追加するのは大変なので、標準ライブラリをまとめて`#include`したヘッダーファイルを作成し、それを`#include`することでまとめてライブラリを取り込むようにする

`#include`をまとめたヘッダーファイル all.h を作成して、以下のように書けば他のヘッダーファイルを`#include`する必要がなくなる

```cpp
#include "all.h"

// その他のコード
```

ファイルに毎回`#include`をするのも大変なため、コンパイルオプションの`-include`でコンパイル実行時にヘッダーファイルを読み込むことができる

```shell
g++ -include all.h -o 出力ファイル名 入力ファイル名
```

# コンパイル済みヘッダー

C++のコンパイル実行には時間がかかるため、プログラムで変更しないファイルを事前にコンパイルしておくと、コンパイル時間の短縮になる
事前にコンパイルしたヘッダーファイルのことをコンパイル済みヘッダーという

以下のコマンドを実行し、コンパイル済みヘッダーファイルの`ヘッダーファイル名.gch` を作成する

```shell
g++ -std=c++17 -Wall --pedantic-errors -x c++-header -o 出力ファイル(all.h.gch) 入力ファイル(all.h)
```

次に、ヘッダーファイルを使うコマンドを実行した時に同名の`.gch`が存在する場合、コンパイル済みヘッダーファイルが使用され処理を短縮する

```shell
g++ -std=c++17 -Wall --pedantic-errors -include all.h -o program main.cpp
```

# ビルドシステム

何千ものソースファイルをコンパイルする必要がある場合、`GUN Make`というビルドシステムを使用することで簡単なコマンドで依存関係を解決してのコンパイルが可能になる

`GUN Make`を使用することで以下のコマンドでコンパイルなどができるようになる

```shell
make // コンパイルと再コンパイル
make run // プログラムを実行（これを使うことで共通した方法でプログラムを実行できる）
make clean // ソースファイルから生成されたプログラムなどのファイルを全て削除
```

## Makefile の書き方

まず作成例として、以下のコマンドを実行する Makefile を作成する

```shell
cat source > program
```

上記のコマンドを Makefile で書くと以下のようになる

```Makefile
program:source
  cat source > program
```

Makefile が作成できたら、make コマンドを実行する

```shell
make
```

ファイル source01, source02, source03 の中身を順番で連結して source ファイルを生成する場合は以下のように書く

```Makefile
program : source
  cat source > program

// 以下のコードを新たに追加
source : source01 source02 source03
  cat source01 source02 source03 > source
```

上記のコードをコマンドで実行する場合、 source01, source02, source03 を source に連結した後に `cat source > program` を実行する必要があるが、Makefile であれば make コマンドを実行することで依存関係を自動で解決してくれる

また、すでに make を実行した後でもう一度 make を実行すると、`make: 'program' is up to date.`という「program は最新だ」というメッセージが表示される

make はファイルのタイムスタンプを調べて実行するため、source02 ファイルが更新された場合、再度 source ファイルの作成から実行し直す

### コメント

`#`で Makefile にコメントを書くことができる

```Makefile
# programを生成するルール
program : source
  cat source > program

# sourceを生成するルール
source : source01 source02 source03
  cat source01 source02 source03 > source
```

### 変数

Makefile には変数を書くことができる

```Makefile
variable = foobar

target: $(variable)
```

これは、

```Makefile
target: foobar
```

と書いたのと同じになる

変数は左側に変数名、右側に変数の内容を書く
変数を使う時は、`$(変数名)`のように`$()`で包む
