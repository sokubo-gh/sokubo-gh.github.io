Java EE(Java EE, GlassFish, NetBeans)
=====================================

Servlet, BASIC認証
------------------
  * web.xml
    * セキュリティタブを選択
    * ログイン構成は基本(BASIC認証)を選択
    * レルム名を設定(別途、GlassFish管理コンソールで設定する)
    * 制約に、URLパターン(設定方法は資料を見た方が良い)、ロール名を設定

  * glassfish-web.xml
    * WEB-INFでGlassFishディスクリプタを追加
    * セキュリティタブでweb.xmlで指定したロールにプリンシパル(ユーザ名)を設定

  * Realm
    * GlassFish管理コンソールでweb.xmlで指定したレルムを追加
    * ファイルレルムを追加
    * Manage Usersでglassfish-web.xmlで指定したプリンシパルをユーザ名にしてユーザを追加


Session Bean、@Scheduleアノテーション(タイマー)
-----------------------------------------------
対象となるのはStateless Session Bean, Singleton Session Bean
メソッドに「@schedule()」アノテーションを追加する

  * @Schedule(hour="13") : 毎日13:00:00に実行
  * @Schedule(hour="*", minute="*/10") : 10分毎に実行
  * @Schedule(minute="15", persistent = false) : 次の00:15:00に実行、サーバーが再起動したら無効
  * @Schedules({@Schedule(hour="2"), @Schedule(hour="4", dayOfWeek="Sat")}) : 毎日2:00:00に実行、土曜日は4:00:00にも実行

|属性名|省略値|指定可能な値|
|:-|:-|:-|
|year|*|4桁数字/*|
|month|*|1～12/Jan/Feb/Mar/Apr/May/Jun/Jul/Aug/Sep/Oct/Nov/Dec/*|
|dayOfMonth|*|1～31/*|
|dayOfWeek|*|0～7/Sun/Mon/Tue/Wed/Thu/Fri/Sat/*|
|hour|0|0～23/*|
|minute|0|0～59/*|
|second|0|0～59/*|

  * 時分秒については省略値すると0になるため、毎時XX分に実行とする場合にもhourも指定する必要有り
  * タイマー時刻の精度はあまり正確ではなく、数秒～数十秒のずれがある(GlassFish V3サーバーで確認)
  * OSの時刻を補正しても次のタイマー時刻までは反映されない(GlassFish V3サーバーで確認)
  * 「persistent = false」の指定が無ければ、アプリケーションサーバーが再起動しても有効
  * Stateless Session Beanであっても対象のメソッド処理中は、次のタイマー時刻になっても実行されない(おそらくメソッド終了時に次のタイマーがセットされる)


GlassFishライフサイクルモジュール
---------------------------------
GlassFishサーバー起動・停止に合わせて何か処理を行いたい場合、LifeCycleモジュールを利用する。

  * NetBeansのライブラリにGlassFish API用のライブラリを登録
    「ツール」、「ライブラリ」から「(GlassFishインストール先)/modules/glassfish-api.jar」をライブラリとして登録
  * プロジェクト作成と利用ライブラリ設定
    「Javaクラスライブラリ」プロジェクトを新規作成し、「プロパティ」、「ライブラリ」にglassfish-api.jarを設定
  * インターフェースLifecycleListenerを実装
  ```java
  public class LifecycleClass implements LifecycleListener {
      public void handleEvent(LifecycleEvent le){
          switch (le.getEventType()){
              case LifecycleEvent.INIT_EVENT:
                  break;
              case LifecycleEvent.STARTUP_EVENT:
                  break;
              case LifecycleEvent.READY_EVENT:
                  break;
              case LifecycleEvent.SHUTDOWN_EVENT:
                  break;
              case LifecycleEvent.TERMINATION_EVENT:
                  break;
          }
      }
  }
  ```
    * サーバー起動時には、INIT_EVENT, STARTUP_EVENT, READY_EVENTの順で呼ばれる
    * サーバー停止時には、SHUTDOWN_EVENT, TERMINATION_EVENTの順で呼ばれる(当然、サーバーをkillした場合は呼ばれない)
  * GlassFishサーバーにLifeCycleモジュールを登録
    * jarファイルをGlassFishサーバーが動作しているサーバー上にコピー
    * 任意の場所で良いがLifeCycleモジュール登録時にクラスパスの設定を不要にするには「(GlassFishインストール先)/lib」に配置する
  * コマンドを利用する場合
    「asadmin create-lifecycle-module」コマンド
  * 管理Webを利用する場合
    * http://(GlassFishサーバのホスト名/IPアドレス):4848にアクセス
    * ライフサイクルモジュールメニューから新規追加を選択
    * 名前:ライフサイクルモジュールを特定する名前
    * クラス名:LifecycleListenerをimplementsしたクラス
    * クラスパス:作成したjarファイルのフルパス


Message Driven Bean
-------------------
  * エンタープライズアプリケーションプロジェクトを作成する
    EJBモジュール、Webアプリケーションモジュールも同時に作成
  * EJBモジュールプロジェクト内に、メッセージ駆動型Beanを作成
    プロジェクトの送信先にキュー or トピックのJNDI名(例:jms/TestQueue, jms/TestTopic)を入力
  * MDBのonMessage()を実装する
    messageをTextMessageにキャストしてgetText()でメッセージテキストを取得する
    テキスト以外の場合ObjectMessageにキャストしてgetObject()でメッセージオブジェクトを取得する
  * 送信元のSession Beanを作成する
    適当なメソッドを実装し、コードを挿入からJMSメッセージの送信を選択する
    接続ファクトリに適当な接続ファクトリ用のJNDI名(例:jms/TestQueueFactory)を入力する
    sendJMSMessageToXXXXXXXX()メソッドにメッセージテキストを渡して送信する
    テキスト以外のメッセージを送信する場合、NetBeansがデフォルトで生成するcreateJMSMessageForXXXX()を修正してObjectMessageを送信する
  * 適当なサーブレットを作成し、送信元Session Beanを呼び出す


