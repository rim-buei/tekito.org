MXE + Qt を使って Linux で Windows GUI アプリをクロスビルドする
###############################################################

:date: 2013-10-18 00:00
:slug: cross-building-on-linux-for-windows-using-mxe-and-qt
:tags: Debian, Linux, MXE, Qt

.. image:: |filename|/data/2013/10/18/sample.jpg
           :width: 400
           :target: |filename|/data/2013/10/18/sample.jpg
           :alt: Hello, Windows!

環境
----
Debian GNU/Linux jessie amd-64 (Linux 3.10.11)

依存パッケージのインストール (Debian)
-------------------------------------
まずは `MXE のドキュメント <http://mxe.cc/#requirements-debian>`_ に従って、必要なパッケージをインストールします。

.. code-block:: text

                $ apt-get install autoconf automake bash bison bzip2 \
                                  cmake flex gettext git g++ intltool \
                                  libffi-dev libtool libltdl-dev libssl-dev \
                                  libxml-parser-perl make openssl patch perl \
                                  pkg-config scons sed unzip wget xz-utils

MXE で Windows 向けの Qt ビルド環境を作成する
---------------------------------------------
MXE は Debian のパッケージリポジトリにはありません。

ソースコードが Github にありますので git clone して make します。

.. code-block:: text

                $ git clone https://github.com/mxe/mxe.git
                $ cd mxe && make qt5
                (make qt すると Qt4 用の環境ができてしまうので make qt5 しました。)

ビルドにはとても時間がかかります。僕の環境では 1 時間くらいかかりました。

クロスビルドする
----------------
mxe ディレクトリ以下の実行ファイルやライブラリを使ってビルドします。

手順はパスを通して qmake -> make でできます。ビルドするときに qmake -project は必要ありませんでした。

.. code-block:: text

                $ export PATH="/path/to/mxe/usr/bin/:${PATH}"
                $ cd /path/to/your/qt-project/
                $ /path/to/mxe/usr/i686-pc-mingw32/qt5/bin/qmake && make

                (Qt4 をビルドした場合 qmake のパスが異なります)
                $ /path/to/mxe/usr/i686-pc-mingw32/qt/bin/qmake && make

ビルドに成功すると release/ 以下に .exe ファイルが生成されます。

.. code-block:: text

                $ ls release/*.exe
                release/sample.exe
                $ file release/sample.exe
                release/sample.exe: PE32 executable (GUI) Intel 80386 (stripped to external PDB), for MS Windows

サンプル
--------
クロスビルドを試すためのサンプルコードを Github に置きました。ウィンドウを表示するだけの簡単なものです。必要であればお使いください。

.. code-block:: text

                $ git clone https://github.com/yus4ku/qt-mxe-sample.git

所感
----
試しに Qt5 で WebKit を使ったアプリを作ってみましたが、MXE 側のライブラリが不足していてビルドできませんでした。make qt すると Qt4 の環境が作成されるあたり、まだ Qt5 は非推奨なのかもしれません。

Qt Creator でも同等のことができそう、且つ Qt 的には王道っぽいので、そっちも近日中に試してみようと思います。
