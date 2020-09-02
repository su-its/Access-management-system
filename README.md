# 某室の入退室管理システム

2020年9月2日(水)時点での情報です

**developブランチの現時点での最新版(1bb9f3e)について書いてある。SQLを使う。masterブランチのcsvを使うバージョン(現時点での最新版はea52942)とは全く異なるので注意**

## できること

- 学生証をタッチした時間を学籍番号と名前と一緒に**SQLサーバーに記録する**
- **入室か退室かを判定して**モニタ(というかコンソール)に表示する
- メンバー以外の学生証がタッチされても一応記録に残す(学籍番号は記録できるが、名前は'Unknown'になる)

## できそうなこと

単に記録をcsvに残すだけだと難しかったメンバーの管理機能も既存のSQLの機能である程度できる(はず。コードが書けるとは言ってない)。

- 解錠権持ちと一緒に入室したかどうか確認する
- 在室状況をリアルタイムに監視する(web経由?)

## できていないこと(必須機能/**太字**は必要度高)

- 入室時刻と退室時刻が日をまたぎそうな場合の処理(0時少し前に、締め作業としてその日の入室記録があるのにまだ退室記録が無い人は自動的に退室処理をしてしまう。日付が変わったら再びタッチを受け付けるようにする、とかで解決できそう。日付が変わった直後に思い出して、退出のつもりでタッチしたら無限ループ入りそう。自動で退出処理が行われた場合は注意してればいいか)
- **入室したメンバーへの通知**(室を利用した理由を書くのは手動でやってもらわないとなので)
- 短時間に何回もタッチするなどのいたずら対策
- **SharePointへの自動アップロード**(定時に自動でローカルと同期させるか、SharePoint上のファイルを直接更新するか。必ずしもSharePointに記録がなくてもいいと思う(Microsoft Graphワカラナイ)。必ずそうしろと言うならしかたないけど)

## あったらいいな(必須以外)

- 音声(動作の完了やエラーを報告)
- カードリーダーに透明の箱をかぶせてそこに駅の改札の「ピタっとタッチ」的なマークを貼る
- モニタの代わりにキャラクタ液晶に情報を映す
- タッチパネル <- New!

## 事前準備

Sonyのカードリーダー `PaSoRi SC-360/P`が必要。実験機のラズパイはUSBポートが1つしかないのでハブを噛ませていますが、ラズパイのUSBポートに余裕があれば直接接続する方がいいと思う。Python3を前提にしている。バージョン依存のコードはなるべく入れないように気をつけたが、もしあったらごめんなさい。バージョンが古いとpathlibモジュールが動かないかもしれない。また2のprintが3ではprint()だったりするそうなので、なるべく3.x系を使用するべし。

```
Linux version 4.19.118+
ARMv6-compatible processor rev 7 (v6l)
BCM2835
Raspberry Pi Model A Rev 2
Raspbian GNU/Linux 10 (buster)

Python 3.7.3
nfcpy 1.0.3

mysql  Ver 15.1 Distrib 10.3.23-MariaDB, for debian-linux-gnueabihf (armv7l) using readline 5.2
mysql-connector-python 8.0.21
```

で動作を確認している。
本体であるgate.pyを動かすには`nfcpy`と`mysql-connector-python`モジュールが必要。インストールされていない場合はここでインストール。

```
$ pip3 install -U nfcpy mysql-connector-python
```

テーブルの構造は`src/create_schema.py`に書いてある。itsgateというDBが作成されている状態で、しかるべき設定を行い、これを実行するとテーブルの作成とメンバー情報の登録をやってくれる。


ディレクトリ構成はこんな感じ。設定ファイルの`config.json`は手作業で作る。

```
nfc_dev
|-- README.md // このファイル
`-- src
    |-- auto_close_cursor.py // カーソルのラッパー
    |-- config.json // 設定ファイル
    |-- create_schema.py // データベーススキーマの定義と作成スクリプト。1度作成した後は使わないと思う
    |-- gate.py // 本体。これを実行する
    `-- gate_manager.py // 本体2
```

## 使い方

```
$ cd nfc_dev/mysrc/
$ ./gate.py # 実行権限不足で動作しないなら chmod o+x gate.py
[msg]Good morning!
Please wait.
[msg]Trying to establish connection to itsgate... Success
[msg]Start main routine--- Press Ctrl+C to stop.
(以下略)
```

## 覚え書き

- 止めるときはCTRL+C長押し。
- コンソール以外からテーブルをいじれるようにした方が良いかも?
- (このコード内でデータベースに接続する)ユーザーからはUPDATE権限を剥奪しておくと記録の改ざんが軽減できるかも?

