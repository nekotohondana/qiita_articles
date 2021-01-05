---
title: emacs evil mode wo termux de
tags: Vim Evil Emacs Termux
author: dauuricus
slide: false
---
2020.12.18
ここに記載した情報と直接は関係しませんが、アンドロイドのターミナルエミュレーターで日本語文字を入力するためのいく通りあるかわからない方法を探すのには時間がかかりますから、書いときます。
もしあなたが Termux で日本語入力をしたい場合 IME アプリ [google 日本語入力](https://www.google.co.jp/ime/ "google 日本語入力")または、オープンソースの mozc for Android を試してください。

![Screenshot_20201227-223311.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/bee7c692-9adf-0512-0150-169239e2dc0c.png)


Termux vim (デフォルトの状態)and google IME 
https://youtu.be/mZLDSxeXo1I

![Screenshot_20201228-172213.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/e18b2144-d26a-4348-7160-d880ba7c25b5.png)

Termux emacs Evil and mozc IME       
https://youtu.be/JEnO2Mnh9bo

---

# Emacs using Evil Mode on Android

## 1.install termux
then

```
apt update
apt upgrade
```

## 2.install emacs on termux (no need to root privilege, it means without sudo)
(eg: apt install emacs)

If you fail to fetch during installation
eg :
my case ...

```shell_session
Error reading from server - read (5: I/O error) [IP: 54.187.47.108 443] E: Unable to fetch some archives, maybe run apt-get update or try with --fix-missing?
```

I think you should switch termux repogitory default host to
mirror host.
![Screenshot_20201220-174524_kindlephoto-71325254.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/bbd0cc96-c6c6-ccf4-07d9-122037e4d2d7.png)


Here is a hint
[how to change repogitory](https://youtu.be/nCaYiozDRNE)

https://github.com/termux/termux-packages/wiki/Mirrors#mirrors-by-the-tsinghua-university-tuna-association

## 3.install git (eg: pkg install git)
![IMG_20201218_194406_998.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/40228de9-7d00-7ae9-5490-ee0264f3a66c.jpeg)

## 4. git clone

```shell_session
git clone https://github.com/emacs-evil/evil ~/.emacs.d/evil
```
![Screenshot_20201218-191406.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/3fc71d8b-bdd9-98e6-179b-7188d2aa1fc5.png)

https://youtu.be/nI7fWExzr9s

## 5. Edit ini.el file by emacs 
eg:
~$ emacs ~/.emacs.d/init.el
![IMG_20201218_194406_995.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/54bb5713-6650-d715-02a6-f8547be87df8.jpeg)


then,
copy and paste in emacs frame

```
(add-to-list 'load-path "~/.emacs.d/evil")
(require 'evil)
(evil-mode 1)
```

as you can see
![IMG_20201218_194407_000.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/9cd35ff3-dae4-95ce-222d-ffd8c0654faf.jpeg)
## 6.type command (C = control key, M = Alt key)
type
`C-x` `S`
(emacs : press down `control key + x` then `s` , this combo is save command)
*save*
`.emacs.d/init.el` file
![Screenshot_20201218-191758.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/b48f4d38-e84e-92af-51e2-74e64af49af6.png)


type 
`M-x` eval-buffer
(emacs : press down M = `Alt key` + `x` key , then type `eval-buffer` )

see [emacs cheat sheet on duckduckgo](https://duckduckgo.com/?q=emacs+cheatsheet&ia=cheatsheet&iax=cheatsheet)

![IMG_20201218_194407_011.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/50cdc064-cf1a-b558-3202-1bb7e81b8da9.jpeg)
## 7. Now `<N>` is vim mode = Evil mode
Emacs : `<E>`
Vim : `<N>`
Vim command ... you can search [`vim cheat sheet` on duckduckgo](
https://duckduckgo.com/?q=vim+cheatsheet&ia=cheatsheet&iax=1)
Use `CTRL-z` changing mode

![IMG_20201218_194407_016.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/482ca3b8-9e51-8e0b-3a91-c6a601018484.jpeg)

see [youtube vid](https://youtu.be/nbC5HC_NJy0)

Cf.
How to Navigate Emacs using Evil Mode
Updated Wednesday, October 7, 2020 , by Edward Angert
Link 
[https://www.linode.com/docs/guides/emacs-evil-mode/](https://www.linode.com/docs/guides/emacs-evil-mode/)

Good guide about Evil
evil-guide by noctuid
Emacs/Evil for Vim Users
https://github.com/noctuid/evil-guide
