# Readme センターHP管理マニュアル

2026.04.09

## 概要

ドメイン名 sh-center.org で運用しているセンターのHPは、これまでテレコンサービス（松澤さん）がWordPressを利用してサイトを制作し、Drive Networkでホスティングしていた。2025.12.01にDrive Networkの事業廃止に伴い、サーバーはDrive.LNXに移管された。センターの今後の運営を考慮し、静的サイトジェネレーターのHUGOを利用してサイトを制作し、さくらインターネットでホスティングする。
本ドキュメントは、HUGOを利用してコンテンツを制作し、アップロードする手順を記述する。
ドメインをさくらインターネットに移管するのは、2026.06.01の予定である。
このドキュメントは、現時点で sh-center.org ではなく、https://sh-center.sakura.ne.jp として記述する。

ドメインをさくらインターネットに移管後は、/kaithems/hugo.toml を編集し、baseURL = "https://sh-center.sakura.ne.jp"をbaseURL = "https://sh-center.org"に修正すること。

## 準備

以下のアプリをあらかじめダウンロードすること。

- Editor: [Visual Studio Code(VSCode)](https://code.visualstudio.com/)

コンテンツを追加する場合は、markdownファイルを作成する。エディターはなんでもかまわないが、VSCodeを前提に話を進める。

- 静的サイトGenerator: [Hugo](https://gohugo.io/)

markdown fileやその他のコンテンツ(jpg, png, pdf, docx)などから、web serverにupload可能なファイル群(html, css, JavaScript, jpg, png, pdf, docx)を作成する静的サイトGenerator Hugoを利用する。

- クラウドストレージとローカルファイルの同期ツール: [rclone](https://rclone.org/)

Hugoが作成した静的サイトをクラウドにアップロードするツールとして、rcloneを利用する。rcloneを利用することで、変更があったファイルのみを更新することができる。

## コンテンツ制作に関して

ここでは、「お知らせ」の追加と、既存のページの修正に関して記述する。
ホームページの修正や、サイトのディレクトリ構造の変更が必要な作業に関しては、別途記述する。
「お知らせ」やホームページ以外の既存のページは、markdownで記述する。markdownはHTMLを簡単に記述できるようにしたものである。VSCodeでmarkdownを記述すると、VSCode内でプレビューすることができる。
「見出し」「リスト」「テーブル（表）」「図の挿入」の記法を理解すれば、とりあえず使える。

「お知らせ」の追加方法

1. /kaithems/content/blog folder の下に、フォルダを作成する。フォルダ名は YYYY-MM-DD のフォーマットとする。（例：2026-04-25）
2. 作成したフォルダ内に、VSCodeでindex.ja.md という名前のテキストファイルを作成する。
3. ファイルの先頭に、フロントマター(*1)を記述する。
4. フロントマターに続いて、markdown記法でコンテンツを記述する。
5. 図（.png, .jpg）は、作成したフォルダ内に入れる。

(*1) フロントマターの例

```
---
title: "Dr. Sukumal 氏とDr. Kalika 氏と梅嶋氏来訪"
date: 2025-09-22
summary: "タイ国立カセサート大学のDr. Sukumal 氏とタイ国立電子コンピューター技術研究センターのDr. Kalika 氏、がIECシステム委員会コンビーナ梅嶋 氏が神奈川工科大学にご来訪されました。"
---
```

## サイトの確認

コンテンツを制作しているPC/Mac上でWeb serverを動作させて、サイトの確認を行う。

```
> hugo server
...
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

Chromeなどのブラウザーを起動し、localhost:1313 をアクセスするとサイトが表示される

## 公開用のファイルの生成

以下のコマンドを実行すると、/public フォルダー内に静的なサイトが作成される

```
> hugo --gc --minify --cleanDestinationDir
```

オプションの説明:  

```
–gc：クリーンアップのタスクを実行する
–minify：ファイルを小さくする
–cleanDestinationDir：不要なファイルは削除する
```

## コンテンツのアップロード

/public フォルダをWeb serverにアップロードする。ftpで全ファイルをアップロードしてもいいが、２回目以降のアップロードは差分だけアップロードするほうが効率的である。
そのために、クラウドストレージとローカルファイルの同期ツールである rclone を利用する。

設定（以下、さくらインターネットの場合）

```
> cd </kaithemsディレクトリ>
> rclone config
n （new remote）を選択
name に sakura を入力（判別できる名前）
Storage に18 (FTPに対応する番号) を入力
host に sh-center.sakura.ne.jp を入力
user に sh-center  を入力
port はエンター（デフォルト）
FTP password に y を入力
password に ftp server 用のパスワードを入力
もう一度パスワードを入力
tls はエンター（デフォルト）
explicit_tls は true を入力
Edit advanced config? はエンター
Keep this "sakura" remote? はエンター
最後に q で終了
```

同期（アップロード）

以下のコマンドで /kaithems/public 以下のファイルがアップロードされる。

```
> rclone sync -v ./public/ sakura:/home/sh-center/www/
```

npmがインストールされていれば、以下のpackage.json を作るとコマンド入力が容易になる

{
  "name": "hugo",
  "version": "1.0.0",
  "description": "",
  "scripts": {
    "build": "hugo --gc --minify",
    "sync": "rclone sync -v ./public/ sakura:/home/sh-center/www/",
    "serv": "npm run clean && hugo server --gc -D"
  }
}

```
> npm run build でビルド
> npm run sync でアップロード
> npm run serv でローカルサーバーが立ち上がる

## メニューの修正

メニュー修正は、/kaithems/hugo.toml ファイルを編集する

## ホームページの修正

ホームページの修正は、/kaithems/layouts/index.html を編集する

## CONTACT

ContactはGoogle formを利用する。
専用のアカウントを作成
contactkaithems@gmail.com, kait2012, 再設定用：me@hirofujita.com

- 右上（９個のdots:Google apps）からFormsを選択
- 新しいフォームを作成／空白のフォーム
- 無題のフォーム -> お問い合わせ　に修正
- お名前、メールアドレス、会社名（所属）、題名、メッセージ本文
- 回答Tabを選択
- 「回答のその他のオプション」をクリック
  - 「新しい回答についてのメール通知を受け取る」を選択
- 公開をクリック
- 「回答者へのリンクをコピー」をクリック
  - https://docs.google.com/forms/d/e/1FAIpQLSesTgzTWhHJx0m4qL1o2uMUSWlGIV6GnO10sZVuMv8g5ma_hA/viewform?usp=header

contactkaithems@gmail.com のGMailの転送設定(forwarding)がうまくいかない。
kait_shpj@googlegroups.comへは転送されない。
tundora@gmail.com, me@hirofujita.com, hiro.fujita@he.kanagawa-it.ac.jpへは正常に転送される。

