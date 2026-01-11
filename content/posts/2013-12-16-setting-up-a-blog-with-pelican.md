静的サイトジェネレータ Pelican で Blog を書こう
###############################################

:date: 2013-12-16 02:10
:slug: setting-up-a-blog-with-pelican
:tags: Debian, Linux, Pelican, Python

.. _Pelican: http://getpelican.com/
.. _octopress: http://octopress.org/

先日ブログを Wordpress から Pelican_ に移行しました。

Wordpress は機能が充実していて、使っていてスゲー！と思うところは多々あったのですが、利用者が多いせいか、なにかと攻撃の的になりやすく、セキュリティ面での管理にわずらわしさを感じていました。

この辺りの不満は静的サイトジェネレータで解消できそうだったので、そのうちのひとつである Pelican_ に移行してみました。

Pelican_ は Python で書かれた OSS です。似たようなツールに Ruby で書かれた octopress_ というのがあるようですが、特に比較検討はしていません。

Pelican_ は Markdown 形式や reStructuredText 形式でブログ記事を書けて、そこから HTML ファイルを自動生成できます。ちょっとした作業メモなどを普段からそれらの形式で書いていれば、サクッとブログ記事に発展させられるところがいいですね。

実際に Wordpress から移行してみてわかったメリット/デメリットは以下が挙げられます。

- メリット

  - セキュリティ面の安心感がある
  - とっつき易いマークアップ言語で記事が書ける
  - サーバのソフトウェアをアップデートしてもブログが壊れない
  - 記事作成＆推敲をオフラインでできる
  - 記事閲覧時のレスポンスが早い
  - 記事をバックアップしやすい
  - テーマをカスタマイズしやすい

- デメリット

  - 公式ドキュメントは充実しているが Wordperss と較べ情報量は圧倒的に少ない
  - テーマやプラグインが少なく何かと自力解決が必要
  - 記事生成処理がクライアントに依存する
  - コメント欄が Disqus (メリット？)


インストール
------------

公式ドキュメントは `こちら <http://docs.getpelican.com/>`_ にあります。

僕は Debian の python-pelican パッケージをインストールして使っていますが、Pelican の公式ドキュメントでは virtualenv を使った仮想環境内で利用することが推奨されています。ここでは公式ドキュメントに倣ってインストールします。

.. code-block:: bash

  $ sudo apt-get install virtualenv
  $ mkdir ~/python-virtualenvs
  $ virtualenv ~/python-virtualenvs/pelican
  $ cd ~/python-virtualenvs/pelican
  $ . bin/activate
  $ pip install pelican

これでインストールは完了です。


スタートアップ
--------------

Pelican には pelican-quickstart という対話式のコマンドが用意されており、このコマンドを実行すると、サイト生成に最低限必要なファイルが自動生成されます。

試しにブログ用のディレクトリを作成し、pelican-quickstart を実行してみましょう。このときの設定は後で変更できますので、迷ったら適当に設定してしまってオッケーです。

.. code-block:: bash

  $ mkdir myblog
  $ cd myblog
  $ pelican-quickstart
  Welcome to pelican-quickstart v3.3.0.

  This script will help you create a new Pelican-based website.

  Please answer the following questions so this script can generate the files
  needed by Pelican.


  > Where do you want to create your new web site? [.]
  > What will be the title of this web site? Blog-Title
  > Who will be the author of this web site? Yusaku
  > What will be the default language of this web site? [en] ja
  > Do you want to specify a URL prefix? e.g., http://example.com   (Y/n)
  > What is your URL prefix? (see above example; no trailing slash) http://blog.tekito.org
  > Do you want to enable article pagination? (Y/n)
  > How many articles per page do you want? [10]
  > Do you want to generate a Fabfile/Makefile to automate generation and publishing? (Y/n)
  > Do you want an auto-reload & simpleHTTP script to assist with theme and site development? (Y/n)
  > Do you want to upload your website using FTP? (y/N)
  > Do you want to upload your website using SSH? (y/N)
  > Do you want to upload your website using Dropbox? (y/N)
  > Do you want to upload your website using S3? (y/N)
  > Do you want to upload your website using Rackspace Cloud Files? (y/N)
  Done. Your new project is available at /home/foo/myblog

Pelican v3.3.0 では pelican-quickstart 実行後に下記ファイル/ディレクトリが生成されました。

.. code-block:: bash

  $ ls -1
  Makefile
  content
  develop_server.sh
  fabfile.py
  output
  pelicanconf.py
  publishconf.py

pelicanconf.py が主要な設定ファイルです。サイト名、テーマ、言語やタイムゾーン、ディレクトリ構成などなど、たいていの設定はここに記述します。

また、Makefile も生成されます。make コマンドを使って HTML ファイルの自動生成やアップロードを行います。


記事を書いてみる
----------------

Pelican では以下の流れでブログ記事を作成します。

1) Markdown や reStructuredText でマークアップした記事を作成する
2) ローカルマシンで HTML 化して記事を確認する
3) リモートマシンにアップロードする

順を追ってみていきましょう。


1. Markdown や reStructuredText でマークアップした記事を作成する
================================================================

Pelican は Markdown, reStructuredText, AsciiDoc 形式に対応しています。どれで書くかは自由です。

ブログ記事は content ディレクトリ以下に作成します。

試しに content/my-first-article.rst に下記ファイルを作成してみましょう。

.. code-block:: text

  Pelican でブログ
  ################

  :date: 2013-12-15 00:00
  :slug: my-first-article
  :category: misc
  :tags: python

  これは reStructuredText 形式です。


2. ローカルマシンで HTML 化して記事を確認する
=============================================

記事を作成したら make html を実行して HTML ファイルを生成してみましょう。HTML ファイルは output ディレクトリ以下に生成されます。

.. code-block:: bash

  $ make html
  pelican /home/foo/myblog/content -o /home/foo/myblog/output -s /home/foo/myblog/pelicanconf.py
  Done: Processed 1 articles and 0 pages in 0.10 seconds.

マークアップにおかしな点があれば、この段階で ERROR や WARNING が発生します。

次に HTML 化した記事を確認するために make serve を実行し、ローカルマシンに HTTP サーバを建てましょう。

.. code-block:: bash

  $ make serve
  cd /home/foo/myblog/output && python -m pelican.server
  serving at port 8000

デフォルトのポート番号は 8000 になっています。http://localhost:8000 にアクセスすると生成されたサイトが表示されると思います。

.. image:: |filename|/data/2013/12/16/my-first-article.jpg
           :width: 500
           :target: |filename|/data/2013/12/16/my-first-article.jpg
           :alt: 自動生成されたサイト

記事のドラフト作成から推敲まで手元でできます。この辺りがすごく便利ですね！


3. リモートマシンにアップロードする
===================================

アップロードする前にやることがひとつあります。

make html で生成される HTML ファイルはローカルマシンで閲覧するためのものです。これをリモートにアップロードする前に make publish を実行し、公開用の HTML ファイルにする必要があります。

.. code-block:: bash

  $ make publish
  pelican /home/foo/myblog/content -o /home/foo/myblog/output -s /home/foo/myblog/publishconf.py
  Done: Processed 1 articles and 0 pages in 0.11 seconds.

make publish を実行すると、サイト内のリンクが http://localhost から実際に使う URL に切り替わり、公開用の HTML ファイルが生成されます。(make html と make publish では読み込む設定ファイルが異なります。前者は pelicanconf.py で後者は pelicanconf.py です。設定ファイルの設定値によって切り替わっています。)

make publish を実行したら、まずは試験的にリモートマシンの適当なディレクトリにアップロードしてみるとよいと思います。

アップロード方法は、どこにどうやってアップロードするかによって変わりますが、基本的にこれまでの流れ通り make コマンドで行えます。make help を参考にしましょう。

SSH, rsync, FTP でアップロードする場合は Makefile の SSH_*, FTP_* 辺りの変数を書き換えます。以下は rsync でのアップロード例です。

.. code-block:: bash

  $ cat Makefile
  ..
  SSH_HOST=tekito.org
  SSH_PORT=22
  SSH_USER=pelican
  SSH_TARGET_DIR=/home/pelican/blog
  ..

.. code-block:: bash

  $ make rsync_upload
  (ビヨーンと転送)

SSH, FTP のほか GitHub Pages や Amazon S3 にも対応しているようです。


おわりに
--------

駆け足でしたが Pelican のインストールから、自動生成したサイトのアップロードまでの流れをご紹介しました。

Pelican は Wordperss や Movable Type などの他のブログツールと較べると、やや技術的な敷居が高いツールではありますが、ログインページがない安心感や、好きなエディタを使ってとっつき易いマークアップ言語でブログ記事が書ける点が、個人的にはとても魅力を感じています。

開発も活発なようですので、これからの発展にも期待です！
