# FMVU14005にDebian12を入れてみた

訳あって[FMVU14005](https://jp.fujitsu.com/platform/pc/product/lifebook/1801/u938s/spec.html)の中古品を得たので、DebianをHeadlessでインストールした。

インストール・セットアップ手順はすでに数多くの先行者の記事が存在するので、本稿では作業において非自明だった点に絞って記述する。

## NongraphicalなInstallでキー入力が効かない

[インストーラ選択画面](https://www.debian.org/releases/testing/amd64/ch05s01.ja.html#boot-screen)において、`Grahpical install`を選択した場合にはその後立ち上がるインストーラは問題なくキー入力を受け付けるが、(Nongraphicalな)`Install`を選択するとキー入力を受け付けない。

NongraphicalなインストーラはTUIで動いているので、キー入力を受け付けないことにはインストールを進行不能となる。

これは備え付けのPS/2キーボードか、LinuxのPS/2デバイスコントロールモジュールであるi8042かのどちらかのバグによる。(インストール後には再発しないのでインストーラの問題か?)古伝[^nomux]にある通りカーネルパラメータに

```sh
i8042.nomux=1 i8042.reset
```

を入れてインストーラを起動すると解消した。このパラメータはインストール後には`/etc/default/grub`に入っており、消したければ消せる。

[^nomux]: 例えば、[reddit](https://www.reddit.com/r/linux4noobs/comments/1ahmvvo/debian_12_installerlive_usb_does_not_recognize/) や [stack exchange](https://unix.stackexchange.com/questions/28736/what-does-the-i8042-nomux-1-kernel-option-do-during-booting-of-ubuntu)に情報が散在している。

## 一般ユーザ名にadminを使えない

[一般ユーザの作成](https://www.debian.org/releases/testing/amd64/ch06s03.ja.html#make-normal-user)でユーザ名に`admin`を指定すると、予約されているユーザ名とエラーを出される。

これは[著名なパッケージによって作成される(され得る)システムユーザ名だと判定されている](https://lists.debian.org/debian-boot/2007/11/msg00303.html)ためなので、諦めて別のユーザ名を使用するのが良い。どうしても変えたい場合は以下の手順(また試していないので自己責任で)で変更できるが、**非推奨**とする。

1. 物理コンソールからrootでログインする(ログイン中のユーザは変更できない。)
1. 適当な記事を参考に変更する。例えば[Linuxでユーザーアカウント名、グループ名の変更](https://qiita.com/tukiyo3/items/eb160174fa0bad551a45)を見ながら

    ```sh
    usermod -l admin <username>
    groupmod -n admin <username>
    usermod -d /home/admin -m admin
    ```

    等すれば良い。

## パーティションを切るとventoy USBもいじられる

[パーティションの分割](https://www.debian.org/releases/testing/amd64/ch06s03.ja.html#di-partition)でメインSSDについて設定すると、ventoyの入ったUSBについても弄られてしまう。

原因は謎だが、USBの状態をEFIとして使用、ブートフラグオンとすると元の状態に可能な限り近づくようである。(情報求む)

## 追加ソフトウェアの中身がわからない

[追加ソフトウェアのインストール](https://www.debian.org/releases/testing/amd64/ch06s03.ja.html#di-install-software)で列挙された項目が実際に何をインストールするか不明瞭である。

上の方は全てGUIの選択肢なので、Headlessで使う分には考えなくて良い。そうでない項目については以下の通り:

- webサーバ: apache
- sshサーバ: openssh
- standard system utilities: 謎 (adduser等の便利コマンドが入っていそうだが、pythonなどの明らかに余計なものも入っているように見える)

## 画面上のコンソールで日本語が化ける

実機の物理コンソール上で日本語が正しく表示されず、`�`として表示される。

`ssh`経由のターミナルエミュレータでの世界に慣れきっていたために知らなかったが、これは仕様である。裸のttyはマルチバイト文字を表示できるように作られていない。

広く知られた解決手段としては`fbterm`がある。

## `<hostname>.local`でアクセスできない

`ping <hostname>.local`が届かなくて困惑するが、これは単に`avahi-daemon`がデフォルトで入っていないためである。`apt install avahi-daemon`とすればよく、特に設定はいらない。

## sudoが使えない

`sudo`が使えないのもデフォルトで入っていないためだが、インストールしても使えないのは

- ユーザが`sudo`グループに入っていない
- ユーザが設定ファイル`sudoers`に記述されていない

ためである。前者を解消すれば`sudo`グループのユーザとして`sudo`が実行できるようになるが、パスワードが毎回要求されて鬱陶しいので、`sudoers`に`NOPASSWD`で登録すると(危険ではあるが)便利になる。[sudo のパスワードを入力なしで使うには](https://qiita.com/RyodoTanaka/items/e9b15d579d17651650b7)等適当な記事を参照してほしいが、書き込むファイルについては`/etc/sudoers.d/`以下に別ファイルを立てる方が記述が混ざらなくて良いだろう。

## 画面を閉じるとsshが死ぬ

実機の画面を閉じると`ssh`セッションが切断されてしまう。これは正確には**全プロセス**が停止していて、要するにsuspendしているのである。

[ノートパソコンのカバーを閉じても、Suspendしない設定](https://qiita.com/kotai2003/items/5f4f52ffe5d8f41d9934)等にもある通り、Debian 12で電源管理を担っている`logind`のデフォルトの挙動を`/etc/systemd/logind.conf`で見ると

```conf
[Login]
...
HandleLidSwitch=suspend
```

と、画面を閉じるとsuspendするようになっている。

なので`logind`の設定ファイルを書き換えてやれば解決する。この時`/etc/systemd/logind.conf`に書かれているように、`/etc/systemd/logind.conf.d/`の下に適当なファイルを作って

```conf
[Login]
HandleLidSwitch=ignore
```

と上書きするのが良い。

## マイクが音を拾わない

`pulseaudio`(+ `ssh` の X11Forwarding)を使ってスピーカやマイクを使おうとすると、内蔵マイクが反応しないことに気づく。

[カタログのカスタムメイド項目(55ページ)](https://jp.fujitsu.com/platform/pc/product/catalog/ctlg_lifebook_201804-1.pdf)を見ると、FMVU14005ではマイクはオプションなので、単にオプションが付けられていないタイプだったのだろう。

## イヤホンジャックのマイクが出現しない

外部マイクを使ってみようとイヤホンジャックにマイク付きのヘッドフォンを挿してみると、外部スピーカは稼働するが外部マイクは稼働しない。

これはサウンドカードを担当するsnd-hda-intelモジュールが一般的なモードで動作しているためである。まず`pactl list`などでサウンドデバイスを確認すると`ALC255`と出る。そこで[HD-Audio Codec-Specific Models](https://docs.kernel.org/sound/hd-audio/models.html#alc22x-23x-25x-269-27x-28x-29x-and-vendor-specific-alc3xxx-models)を見ると`lifebook-extmic`といかにもそれらしいオプションがある。

`/etc/modprobe.d/`以下に適当にファイルを作り、`options snd-hda-intel model=lifebook-extmic`と書いて再起動・モジュール再読み込みしてやれば外部マイクを拾うようになる。[^input]

[^input]: 実際、`dmesg`をみてみると読み込むピンが増えている。
