MDwiki
======

関連リンク
----------

  * [MDwiki 公式](http://dynalon.github.io/mdwiki/)
  * [MDwiki Quick Start](http://dynalon.github.io/mdwiki/#!quickstart.md)
  * [MDwiki Customizing](http://dynalon.github.io/mdwiki/#!customizing.md)
  * [MDwiki ダウンロード](https://github.com/Dynalon/mdwiki/releases)
  * [nginx 公式](http://nginx.org/)
  * [nginx ダウンロード](http://nginx.org/en/download.html)
  * [Heroku](http://heroku.com/)


セットアップ
------------

  * MDwikiダウンロード(GitHub)から「mdwiki-0.6.2.zip」をダウンロード
  * 「mdwiki-0.6.2.zip」を展開
  * 「mdwiki.html」をWebサーバに配置
    ファイル名は変更可能、「index.html」としておけばURLからファイル名を省略可能
  * 「mdwiki.html」と同じフォルダに「index.md」ファイルを作成
    htmlファイルにアクセスすると「～.html#!index.md」にリダイレクトされる
  * 設定は「config.json」を作成、ページトップのメニューは「navigation.md」を作成


記述方法
--------

  * 「mdwiki.hml」と同じフォルダにMarkdown形式でmdファイルを配置
  * Markdown形式の書き方は[MDwiki Quick Start](http://dynalon.github.io/mdwiki/#!quickstart.md)を参照
  * 「mdwiki.html#!～.md」で各mdファイルを表示可能
  * 別のmdファイルのリンクは以下の形式で記載
  ```markdown
  [リンクテキスト](～.md)
  ```


設定方法
--------

  * テーマ
    * ページトップのメニューを定義する「navigation.md」の中で定義
    * 決まったテーマを適用するには以下のように記載
    ```markdown
    [gimmick:theme](テーマ名)
    ```
    * テーマ選択をする場合は以下のように記載
    ```markdown
    [gimmick:themechooser](テーマ変更)
    ```
    * テーマ一覧は[MDwiki 公式](http://dynalon.github.io/mdwiki/)の右上の「Change Theme」で確認できる


  * その他設定
    * 「config.json」でパラメタ設定することでカスタマイズ可能
    * 「title」:`<title>`として定義され、ページトップのメニューの一番左にも表示
    * 「additionalFooterText」:フッターの前半部分
    ```json
    {
        "title": "memo",
        "additionalFooterText": "&copy; sokubo, "
    }
    ```


Windows版nginx
--------------

基本的には、ZIPファイルをダウンロードして展開するだけでOK

  * [nginx ダウンロード](http://nginx.org/en/download.html)からZIPファイルをダウンロード
  * ZIPファイルを任意の場所に展開
  * htmlフォルダに「mdwiki.html」、「config.json」、MDファイルを配置
  * コマンドプロンプトから`start nginx`でWebサーバ起動
  * コマンドプロンプトから`nginx -s stop`でWebサーバ停止



コマンドプロンプトからの入力が面倒、ドキュメントルートが「(nginxフォルダ)\html」となり面倒なので、バッチファイルの作成、フォルダ構成の見直し


|No.|フォルダ|内容|
|:-:|:-|:-|
|1|(任意のフォルダ)|起動・停止バッチを配置|
|2|(任意のフォルダ)\nginx|nginxのZIPファイルの中身|
|3|(任意のフォルダ)\html|mdwiki.html、config.json、mdファイル|

  * No.1に起動用バッチファイルの作成
  ```
  @echo off

  cd (任意のフォルダ)\nginx\
  start (任意のフォルダ)\nginx\nginx.exe

  echo nginx started > (任意のフォルダ)\nginx.txt
  date /t >> (任意のフォルダ)\nginx.txt
  time /t >> (任意のフォルダ)\nginx.txt
  ```
  *  No.1に停止用バッチファイルの作成
  ```
  @echo off

  del (任意のフォルダ)\nginx.txt

  cd (任意のフォルダ)\nginx\
  (任意のフォルダ)\nginx\nginx.exe -s stop
  ```
  * No.2の配下のconfフォルダの「nginx.conf」ファイルを変更
  ```
  server {
      listen       80;
      server_name  localhost;
  # (省略)
      location / {
          #root   html;
          root   (任意のフォルダ)/html;  # No.3フォルダ、"\"(半角￥)は"/"に変えて記載
          index  index.html index.htm;
      }
  # (省略)
  ```


Linux(WSL,Debian)版nginx
------------------------
Windows Subsystem for LinuxにDebianを入れた環境が前提

  ```bash
  $ apt install nginx-light  # インストール
  (/var/www/htmlにmdwiki.html, config.json, *.mdファイルを配置)
  $ /etc/init.d/nginx start  # nginx起動
  $ /etc/init.d/nginx stop   # nginx停止
  ```


Heroku
------
Heroku CLI, gitインストールされた環境が前提

  * 任意のディレクトリでmdiwiki.html, config.json, *.mdファイルを作成
  * 同ディレクトリに以下の「index.php」ファイルを作成
  ```php
  // index.php
  <?php
      header('Location: /mdwiki.html');
  ?>
  ```
  * Herokuにデプロイ
  ```bash
  $ cd (mdiwiki.html, config.json, *.mdファイル格納先)
  $ git init
  $ git config --global user.email "(メールアドレス)"
  $ git config --global user.name "(名前)"
  $ git add .
  $ git commit -m "first commit"
  $ heroku create (アプリ名)
  $ git push heroku master
  ```


GitHub Pages
------------

  * GitHub Pages用のリポジトリ(\[username\].github.io)を作成
  * リポジトリにREADME.mdを作成
  * mdwiki.html, config.json, *.mdファイルを格納
