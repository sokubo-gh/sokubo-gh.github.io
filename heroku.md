Heroku
======

関連リンク
----------

 * [Heroku](https://heroku.com)


WSL(Windows Subsystem for Linux)にherokuクライアント環境を構築
--------------------------------------------------------------

  * 「コントロールパネル」、「プログラムのアンインストール」、「Windowsの機能の有効化または無効化」
  * 「Windows Subsystem for Linux」にチェックを入れてOKしWindows再起動
  * ストアアプリから「debian」で検索してDebian GNU/Linuxをインストールして起動
  ```bash
  Installing, this may take a few minutes…
  Please create a default UNIX user account. The username does not need to match your Windows username.
  For more information visit: https://aka.ms/wslusers
  Enter new UNIX username: (ユーザ名)  # 一般ユーザ名を入力
  New password: (パスワード)  # 設定したいパスワードを入力
  Retype new password: (パスワード)  # 設定したいパスワードを入力(2回目)
  passwd: password updated successfully
  Installation successful!
  
  $ sudo passwd  # rootのパスワードを変更
  New password: (パスワード)  # rootに設定したいパスワードを入力
  Retype new password: (パスワード)  # rootに設定したいパスワードを入力(2回目)
  passwd: password updated successfully
  
  $ su -
  Password: (パスワード)
  $ apt update  # パッケージ定義を更新
  $ apt upgrade  # パッケージを最新化
  $ cat /etc/debian_version  # マイナーバージョンを確認
  10.3
  ```
  * 必要なパッケージをインストール
  ```bash
  $ apt install curl gnupg git
  ```
  * Heroku CLIをインストール
  ```bash
  $ curl https://cli-assets.heroku.com/install-ubuntu.sh | sh
  ```


PHPアプリをデプロイ(WSL Debian)
-------------------------------

  * phpファイルを適当のフォルダに作成しておく
  * 必要なパッケージをインストール
  ```bash
  $ apt install php php-mbstring composer
  ```
  * git初期化と最初のコミット
  ```bash
  $ git init
  $ git add .
  $ git config --global user.email "(メールアドレス)"
  $ git config --global user.name "(名前)"
  $ git add .
  $ git commit -m "first commit"
  ```
  * Herokuアプリの作成
  ```bash
  $ heroku create (アプリ名)
  $ git push heroku master
  ```
  * `https://(アプリ名).herokuapp.com`にアクセス


  * composer定義
  ```bash
  $ vi composer.json
  {
        "require": {
                "ext-mbstring": "*"
        }
  }
  $ composer update  # composer.lockが作成される
  $ vi .gitignore
  vendor/
  $ git add .
  $ git commit -m "composer"
  $ git push heroku master
  ```


PHPアプリをデプロイ(Windows, NetBeans)
--------------------------------------

  * Windows上の必要アプリケーションをインストール
    * OpenJDK
    * NetBeans
    * Git
    * PHP
    * Composer
    * Heroku CLI
  * NetBeans上でPHPアプリケーションのプロジェクトを作成
    * 必要ならComposerの定義もしておく
  * プロジェクトフォルダでgit初期化
  ```
  (プロジェクトフォルダ)> git init
  ```
  * .gitignoreファイルを作成
  ```
  vendor/
  nbproject/
  ```
  * Herokuアプリケーションを作成(git initの後に実行)
  ```
  (プロジェクトフォルダ)> heroku create (アプリケーション名)
  ```
  * gitコミット
  ```
  (プロジェクトフォルダ)> git add .
  (プロジェクトフォルダ)> git commit -m "first commit"
  ```
  * Herokuへデプロイ
  ```
  (プロジェクトフォルダ)> git push heroku master
  ```
  * `https://(アプリ名).herokuapp.com`にアクセス


NetBeans WebアプリケーションをHerokuに配備
------------------------------------------

  * WindowsにJDK、NetBeans、Tomcatをインストール
    * Tomcatはローカルでの動作確認用
    * Tomcatのmanager-scriptロールにユーザを追加しておく
    * Tomcatをサービスとして登録する場合、サービスのログオンを「Local System account」に変更
  * NetBeansの「File」メニューから「New Project」を選択
  *「Choose Project」の「Java with Maven」から「Web Application」を選択しプロジェクトを作成
  * プロジェクトを作成され開いたら、「pom.xml」に以下を追加
  ```
  <build>
      ...
      <plugins>
          ...
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-dependency-plugin</artifactId>
              <executions>
                  <execution>
                      <phase>package</phase>
                      <goals><goal>copy</goal></goals>
                      <configuration>
                          <artifactItems>
                              <artifactItem>
                                  <groupId>com.heroku</groupId>
                                  <artifactId>webapp-runner</artifactId>
                                  <version>9.0.30.0</version>
                                  <destFileName>webapp-runner.jar</destFileName>
                              </artifactItem>
                          </artifactItems>
                      </configuration>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```
    * ビルド時に「<artifactId>maven-dependency-plugin</artifactId>」が重複しているため警告がでるが、とりあえずこのまま
  * プロジェクトの直下に「Procfile」を作成(拡張子なし)
  ```
  web: java $JAVA_OPTS -jar target/endorsed/webapp-runner.jar --port $PORT target/*.war
  ```
    * [Heroku公式](https://devcenter.heroku.com/articles/java-webapp-runner)では、「target/dependency/webapp-runner.jar」となっているが、ここでは変えておく
    * 変更しない場合、Herokuでの実行時にwebapp-runner.jarが見つからずエラーになる
    ```
    Error: Unable to access jarfile target/dependency/webapp-runner.jar
    ```
    * Procfileのファイル名を間違うとビルド時にアプリケーションのタイプ判別に失敗する
    ```
    remote: -----> Discovering process types
    remote:        Procfile declares types -> (none)
    ```
  * gitの初期化
  ```
  (プロジェクトフォルダ)> git init
  ```
  * 「nb-configuration.xml」を.gitignoreへ追加
  * Herokuアプリケーションの作成
  ```
  (プロジェクトフォルダ)> heroku create (アプリケーション名)
  ```
  * gitへのコミット
  ```
  (プロジェクトフォルダ)> git add .
  (プロジェクトフォルダ)> git commit -m "first commit"
  ```
  * Herokuへの配備
  ```
  (プロジェクトフォルダ)> git push heroku master
  ```


WicketアプリケーションをHerokuに配備
------------------------------------

  * [Apache Wicket Quickstart](https://wicket.apache.org/start/quickstart.html)のウィザードでMavenのコマンドを生成し実行
  * NetBeansで「Open Project...」で作成されたフォルダを開く
  * プロジェクト名が「Quickstart」になっているため必要なら変名する
  * プロジェクトを作成され開いたら、「pom.xml」に以下を追加
  ```
  <build>
      ...
      <plugins>
          ...
          <plugin>
              <groupId>org.apache.maven.plugins</groupId>
              <artifactId>maven-dependency-plugin</artifactId>
              <executions>
                  <execution>
                      <phase>package</phase>
                      <goals><goal>copy</goal></goals>
                      <configuration>
                          <artifactItems>
                              <artifactItem>
                                  <groupId>com.heroku</groupId>
                                  <artifactId>webapp-runner</artifactId>
                                  <version>9.0.30.0</version>
                                  <destFileName>webapp-runner.jar</destFileName>
                              </artifactItem>
                          </artifactItems>
                      </configuration>
                  </execution>
              </executions>
          </plugin>
      </plugins>
  </build>
  ```
  * プロジェクトの直下に「Procfile」を作成(拡張子なし)
  ```
  web: java $JAVA_OPTS -jar target/dependency/webapp-runner.jar --port $PORT target/*.war
  ```
    * こちらの場合は「target/dependency/webapp-runner.jar」のままで問題ない
  * JDKのバージョンを変更(HerokuのJavaのデフォルトは1.8)するためにpom.xmlを修正
  ```
  <build>
      ...
      <plugins>
          ...
          <plugin>
              ...
              <groupId>org.apache.maven.plugins</groupId>
              ...
              <configuration>
                  <source>15</source>
                  <target>15</target>
              ...
          </plugin>
          ...
      </plugins>
  </build>
  ```
  * プロジェクトの直下に「system.properties」を作成(拡張子なし)
  ```
  java.runtime.version=15
  ```
  * gitの初期化
  ```
  (プロジェクトフォルダ)> git init
  ```
  * 「nb-configuration.xml」を.gitignoreへ追加
  * Herokuアプリケーションの作成
  ```
  (プロジェクトフォルダ)> heroku create (アプリケーション名)
  ```
  * gitへのコミット
  ```
  (プロジェクトフォルダ)> git add .
  (プロジェクトフォルダ)> git commit -m "first commit"
  ```
  * Herokuへの配備
  ```
  (プロジェクトフォルダ)> git push heroku master
  ```


ＰythonアプリケーションをHerokuに配備(Termux, GitHub, Python, Heroku)
---------------------------------------------------------------------

  * Pythonアプリケーションを作成
  ```
  $ nano hello.py
  $ echo '# hello' >> README.md
  ```
  * gitリポジトリの初期化とコミット
  ```
  $ git init
  $ git add .
  $ git config --global user.email "sokubo55@protonmail.com"
  $ git config --global user.name "sokubo"
  $ git commit -m "first commit"
  $ git branch -M main
  ```
  * SSHを利用してGitHubへコミット
  ```
  $ pkg install openssh
  $ ssh-keygen -t rsa
  $ cat .ssh/id_rsa.pub
  ```
  * id_rsa.pubの内容をGitHubに登録
  ```
  $ git remote set-url origin  git@github.com:sokubo55/hello.git
  $ git remote -v
  $ git push origin main
  ```
  * Herokuに必要なファイルの作成
  ```
  $ pip install gunicorn
  $ pip freeze -l > requirements.txt
  $ echo "web: gunicorn hello:app --log-file -" > Procfile
  $ echo "python-3.9.2" > runtime.txt
  ```
  * GitHubにコミット
  ```
  $ git add .
  $ git commit -m 'heroku'
  $ git push origin main
  ```
  * Herokuでアプリケーションを作成しGitHubリポジトリに接続


