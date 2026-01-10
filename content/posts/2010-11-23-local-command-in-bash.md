+++
date = 2010-11-23T19:52:00+09:00
title = "bash のビルトインコマンド “local” について"
+++

実験のまとめ。

前置き
------
`bash` では特に指定しない限り、 **変数は常にグローバル** な変数として扱われてしまいます。

```shell
#!/bin/bash

function foo() {
    hoge="foo"
}

foo
echo $hoge # foo と出力される
```

変数をつくるたびに、グローバルな領域を汚染していくのはよろしくないので、ここではグローバル化の対策として `bash` のビルトインコマンド `local` を使用します。

```shell
#!/bin/bash

function foo() {
    local hoge="foo"
}

foo
echo $hoge # 空出力
```

このように、明示的にローカルな変数だよーって宣言してあげると、関数をでたときには忘れる。なんだか逆な気がしますが...。

本題
----
`bash` では **for ループに使用する変数** や、**read で受け取る変数** も他の変数と同様に、特に指定しない限りグローバル変数になります。

```shell
#!/bin/bash

function foo() {
    for hoge in `echo one two three`; do
        :
    done
}

function bar() {
    read piyo
}

foo
bar
echo $hoge # three と出力される
echo $piyo # 入力された文字列が出力される
```

対策として `local` を使用したいところですが、 **for や read の変数名の前に local を置くと** 残念ながら **構文エラー** になってしまいます。

```shell
#!/bin/bash

function foo() {
    for local hoge in `echo one two three`; do # syntax error
        :
    done
}

function bar() {
    read local piyo # syntax error
}
```

どうしたものかと頭を抱えるところです...。

でも実はそのあたりもちゃんと対策されていて、一度 `local` コマンドに名前を渡すと、 **同じスコープ内ではその変数名はずっとローカル** として扱う特性があります。具体的には以下のように。

```shell
#!/bin/bash

function foo() {
    local hoge # 一度 local と宣言したら
    hoge="foo" # global っぽく代入しても local として扱われる
}

foo
echo $hoge # 空出力
```

すこし誤解を招きそうなこの特性ですが、これを利用して先程の `for` や `read` の問題も解決できます。

```shell
#!/bin/bash

function foo() {
    local hoge
    for hoge in `echo one two three`; do
        :
    done
}

function bar() {
    local piyo
    read bar
}

foo
bar
echo $hoge # 空出力
echo $piyo # 空出力
```

余談
----
実験中に発見したのですが、`zsh` だと以下は構文エラーになりません。

```shell
#!/bin/zsh

function foo() {
    for local hoge in `echo one two three`; do
        :
    done
}

function bar() {
    read local piyo
}
```

`bash` も将来的にはできるようになるかもしれませんね。
