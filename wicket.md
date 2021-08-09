Apache Wicket
=============

関連リンク
----------

  * [Apache Wicket](https://wicket.apache.org/)
  * [Heroku](https://heroku.com/)
  * [OpenJDK](https://openjdk.java.net/)
  * [Apache NetBeans](https://netbeans.apache.org/)
  * [Apache Tomcat](http://tomcat.apache.org/)


サンプルアプリケーション(Quickstart)をHerokuに配備
-----------------------------------------------

  * 前提
    * OpenJDK 15.0.2
    * Apache NetBeans 12.4
    * Apache Maven 3.6.3(NetBeans同梱)
    * Apache Tomcat 9.0.48(ローカルテスト用)


  * サンプリアプリケーションの作成
    * [Apache Wicket](https://wicket.apache.org/)の[QUICK START](https://wicket.apache.org/start/quickstart.html)でMavenコマンドを作成・実行
      * Group IDは任意(例:sokubo)
      * Artifact IDはプロジェクト名(例:TestWicket)
      * Wicket Versionは9.x系を選択(Wicket 9.xからはJDK 11以上)
      * 表示されたMavenコマンドを実行(実行時のフォルダにプロジェクトのフォルダが作成)
    * プロジェクトの微修正
      * NetBeansでプロジェクトを開く際に作成されたフォルダを選択
        * プロジェクトのプロパティ、一般にある名前が「quickstart」となっている場合は、プロジェクト名(例:TestWicket)に修正
        * プロジェクトのプロパティ、ビルド、RunのServerで導入済のTomcatを選択
        * プロジェクトのプロパティ、ビルド、RunのJava EE ServerでJava EE 7 Webを選択(デフォルト)
        * プロジェクトのプロパティ、ビルド、RunのContext Pathが空欄の場合、プロジェクト名(例:TestWicket)に修正
    * ローカル環境での動作確認
      * NetBeansから実行して、ローカルのTomcat上で起動を確認


  * Heroku用のJavaアプリケーションの作成
    * 「pom.xml」に以下を追加
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


  * Gitへの登録
    * ローカルリポジトリの初期化
    ```
    (プロジェクトフォルダ)> git init
    ```
    * ローカルリポジトリへのコミット
    ```
    (プロジェクトフォルダ)> git add .
    (プロジェクトフォルダ)> git commit -m "first commit"
    ```


  * GitHubへの登録
    * GitHub上でリポジトリを作成(例:TestWicket)
    * ローカルリポジトリからGitHubへPush
    ```
    (プロジェクトフォルダ)> git remote add origin https://github.com/sokubo-gh/TestWicket.git
    (プロジェクトフォルダ)> git branch -M main
    (プロジェクトフォルダ)> git push -u origin main
    ```


  * Herokuアプリケーションの作成
    * Herokuにてアプリケーションを作成(例：sokubo-testwicket)
    * 作成したアプリケーションのDeployment methodにGitHubを選択
    * GitHub上のリポジトリを選択してDeploy Branchボタンをクリックして配備
