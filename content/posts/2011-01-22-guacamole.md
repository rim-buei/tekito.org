+++
date = 2011-01-22T04:26:00+09:00
title = "Guacamole でウェブブラウザからデスクトップ環境を使ってみた"
+++

[Guacamole](https://guacamole.apache.org/) はHTML5とAjaxで実現されたVNCクライアントです。

ウェブブラウザさえあれば、どこからでもアクセスできるし便利そうなので試してみました。

![Guacamole](guacamole.jpg)

(Guacamoleを使ってノートPCからデスクトップPCにつないでみた様子。Chromeの中でChromeを立ち上げてしまうセンスの無さ...。)

実際にGuacamoleを使ってグリグリ動かしている動画が [公式サイト](https://guacamole.apache.org/) にもあるので、そちらも参照のこと。

Guacamoleは日本語だと「ガカモレ」とか「ワカモレ」と発音するようです。

基本的に [公式サイト](https://guacamole.apache.org/) に載ってるインストール手順通りでハマるところはありませんでしたが、ググっても日本語記事が少ないようなのでメモ。

はじめに
--------
Guacamoleを利用するためには `vncserver`, `apache`, `tomcat` のほか **デスクトップ環境** が必要です。デスクトップ環境の構築方法については触れないので、その辺りが分からない方はググってください。

また、この記事はとりあえず実験的に動かすことを目的にしているので、継続して利用するのであれば設定を見直す必要があります。

インストール手順
----------------
まずは必要なパッケージをインストールする。(debian squeezeの場合)

```text
$ sudo apt-get install vnc4server apache2 tomcat6
```

1. Apache Tomcat

   ApacheとTomcatはインストール直後に既に実行されていたが、一応確認。

   ```text
   $ /etc/init.d/apache2 status
   Apache2 is running (pid 1250).
   $ /etc/init.d/tomcat6 status
   Tomcat servlet engine is running with pid 2601.
   ```

   `netstat` すると8080番でTomcatがLISTENしているので、アクセスしてみる。

   ```text
   $ curl http://localhost:8080/
   <?xml version="1.0" encoding="ISO-8859-1"?>
   <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
      "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
   <html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en">
   <head>
       <title>Apache Tomcat</title>
   </head>

   <body>
   <h1>It works !</h1>
   ```

   ぶじに実行できている。

2. VNCサーバ

   VNCサーバを起動する。

   ```text
   $ vncserver
   ```

   初回起動時にはパスワードを問われるので、適当に設定。このパスワードはGuacamoleの設定でも使用します。

   ```text
   You will require a password to access your desktops.

   Password: [password]
   Verify: [password]
   ```

   `vncserver` は起動時のオプションで解像度や色数も変えられます。使いづらかったらその辺りをいじってみましょう。

3. Guacamole

   ここまできたら、いよいよGuacamoleのダウンロードと設定です。ダウンロードは [このへん](http://sourceforge.net/projects/guacamole/files/) からお好きなものを。

   ぼくは `guacamole-0.3.0rc1.tar.gz` (最新版:2011/1/21時点) をつかいました。

   ダウンロードしたら解凍して中身を見てみる。

   ```text
   $ tar xf guacamole-0.3.0rc1.tar.gz
   $ ls guacamole-0.3.0rc1
   LICENSE.txt  guacamole-src.tar  guacamole-users.xml  guacamole.war  guacamole.xml
   ```

   この中で使うのは `guacamole-users.xml`, `guacamole.war`, `guacamole.xml` の3ファイルです。

   3.1. `guacamole.war`

   まずは `guacamole.war` を任意の場所に設置します。

   設置場所のパスは `guacamole.xml` で指定することになりますが、ここでは `guacamole.xml` の初期設定で指定されている `/var/lib/guacamole/guacamole.war` に置くことにします。

   ```text
   $ sudo mkdir -p /var/lib/guacamole
   $ sudo cp guacamole.war /var/lib/guacamole/
   ```

   3.2 `guacamole-users.xml`

   つぎに `guacamole-users.xml` を `/var/lib/tomcat6/conf/` にコピーして設定を編集します。

   このファイルにはBASIC認証でのアクセス時に使用するユーザーとパスワードの設定を行います。

   ```text
   $ sudo cp guacamole-users.xml /var/lib/tomcat6/conf/
   $ sudo nano -wk /var/lib/tomcat6/conf/guacamole-users.xml
   ```

   5行目の `username=""`, `password=""` を適当に設定しましょう。

   ```xml {linenos=inline hl_lines=5}
   <?xml version='1.0' encoding='utf-8'?>

   <tomcat-users>
     <role rolename="guacamole"/>
     <user username="guacamole" password="changeme" roles="guacamole"/>
   </tomcat-users>
   ```

   3.3 `guacamole.xml`

   さいごに `guacamole.xml` をTomcatの `./conf/Catalina/HOSTNAME/` 以下に設置します。

   `HOSTNAME` は実行するホストに応じて決める。ぼくはとりあえず動作確認のため localhost に置きました。

   ```text
   $ sudo cp guacamole.xml /var/lib/tomcat6/conf/Catalina/localhost/
   ```

   `guacamole.xml` の編集の前に、VNCサーバが待ち受けてるポート番号を調べます。

   ```text
   $ sudo netstat -anp --tcp
   稼働中のインターネット接続 (サーバと確立)
   Proto 受信-Q 送信-Q 内部アドレス            外部アドレス            状態        PID/Program name
   tcp        0      0 0.0.0.0:6001            0.0.0.0:**               LISTEN      4752/Xvnc4
   tcp        0      0 0.0.0.0:22              0.0.0.0:**               LISTEN      1405/sshd
   tcp        0      0 127.0.0.1:25            0.0.0.0:**               LISTEN      1619/exim4
   tcp6       0      0 :::8080                 :::**                    LISTEN      4825/java
   tcp6       0      0 :::80                   :::**                    LISTEN      1250/apache2
   tcp6       0      0 :::22                   :::**                    LISTEN      1405/sshd
   tcp6       0      0 ::1:25                  :::**                    LISTEN      1619/exim4
   tcp6       0      0 127.0.0.1:8005          :::**                    LISTEN      4825/java
   tcp6       0      0 :::5901                 :::**                    LISTEN      4752/Xvnc4
   ...
   ```

   Xvnc4ってのがVNCサーバ。

   tcpとtcp6の両方で待ち受けていることがわかりますが、javaの待ち受けがtcp6の8080番なので、VNCサーバもそれに合わせてtcp6のほう(5901番)を使います。

   VNCサーバのポート番号はわかったので `guacamole.xml` を編集しましょう。

   ```text {linenos=inline hl_lines=[25,26,"33-35"]}
   $ sudo nano -wk /var/lib/tomcat6/conf/Catalina/localhost/guacamole.xml
   <?xml version="1.0" encoding="UTF-8"?>

   <!--
       Guacamole - Pure JavaScript/HTML VNC Client
       Copyright (C) 2010  Michael Jumper

       This program is free software: you can redistribute it and/or modify
       it under the terms of the GNU Affero General Public License as published by
       the Free Software Foundation, either version 3 of the License, or
       (at your option) any later version.

       This program is distributed in the hope that it will be useful,
       but WITHOUT ANY WARRANTY; without even the implied warranty of
       MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
       GNU Affero General Public License for more details.

       You should have received a copy of the GNU Affero General Public License
       along with this program.  If not, see <http://www.gnu.org/licenses/>.
   -->

   <Context antiJARLocking="true" path="/guacamole" docBase=﻿"/var/lib/guacamole/guacamole.war">

       <!-- Change the lines below to match your VNC server -->
       <Parameter name="host" value="localhost"/>
       <Parameter name="port" value="5900"/>

       <!-- Password (VNC Authentication)

            Uncomment and change the line below if your VNC server is
            password protected. -->

       <!--
       <Parameter name="password" value="PASSWORD"/>
       -->
   ```

   name="host", name="port", name="password" の value を編集。

   `password` には `vncserver` 起動時に設定したパスワードを設定する。パスワードの設定は **コメントアウトされている** ので解除する必要があります。そこだけ注意。

これで設定は完了！
------------------

あとはTomcatを再起動してから http://localhost:8080/guacamole へアクセスすればおっけー。

```text
$ sudo /etc/init.d/tomcat6 restart
Stopping Tomcat servlet engine: tomcat6.
Starting Tomcat servlet engine: tomcat6.
```

使ってみた感想
--------------
レスポンスもなかなかいいし、設定も簡単なのでとっつきやすいです。あと、ブラウザにデスクトップが表示されるのはちょっと感動する。

操作性はデスクトップ環境にもよると思いますが、タイル型のWMとの相性はかなり悪め。この辺りは設定次第なのかもしれません。

重たい処理もやってみようと思い、ためしに Neverwinter Nights(PCゲーム) を動かしてみた。PCが非力なのもあってか、カックカクでゲームってレベルじゃない。色もおかしいし。

![Neverwinter Nights](nwn.jpg)

でも、ブラウザ内で3DのPCゲームが動くってすごいね。

Wineも使ってあれこれやってみたけど、そっちはうまくいかなかったのでまた今度挑戦しよう。

それにしてもGuacamoleって覚えづらい名前だ...。ブラウザから見に行く時に「http://localhost:8080/gua... gua... えーと、なんだっけ？」ってことが何度もあった。
