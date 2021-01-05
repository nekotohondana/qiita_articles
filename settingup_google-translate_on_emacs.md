---
title: emacsでgoogle-translateしたい。（Termux）
tags: Termux Emacs Evil
author: dauuricus
slide: false
---
emacs で google 翻訳したいと思いました。

bash などの terminal の CUI プログラムから google API に文字列を投げ込んで、処理されて戻ってきたものを表示するというイメージです。
多分できるんだろう、と emacs 見たこともなかったところからスタートでした。

できました。
ひと月かかったような気がします。

（イチから作ったわけではなく、すでにあるものの使い方を理解するのにゼロから始めて、フルタイムでという意味です。特に理解力がないとか、特別にあるとかは、無いです。ゼロ知識のレベルに合わせた情報がほとんど見つからないので、読む（もしくは読まない）ことに、そのくらいかかると思います。）

atykhonov/google-translate: Emacs interface to Google translate
https://github.com/atykhonov/google-translate

こちらを使わせていただきます。
atykhonov/google-translate を　MELPA という emacs のプログラムパッケージのリポジトリからダウンロードして、emacs へセットアップするということになります。

それともう一つ popup も使わせていただきます。
popup.el　Copyright (C) 2011-2015 Tomohiro Matsuyama
https://github.com/auto-complete/popup-el

atykhonov　google-translate　では、単体で翻訳結果を別のバッファを開いて表示する機能のすべてを持ってないようなので、ポップアップ（新たにバッファなど開いて表示する）プログラムを呼び出すのに、必要になるのだと思います。atykhonov　google-translate　をインストールするときに、 popup が無いというエラーになりますから、　popup.el　を後から MELPA 経由でインストールすると問題がないと思います。
atykhonov　google-translate　インストール時にエラーがないという場合には、必要ありません。

ありがとうございます。

---

順を追って、説明したいと思いますが、まず最終的なところから。

これを見てください。何もわからなくて当然ですから、わからなくていいです。見て、見たと思えば一歩前進です。

##### 設定ファイル init.el
https://gist.github.com/dauuricus/73cec5c249eca5a9a1079a3da490d418#file-init-el

これは、 emacs の設定の記載です。後ほど説明しますが、先に眺めてもらって、こういうのがいるんだな、とか今現在これで通るんだ、とか、これじゃ駄目だとか、おもいおもいに感想をもってください。
最低限この設定で、 google-translate は期待したとおり動作しました。

Termux の package の emacs で Evil mode で google 翻訳を使っている場面（ youtube 動画）
https://youtu.be/ERPeCYSM8zI

この動画のように emacs 内で google 翻訳機能を使うことを想定して、その設定について説明していきます。

`Evil mode` についてはここではふれませんが、設定ファイルのおしりのところに記載がありますから、 `Evil mode` の設定までのことを別の記事で簡潔にまとめてありますので興味があるようでしたら、そちらをどうぞ。
https://qiita.com/dauuricus/items/fcd30edb55127c5a5986
必要ない方は、設定ファイル `init.el` のおしりのところは削除して無視していただけば問題ないないです。

これからじょじょにまとめていきます。
 CUI emacs に `google-translate`　をインストールしている作業を動画でキャプチャーしました。スムーズな作業とは云えませんが、一連でひととおりインストールから機能するまでの流れになっています。
https://youtu.be/lONj2d9HdEQ?t=131

つづく・・・
