---
title: Android OS に openSSH で Android OS を接続する。 
tags: SSH Termux OpenSSH
author: dauuricus
slide: false
---
GNU screen とはどういうものか、やってみたかったので、Android OS (Amazon Fire 7) に Android OS (Amazon Fire 8) を接続してみることにした。

よくわかっていなかったので、GNU screen とは、SSH を内包していて、片方のリモートされる側に SSH デーモンを、もう片方のリモートする側に GNU Screen を用意するものと考えていたが、やってみると違い[^1]、 SSH の設定自体が成功していないために、フォーカスを SSH 接続にして、SSH での接続をトライアンドエラーでやっていると、かなり愕然とするほど時間が経っていて、これは大変、大変だわと思いました。
なので、間違いのないように記録しておく。

ここで、 SSH てどうなってこうなってる？を、パブリックキー（交換鍵）の使われ方をダイヤグラム、フローチャートみたいなもので見渡せたら想像できるから最初に描こうと思い、参考になるのを探そうとして、まるで見つからない、もしくは見つけることができないということがずいぶん探し回ってから、解りそれを保留。（ RSA のすごく古い本には載っていた気がする。一日、二日で本当のところが判別つかない。）

* SSH てどうなってこうなってる？
cf.
@angel_p_57
2018年09月17日
SSHの公開鍵認証における良くある誤解の話
https://qiita.com/angel_p_57/items/2e3f3f8661de32a0d432#fn5

* SSH てどうなってこうなってる？
cf.
SSHの進化
04kc030　賈 音
指導教員　坂本 直志　准教授
http://www.net.c.dendai.ac.jp/~daniel/#A4_1_3

* SSH てどうやってセットアップしてんの？
cf.
SSH Crash Course | With Some DevOps
a local Ubuntu server on my network and then create a remote Digital Ocean Droplet to work with
https://www.youtube.com/watch?v=hQWRp-FdTpc

[^1]: GNU Screen [youtube 動画](https://www.youtube.com/watch?v=C2mtQw3a4QA&feature=youtu.be)

---

ここでは、 Android A が Fire7 で Android B が Fire8 としておく。

Termux が A また B 両方にインストールされている。

SSH が正常につながらないのは、公開鍵をつかって認証し、パスワードなしで接続するべきところで、パスワードを要求されて認証失敗になり、アクセス拒否されるというような動作になったということで、このような状態であった。

![IMG_20210101_225548_085.jpg](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/225786/5fd90716-3985-4837-dfd4-e4b30df62bb0.jpeg)

ここで整理しておくと、

- A （ fire 7 ）には openssh がインストールされていて、 sshd でデーモンが立ち上がっていて、ポートの 8022 番で待ち受けている。

- B （ fire 8 ）には openssh がインストールされていて、 A と B がともに同じ LAN 内にいる。

- B から アドレスとポート番号を指定して A へ接続を試みるためには、 A の IP アドレスがわかっている必要がある。

- SSH でパスワードなしで接続するためには、 A または B でパブリックキー（公開鍵）、プライベートキー（秘密鍵っていうのかな）、フィンガープリントを生成して、パブリックキーの方を、接続する相手に登録しておく必要がある。

ここが失敗している。なので、この部分にフォーカスしていくことになる。

SSH 接続自体よりも、この概念をよく見ていくということである。

パブリックキー（公開鍵）とプライベートキー（秘密鍵または Id と呼ばれる）がどのようにしてどのようにして照合するのかホントのところまではわからない。なので、そこのところは最も重要だけれども、ここでの範疇から除外する。というスカスカな内容で、セットアップに必要なことだけ記録する。

openSSH にはこのようなものが含まれている。

* scp, Secure copy protocol 
* sftp, SSH File Transfer Protocol
* **ssh**, Secure Shell
* ssh-add and ssh-agent, 
* **ssh-keygen**, key generate
* **sshd**, the SSH server daemon


## 鍵の生成（ key generate ）と、鍵の登録 （ Authorized ）

リモートされる側で、リモートする側の相手側のパブリックキー（公開鍵）をあらかじめ登録する場合と、その逆がある。
その逆とは、リモートされる側で生成したパブリックキー（公開鍵）をリモートする側で登録することで、この方向はどちらかで、一貫した作業の流れがあるので、本件は、`リモートする側でキーを生成（パブリックキー（公開鍵）、プライベートキー、フィンガープリント）し、リモートされる側、つまり sshd が動いてる側で、リモートする側のパブリックキーを登録する`という流れを説明する。

言い換えると、`リモートする側 Android B （ fire 8 ）で キーを生成し、そのうちのパブリックキーだけをコピーして、リモートされる側である Android A （ fire 7 ）の .ssh/authorized_key　へ書き込むことで、「キーを登録する == Authorized 」。`

以上のことが整理されていれば、あとはそのやり方の一つの例という感じで、理解しやすいのではないかと思う。


cf.
Connectiong by SSH from Android Termux to Desktop and vice-versa.md
Copyright (c) 2019 Evandro Coan
https://gist.github.com/evandrocoan/f503188587587d7b1d1ba8746c9c6107

cf.
Termux wiki >> 'Remote Access' >> 'Using the SSH server'
https://wiki.termux.com/wiki/Remote_Access


では、どうやって・・・・
## お献立
Termux の openssh を使って SSH 接続で認証に使うパブリックキー（公開鍵）をつくる。

Android A のターミナル（ Termux ）に　Android B のターミナル（ Termux ）から接続するのに、ユーザー名とパスワードで SSH 接続するのではなく、 openssh で鍵を作ってパブリックキー（公開鍵）の受け渡しで、パスワードは無しで SSH 接続するところまで。　


## レシピ
* wi-fi とインターネット接続環境が必要。
* ここでは２つのアンドロイドタブレット　Android A （ fire 7 ）と　Android B （ fire 8 ）だが、手ごろなものがあればなんでもいい。つまりは　Termux　に　Termux　を ssh をつかって接続する。そういった解説がなかったのでやってみたということなので、要件は Termux が使える Android OS のインターネット接続可能なもの 2 こ。
* A と B が LAN へ接続されていること。
* A 及び B が　Firetoolbox　で程よく自由になっていること（　kindle FireOS　の場合）。
* termux ・・・ターミナルエミュレーター　アプリ。
* openssh　・・・ Termux のパッケージに登録されているプログラム。 Termux 内でパッケージをインストールする。今回の主役。同じく SSH 用のパッケージ dropbear とは別のもの。 dropbear は openssh に比べるとドキュメントが少ないので、まずは openssh を使う。ファイルサイズは dropbear のほうが遥かに小さい。またオーストラリアの可愛い動物のことを思い出せる。
* nmap ・・・ Termux のパッケージに登録されているプログラム。 Termux 内でパッケージをインストールする。（なくてもいい）

#### インストール

termux は F-Droid でアプリとしてインストール。

>Important: Do not mix installations of Termux and Addons between Google Play and F-Droid. They are presented at these portals for your convenience. There are compatibility issues when mixing installations from these Internet portals. This is because each download website uses a specific key for keysigning Termux and Addons.

>https://wiki.termux.com/wiki/Termux:API

（ Termux-API というTermux の機能拡張アプリがありますが、 google play store でインストールした Termux は同じ google play store でインストールした Termux-API でないと不具合があると注意書があります。F-Droid の Termux APK の場合は、F-Dorid で揃えて Termux-API APK を使ってください。なお、ここでは Termux-API は使用しません。）  　 

ターミナルエミュレーター（ termux ）で

```terminal
pkg update && update
```
としてリポジトリ情報を更新してから、各プログラムのパッケージをインストールする。

```terminal
pkg install openssh
```

※インストールすると自動的にいくつかのキーが生成されます。

```terminal
pkg install nmap
```

#### アンインストール

```terminal
apt autoremove パッケージの名前
```

でインストールしたプログラムを削除（アンインストール）することができる。


例えば nmap をアンインストールしたい場合は、このようになります。

```terminal
apt autoremove nmap
```

## お道具

ここで使うパーツとして、ターミナルで使うかもしれないコマンドを紹介しておきます。
全部使うわけではないですが、これらのコマンドを知ってれば、やりたいことをこれらコマンドの組み合わせて使えばできます。
後に一例として 6 ステップの手順を示しますが、材料が揃っていれば応用が利くので、先に此処に並べておきます。賢い人は、材料だけで料理できると思います。
組み合わせのパズルなので、手順通りやればできると思いますが、手順通りやる必要もないので、色々やって、途中でやめるのも賢いと思います。
あれ❓変だなということに気がついたら調べてみてください。
賢い必然性はありません。

##### bash コマンド
* `ls -a`
* `cd`
* `mv`
* `mkdir`
* `rm -i`
* `cp`
* `cat`
* `chmod`
* `exit`

* 現在の　user 名をたずねるコマンド。私は誰だっけ？

```bash
whoami
```

* wi-fi LAN でデバイスのローカルアドレスってどうなっているか等、教えてくれるコマンド。

```bash
ifconfig wlan0
```

デバイスに割りあてられている　IP アドレスを確認するコマンド

 
```bash
ip addr list wlan0
```

* TCP で動いているものを見るコマンド

```bash
netstat -tln
```

* ssh のデーモンのプロセスを止めるコマンド。リモートされる側で始めた sshd を止めるのはこちら。

```bash
pkill sshd
```

* TCP のポート番号 8022 番を使っているプロセスを kill するコマンド。 `sudo` 無しでプロセス番号不明でも使える。
Termux の sshd はデフォルトで 8022 ポートを使う。つまり、デフォルトでポート番号8022を使っている場合に限って、 sshd を止めれる。

```bash
fuser -k -n tcp 8022
```

* ssh 接続後の ssh 接続状態で ssh を離脱するコマンド。リモートする側でリモートされる側の sshd からログアウト。

```bash
~.
```

##### Termux コマンド
Termux のアクセスできるストレージを拡張します。
デフォルトだと、 Android OS の仕様でアプリに割り当てられた以外の領域に書き込みができないため、｀~/strage/downloads/` に Termux のターミナルからファイルコピーなどの操作ができません。つまりパーミッションの変更ですね。アプリだとよくありますよね、写真をアップロードするアプリだとかで、「ストレージに色々アクセスしますけどいいですか？」みたいな質問がよくあると思います。
あれの設定と同じようなもので、それをターミナルでコマンドでやります。

```terminal
termux-setup-storage 
```
これは、以降 ssh で鍵の生成をするので、その鍵のコピーをタブレットの「ダウンロード」フォルダにコピーしたり、そこからファイルを読み取ったりしたい場面で必要になってきます。

cf.
[Termux wiki >> 'Termux-setup-storage'](https://wiki.termux.com/wiki/Termux-setup-storage)

##### openssh コマンド

* 鍵を作るコマンド

```
ssh-keygen
```

実行するとパブリックキー（公開鍵）`~/.ssh/id_dsa.pub`　ができます。（他にもうひとつプライベートキー（秘密鍵）と、フィンガープリントが生成されます。）

>```
ssh-keygen -t 作成する鍵の暗号化形式を指定
              rsa（デフォルト）
              dsa
              ecdsa
              ed25519
           -b ビット数 作成する鍵のビット数を指定（ RSA 形式の場合、デフォルトは 2048bit ）
           -f 生成する keyfile を指定
```

例:

```
ssh-keygen -t rsa -b 2048 -f id_rsa
```

cf.
Termix wiki >> Remote_Access >> Using the SSH server
https://wiki.termux.com/wiki/Remote_Access

* リモートされる側で SSH 接続を待ち受けるためにデーモンを起動するコマンド

```
sshd
```

これに対して sshd を止めるコマンドは `pkill sshd` です。

---

## 調理

### 1. Termux をインストールします。
操作対象:
`Androido A` / `Androido B`

### 2. openssh をインストールします。
操作対象:`Android A` / `Android B`

```terminal
pkg install openssh
```

### 3. 鍵を生成します。
操作対象:`Android B`
接続してリモートする側のターミナルで各種鍵を生成します。鍵は公開鍵と秘密鍵の２種類で、両方ともに必要です。
生成のコマンド:

```
ssh-keygen
```

実行すると以下のようになります。

```
~ $ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/data/data/com.termux/files/home/.ssh/id_rsa):

Enter passphrase (empty for no passphrase):
Enter same passphrase again:

Your identification has been saved in /data/data/com.termux/files/home/.ssh/id_rsa

Your public key has been saved in /data/data/com.termux/files/home/.ssh/id_rsa.pub

The key fingerprint is:..................................

```

２つのキーができたようでんな。
`id_rsa`
`id_rsa.pub`

SSH 認証の全体のワークフローが漠然としていると、まぎらわしいですが、 `id_rsa` は `identification` と呼称されていますが、プライベートキー（秘密鍵）ですね。

パブリックキー（公開鍵）は、`~/.ssh/id_rsa.pub` です。 pub て書いていますしね。生成された場所は `/com.termux/files/`ホームディレクトリ下です。つまり Termux アプリの割り当てられた領域にあります。
`~`はホームディレクトリのことを示しています。

現在どのディレクトリにいても

```bash
cd ~
```

でホームディレクトリに移動できます。 `~` （にょろにょろ）はチルダと呼びます。 `cd` は　`チェンジ　ディレクトリ` でしょうね。

```bash
cd ~
```

としてホームディレクトリに移動して、
以下のコマンドを使って、生成された公開鍵がファイルとして存在しているかを確認してみます。

```bash
~ $ ls -a .ssh
.  ..  authorized_keys  id_rsa  id_rsa.pub  known_hosts
```

在りましたか？ `.ssh` はフォルダですが、先頭にドットが付いた名前のフォルダは隠しフォルダになっていますからコマンド `ls` には `-a` オプションをつけて実行します。
`リスト　-オール`　ということですね。
確認できたら次のステップです。

公開鍵`id_rsa.pub`を、リモートされる側である Android A にある`autorized_keys`に登録するのです。

### 4. 公開鍵のコピーをとります。
操作対象:`Android B`
接続してリモートする側のターミナルでの作業です。

`~/.ssh/id_rsa.pub`をコピーしたいのですが、例えばメールでファイル添付するとして、メールで扱いやすい領域にコピーして移します。コピーなので`~/.ssh/id_rsa.pub`には残ったまま存在することは意識しておいてください。
ここでは、たとえばとして、`ダウンロード`フォルダにコピーします。
まず Termux が`ダウンロード`フォルダに対してファイルを保存できるようにします。

```terminal
termux-setup-storage
```

`ダウンロード`に公開鍵をコピーします。

```bash
cp ~/.ssh/id_rsa.pub ~/storage/downloads/
```

`ダウンロード`にコピーできたか確認できますか？

```bash
ls ~/storage/downloads/
```

できていれば、メールアプリで `id_rsa.pub` を添付してみましょう。
方法は必ずしもメールでなくてもいいです。このファイルを Android A にコピーできる方法をとってください。

例えば gmail やウェブサービスのメールの場合だと、メールサービスにログインしてメールを作成して、 `id_rsa.pub` を添付ファイルにします。
そして下書きのまま保存して、 Android A に移り、同じメールサービスに同じアカウントでログインし、保存されている下書きのメールから、 `id_rsa.pub` をダウンロードすると、 Android A の`ダウンロード`フォルダにファイルがコピーされます。
Android A のターミナルで、`ダウンロード`フォルダは `~/storage/downloads/` です。


### 5. 公開鍵を登録します。「キーを登録する == Authorized 」
操作対象:`Android A`
リモートされる側のターミナルでの作業に切り替わります。
まず Termux が`ダウンロード`フォルダに対してファイルを読み出しできるようにします。

```terminal
termux-setup-storage
```

ステップ 4. の最後でメールなどで Android A でファイルをダウンロード可能の状態にしました。ダウンロードしてください。

Android B の `id_rsa.pub` のコピーが Android A にあるか確認します。
 
```bash
ls ~/storage/downloads/
```

 `ダウンロード`フォルダに `id_rsa.pub` が見あたらなければ、4. に戻って確認してからここに戻ってきてください。
　
Android A の `~/.ssh/authorized_keys` に Android B の公開鍵である `id_rsa.pub` を書き加えます。「キーを登録する == Authorized 」

```bash
cat ~/storage/downloads/id_rsa.pub >> ~/.ssh/authorized_keys
```

ここで使った `>>` の場合は書き加えになります。  `>` の場合は上書きになります。

ファイルとディレクトリのパーミッションを変更します。


```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh 
```

ここまでで、Android B の公開鍵が Andoroid A に登録されたはずです。

確認したければ `authorized_keys` の中身を直接ファイルを開いて見るとわかるはずです。

たとえば、 vim エディターでファイルを開いて中身を見るとすると:


```bash
vim ~/.ssh/authorized_keys
```

一行で、こんな感じの長い文字列が入っていたら、多分大丈夫です。

```
ssh-dss AAAAB3NzaC1kc3MAAACBAMKP/UZzYAazzV2jNkOc/pDtsnwovbF/W3hRbYnmfJBmTG+UXybEX2seE9X0LqR7mblMzVYe5CBs24i5boMFvVdZ93WxgX6La4Wz+M1bk3MX1rQYyq3sgYTtvs+g+ey0U0S7ILfn3uXBQlshK7Wyj4XXnewpTQ+BviXUkXi7iedLAAAAFQC5AxBPR43SzcM/nhsKWYOe+V9XOwAAAIEApF12ZeRZnk7zrN30lzbD0vLL7Ql6CDGS2JcHHu0Sn/wW0EHO0ojO4uAN6IZWPOeY7LpMyfl/kH8xPZ/Z0gfhSYhWq5iXLAAx+drVzli+KSDGjNKXJskuCx7mPufgh9wWYsELmCXCmZ2ZHDCxgF1k8gJEtJ7pVfDqhvMdmsyg7w4AAACAQaJyIfH+Thz0KWxReU1h97KcGUc7zNytFRJe7Rq5y4FyvL+oUygeIj6hkK9LFeel1FvuGZrf0/mHvpkPV2TmuM9VkO62XYo05DGk9kzcyaWLXq8ysuYfpFql7IfilKI2qwk+3rjo3Xs4rt2Y7ImrGCg/9smwWhXf+LFfdy61SrI= u0_a256@localhost
```

以下をタイプすれば、保存せずに vim エディターをぬけれます。

`:q`

次に、確認ですが、 nmap を使います。

nmap のインストール方法は:

```terminal
pkg install nmap
```

Localhost の様子を覗いてみます。とくに何もないのを確認して、使用前と使用後の変化を見るためですので、スキップしてもかまいません。

```
nmap localhost
```

はい、見ました。

次に、 sshd を開始します。これが「Termux で ssh サーバーをたちあげる。」ということになると思います。 SSH で接続可能なように待ち受ける状態にするということで、くどいかもしれないですが、今あなたがいるのは `Android A` のリモートされる側として設定しようとしているタブレットのターミナル（ Termux ）であることを確認してください。 

 sshd を開始します。

```bash
sshd
```

それでは、使用後をチェックして、使用前との差を見てください。

```
nmap localhost
```


```terminal
$ sshd
$ nmap localhost
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-02 23:41 JST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0084s latency).
Not shown: 999 closed ports
PORT     STATE SERVICE
8022/tcp open  oa-system

Nmap done: 1 IP address (1 host up) scanned in 1.32 seconds
$
```

 `sshd` 使用前

```terminal
$ nmap localhost
Starting Nmap 7.91 ( https://nmap.org ) at 2021-01-02 23:40 JST
Nmap scan report for localhost (127.0.0.1)
Host is up (0.0080s latency).
All 1000 scanned ports on localhost (127.0.0.1) are closed

Nmap done: 1 IP address (1 host up) scanned in 1.50 seconds
$
```

TCP 8022 番のポートが使われていることがわかりました。

次は Android A の IP アドレスを調べましょう。

 wi-fi につながるように設定してからコマンドをタイプして実行してください:

```terminal
ip addr list wlan0
```

コマンドを実行したらこんな感じのが出てきましたか？

```
inet 192.168.0.3/24 brd 192.168.0.255 scope global wlan0
```

このリゾルトの場合　`192.168.0.3` の部分が Android A の LAN 内でのローカル IP アドレスです。　`192.168.0.3` の部分はあなたの場合では違うと思いますが、4 つの区切られた数字であれば、たぶんそれが IP アドレスです。TCP 4　の場合は。

次にユーザーアカウント名をチェックしてみましょう。
ターミナルの中でコマンドをタイプしてエンターを押します。

```
whoami
```

何かそっけなく短い答えが返ってきましたね。

私の場合は、`u0_a57`でした。これも、あなたの場合は違っていますが、そう違わないと思います。
Android A での設定は完了です。


### 6. SSH 接続
操作対象:`Android B`
リモートする側の Android B のターミナルでの作業に切り替えます。

まず、ユーザー名を確かめてみましょう。

```
whoami
```

`u0_a228`でした。（これも、あなたのユーザー名とは違っています。）

Android A の IP アドレスはステップ 5. のおしまいのところで調べがついています。
ポート番号は、8022 番でした。
Android B から ssh コマンドで Android A に接続します。
`-p`スイッチで接続するポート番号を指定します。ユーザー名は無しでいいと思います。

```terminal
$ ssh 192.168.0.3 -p 8022 
```

しばらくかかって接続できましたか？

なんかあまり変わらないけど、画面が書き換わったような気がして、 Termux ... っていう見たことあるようなのが出てきていて、画面がスッキリしてるような気がしたら、おそらく SSH 接続成功です。

`whoami`とコマンド入力してエンターキーを押してみてください。

SSH 接続からログアウトしたい場合は、
`~.`
とタイプしてください。チルダとピリオドです。

sshd を止めたい場合は Android A のターミナルで
`pkill sshd`
とコマンドをだせば、ストップします。 Android B でログインしていた場合は SSH 接続が切れます。

sshd が止まっているか確認したい場合は、
`netstat -tln`
とコマンドを出せば確認できます。
