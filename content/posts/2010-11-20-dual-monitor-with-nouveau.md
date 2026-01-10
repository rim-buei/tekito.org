+++
date = 2010-11-20T18:27:00+09:00
title = "nouveau でデュアルモニタの設定をしてみた"
+++

nouveauはNVIDIAのVGA用の非公式ディスプレイドライバで、2.6.33からはカーネルにマージされているらしい。

デュアルモニタの設定でやることは **nouveauの導入** と **xorg.confの設定** だけ。

検索してもあまり日本語のサイトがヒットしなかったので、メモついでに記事にしときます。

nouveauの導入
-------------
お使いのディストリビューションに依る部分です。

fedora は 13 以降、debian は testing 以降であればメインリポジトリにあるようです。

ちなみに debian testing の netinst したら最初から適用されていました。

検索してもなければ、仕方がないのでソースからコンパイルしましょう。

- [nouveau Wiki - Source](http://nouveau.freedesktop.org/wiki/Source)

ただ、ソースからコンパイルするくらいなら、NVIDIA公式のドライバを使ってもいいかも...。

xorg.confの設定
---------------
1枚のグラフィックカードでデュアル出力した場合の設定例です。

以下、`xorg.conf` の設定全文。

`/etc/X11/xorg.conf`:
```text
Section "ServerLayout"
    identifier "Layout"
    screen     0 "screen0" 0 0
Endsection

Section "Monitor"
    Identifier "Monitor0"
    Option     "PreferredMode" "1920x1080"
    Option     "Primary" "True"
EndSection

Section "Monitor"
    Identifier "Monitor1"
    Option     "PreferredMode" "1920x1080"
    Option     "RightOf" "Monitor0"
    Option     "Primary" "False"
EndSection

Section "Device"
    Identifier "Device0"
    Driver     "nouveau"
    Option     "monitor-DVI-D-1" "Monitor0"
    Option     "monitor-VGA-1"   "Monitor1"
EndSection

Section "Screen"
    Identifier   "screen0"
    Device       "Device0"
    Monitor      "Monitor0"
    DefaultDepth 24

    SubSection "Display"
        Depth   24
        Modes   "1920x1080"
    EndSubSection
EndSection
```

解像度はMonitor0、Monitor1共に1920x1080としています。

`monitor-DVI-D-1`, `monitor-VGA-1` のあたりがポイントで、出力したいデバイスにMonitorセクションの設定を割り当てます。

デバイスの定義名は `xrandr -q` の出力を参考に。

```text
$ xrandr -q | grep "connected"
DVI-D-1 connected 1920x1080+0+0 (normal left inverted right x axis y axis) 531mm x 299mm
VGA-1 connected 1920x1080+1920+0 (normal left inverted right x axis y axis) 476mm x 268mm
```

以上で設定はおしまい。

参考
----
- [Xorg/Multiple monitors - Gentoo wiki](https://wiki.gentoo.org/wiki/Xorg/Multiple_monitors)
