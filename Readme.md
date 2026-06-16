# Readme センターWeb site管理マニュアル

2026.06.01

## 1. 概要

ドメイン名 sh-center.org で運用している神奈川工科大学HEMS認証支援センターのWeb siteは、静的サイトジェネレーターのHUGOを利用して作成し、さくらインターネットのレンタルサーバーでホスティングしている。本ドキュメントは、Web siteのコンテンツを追加・変更し、アップロードする手順を記述する。

## 2. 準備

### 2.1 アプリのインストール

- テキストエディター: [Visual Studio Code(VSCode)](https://code.visualstudio.com/)
- 静的サイトGenerator: [Hugo](https://gohugo.io/)
- クラウドストレージとローカルファイルの同期ツール: [rclone](https://rclone.org/)

VSCodeは、コンテンツを追加・編集する際に使用する。
Hugoは、Web siteの素材コンテンツから、web serverにuploadするファイル群(html, css, JavaScript, jpg, png, pdf, docx)を作成する。
rcloneは、Hugoが作成したファイル群をさくらインターネットのレンタルサーバーにアップロードする際に使用する。ftpで全てのファイルをアップロードしても構わないが、rcloneを利用すると、変更があったファイルのみを更新する。

### 2.2 Hugo用Web siteコンテンツのダウンロード

以下のURLから、Hugo用Web siteコンテンツをダウンロードする。

```
https://github.com/KAIT-HEMS/kaithems
```

### 2.3 rcloneの設定ファイル作成

rcloneがファイルをアップロードするサーバー情報を設定する。以下、さくらインターネットのレンタルサーバーの場合。

```
> cd </kaithemsディレクトリ>
> rclone config
n （new remote）を選択
name に sakura を入力（判別できる名前）
Storage に18 (FTPに対応する番号) を入力
host に sh-center.sakura.ne.jp を入力
user に sh-center を入力
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

## 3. 「お知らせ」の追加方法の説明

Web siteのコンテンツ修正に関して、まず「お知らせ」の追加方法を説明する。以下の作業を行うことで、メニューの「お知らせ」のページが更新されるだけでなく、ホームページの最新のお知らせも更新される。

### 3.1 コンテンツ作成

1. /kaithems/content/blog の下にフォルダを作成する。フォルダ名は年月日(YYYY-MM-DD)のフォーマットとする（例：2026-04-25）。
2. 作成したフォルダ内に、VSCodeでindex.ja.md という名前のテキストファイルを作成する。
3. ファイルの先頭に、フロントマター(*1)を記述する。
4. フロントマターに続いて、markdown記法でコンテンツを記述する。
5. 図（.png, .jpg）は、作成したフォルダ内に入れる。
6. VSCodeでindex.en.md という名前のテキストファイルを作成する。フロントマターも含め、文章は英語で記述する。

(*1) フロントマターの例

```
---
title: "Dr. Sukumal 氏とDr. Kalika 氏と梅嶋氏来訪"
date: 2025-09-22
summary: "タイ国立カセサート大学のDr. Sukumal 氏とタイ国立電子コンピューター技術研究センターのDr. Kalika 氏、がIECシステム委員会コンビーナ梅嶋 氏が神奈川工科大学にご来訪されました。"
---
```

### 3.2 プレビュー

ターミナルで以下のコマンドを実行すると、Web serverが起動する。

```
> hugo server
...
Web Server is available at http://localhost:1313/ (bind address 127.0.0.1)
```

Chromeなどのブラウザーを起動し、localhost:1313 をアクセスするとサイトが表示されるので、内容を確認する。

### 3.3 公開用ファイルの生成

ターミナルで以下のコマンドを実行すると、/public フォルダー内に公開用のファイルが作成される

```
> hugo --gc --minify --cleanDestinationDir
```

オプションの説明:  

```
–gc：クリーンアップのタスクを実行する
–minify：ファイルを小さくする
–cleanDestinationDir：不要なファイルは削除する
```

### 3.4 コンテンツのアップロード

ターミナルで以下のコマンドを実行すると、/kaithems/public 以下のファイルがアップロード（更新）される。

```
> rclone sync -v ./public/ sakura:/home/sh-center/www/
```

### 3.5 Web site の確認

ブラウザで以下のURLをアクセスし、変更内容を確認する。

```
https://sh-center.org
```

### 3.6 npm

npmがインストールされていれば、以下のpackage.json を作るとコマンド入力が容易になる

```
{
  "name": "hugo",
  "version": "1.0.0",
  "description": "",
  "scripts": {
    "build": "hugo --gc --minify",
    "sync": "rclone sync -v ./public/ sakura:/home/sh-center/www/",
    "serv": "hugo server --gc -D"
  }
}
```

> npm run build でビルド
> npm run sync でアップロード
> npm run serv でローカルサーバーが立ち上がる

## 4. メニューの修正

メニュー修正は、/kaithems/hugo.toml ファイルを編集する

## 5. ホームページの修正

ホームページの修正は、/kaithems/layouts/index.html を編集する

## 6. 「お問い合わせ」について

お問い合わせページはGoogle formを利用している。

### 6.1 お問い合わせページの修正方法

- Chromeを起動し、以下のアカウントでログインする。
  - user: contactkaithems@gmail.com
- 右上のGoogleアプリ（９個のdots:Google apps）からフォームを選択
- 最近使用したフォームから「お問い合わせ」を選択する
- 必要に応じて修正する
- 公開をクリック

## 6.2 メールの転送先の修正方法

「お問い合わせ」に入力があった場合、contactkaithems@gmail.com宛にメールが送信され、contact@sh-center.orgに転送される。
contact@sh-center.orgで受信したメールは、担当者へ転送される。したがって、「お問い合わせ」に入力があった場合の受信先を修正する場合は、さくらインターネットのレンタルサーバーで、メールアドレスcontact@sh-center.orgの転送先設定を修正することになる。
