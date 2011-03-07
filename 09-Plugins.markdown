プラグイン
==========

Swift Mailer によって提供されるプラグインはシンプルなクラス継承がより複雑になる方法でプラグインのふるまいを拡張するために使います。

AntiFlood プラグイン
--------------------

多くの SMTP サーバーには単独の SMTP コネクションのあいだに送ることのできるメッセージ数の制限があります。AntiFlood プラグインは膨大な数のメールを管理しながらこの制限の範囲にとどまる方法を提供します。

単独のコネクションのよくある制限は100通のメールです。サーバーにこのような制限を課すために、その制限数のメールが送られた後でコネクションを解除することが要求されます。ループと手作業でこれを何とかすることはできますが、 AntiFlood プラグインはこのロジックに悩まなくてすむように必要なラッパーコードを提供します。

サーバーによって課せられた制限にかかわらず、SMTP サーバーのリソースに対して保守的であることは通常はよいアイディアです。サーバーが過剰に使われていると送信が遅くなりますので AntiFlood プラグインを使うことは制限が存在しなければわるいアイディアにはなりません。

AntiFlood プラグインの基本ロジックはプラグインで指定した X 回のメールの数を送信するたびに SMTP サーバーとのコネクションを解除してからすぐに確立することです。

キューを処理してある程度のリソースを回復させる機会を SMTP サーバーに提供するために短い時間 (たとえば100通のメールごとに30秒) のポーズをするのはよいアイディアです。

### AntiFlood プラグイン

AntiFlood プラグインはほかのすべてのプラグインと同じように Swift  Mailer クラスの `registerPlugin()` メソッドで登録されます。これは2つのコンストラクタパラメータをとります: ポーズするまでの送信するメールの数と、オプションとしてポーズする秒数

AntiFlood プラグインを使うには:

 * 選んだ Transport を使って Mailer のインスタンスを作ります。

 * 1つもしくは2つのコンストラクタパラメータを渡して `Swift_Plugins_AntiFloodPlugin` クラスのインスタンスを作ります。

 * Mailer の `registerPlugin()` メソッドを使ってプラグインを登録します。

 * 通常どおりにメッセージを送るために Swift Mailer を使って、続けます。

Swift Mailer がメッセージを送るとき、最後のコネクションから送られたメッセージの数をカウントします。カウント数が指定したしきい値に到達すると、コネクションの切断と再確立を行い、オプションとして指定された秒数だけ停止します。

`batchSend()` と `send()` メソッドの両方はこのプラグインで動きます。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance(
      Swift_SmtpTransport::newInstance('smtp.example.org', 25)
    );

    //100通のメールの後でコネクションを再確立するために AntiFlood を使います
    $mailer->registerPlugin(new Swift_Plugins_AntiFloodPlugin(100));

    //もしくは停止時間を指定します (30秒)
    $mailer->registerPlugin(new Swift_Plugins_AntiFloodPlugin(100, 30));

    //通常どおりメールの送信を継続します
    for ($lotsOfRecipients as $recipient) {
      ...

      $mailer->send( ... );
    }

Throttler プラグイン
--------------------

SMTP サーバーにはメール送信時のレート制限があるので、プログラムのコードはこのレート制限を認識する必要があります。Throttler プラグインは Swift Mailer をレート制限の速度で動作させます。

多くの共用ホストは SMTP サーバーを飛び入りでオープンすることを許可しません。(スパマーを思いとどまらせるために) 通常のホストの方針では 時間/1日単位で定められた数のメールだけ送信することが許可されます

Throttler プラグイン律速の2つのモードをサポートし、値を算出するために算数を行うことが必要です。プラグインは分単位のメールの数もしくは分単位のバイトの転送量にもとづいて制限できます。

### Throttler プラグインを使う

Throttler プラグイン &#8211; はほかのすべてのプラグインのように &#8211; Mailer クラスの `registerPlugin()` メソッドで追加されます。律速を実行する方法を伝える
必須のコンストラクタパラメータは2つです。

Throttler プラグインを使うには:

 * 選んだトランスポートを使って Mailer のインスタンスを作ります。

 * `Swift_Plugins_ThrottlerPlugin` クラスのインスタンスを作り、使っているモードと一緒に、制限したいメールの数もしくはバイト数を渡します。

 * Mailer の `registerPlugin()` メソッドを使ってプラグインを登録します。

 * 通常どおりメッセージを送るために Swift Mailer を使い続けます。

Swift Mailer がメールを送信したとき、メッセージ送信が行われる速度を追跡します。送信の頻度が多すぎることが判明したら、平均速度になるように `sleep()` が実行されます。

`batchSend()` と `send()` メソッドの両方ともこのプラグインと連携します。

    [php]
    require_once 'lib/swift_required.php';

    //トランスポートを使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance(
      Swift_SmtpTransport::newInstance('smtp.example.org', 25)
    );

    //分単位の制限を 100 通にします
    $mailer->registerPlugin(new Swift_Plugins_ThrottlerPlugin(
      100, Swift_Plugins_ThrottlerPlugin::MESSAGES_PER_MINUTE
    ));

    //分単位の制限を 10MB にします
    $mailer->registerPlugin(new Swift_Plugins_ThrottlerPlugin(
      1024 * 1024 * 10, Swift_Plugins_ThrottlerPlugin::BYTES_PER_MINUTE
    ));

    //通常どおりに送信を続けます
    for ($lotsOfRecipients as $recipient) {
      ...

      $mailer->send( ... );
    }

### Logger プラグイン

Logger プラグインは送信プロセスのあいだのデバッグ作業を支援します。SMTP サーバーがアドレスを拒否するのはなぜか、もしくは見つけずらい問題を特定することの助けになります。

Logger プラグインは2つのパーツからなります。このプラグインに加えて、無数の可能性のあるロガーの１つを選べます。たとえば、ロガーがリアルタイムでメッセージを出力できたり、メッセージを配列形式でキャプチャできます。

ほかには Exception メッセージを変更する機能がよく知られています。Exceptions が投げられるが、エラーメッセージが問題のソースに関する決定的な情報を提供しません  (あいまいな SMTP エラーなど) Logger プラグイン SMTP のトランスクリプト全体をエラーメッセージに含めるので、デバッグ作業はシンプルなタスクになります。

Swift Mailer に含まれる利用可能なロガーはいくつかあります。独自の実装を書く作業は信じられないぐらいシンプルで `Swift_Plugins_Logger` インターフェイスを実装する短いクラスを作ることで実現されます。

 * `Swift_Plugins_Loggers_ArrayLogger`: Keeps a collection of log messages inside an array。配列の内容はクリアされるかスクリーンに吐き出されます。

 * `Swift_Plugins_Loggers_EchoLogger`: はリアルタイムでスクリーンに出力を表示します。初歩的なデバッグ出力にとても手軽です。

#### Logger プラグインを使う

Logger プラグインはほかのすべてのプラ群と同じように、Mailer
クラスの `registerPlugin()` メソッドで追加されます。このクラスはコンストラクタのなかで `Swift_Plugins_Logger` のインスタンスで受け入れます。

Logger プラグインを使うには:

 * 選んだトランスポートを使ってメーラーのインスタンスを作ります。

 * `Swift_Plugins_Logger` の Logger 実装のインスタンスを作ります。

 * `Swift_Plugins_LoggerPlugin` クラスのインスタンスを作り、Logger インスタンスをコンストラクタに渡します。

 * Mailer の `registerPlugin()` メソッドを使ってプラグインを登録します。

 * 通常どおりメッセージを送るために Swift Mailer を使い続けます。

 * ロガーの `dump()` メソッドでログの内容を吐き出します。

Swift Mailer がメッセージを送信するとき、内蔵される Transport とのやりとりのすべてのログはすべて保存されます。Logger によってふるまいは異なりますが、すべての実装はログの内容を得るための手段を提供します。

    [php]
    require_once 'lib/swift_required.php';

    //トランスポートを使ってメーラーを作ります
    $mailer = Swift_Mailer::newInstance(
     Swift_SmtpTransport::newInstance('smtp.example.org', 25)
    );

    //ArrayLogger を使うには
    $logger = new Swift_Plugins_Loggers_ArrayLogger();
    $mailer->registerPlugin(new Swift_Plugins_LoggerPlugin($logger));

    //もしくは Echo Logger を使うには
    $logger = new Swift_Plugins_Loggers_EchoLogger();
    $mailer->registerPlugin(new Swift_Plugins_LoggerPlugin($logger));

    //通常どおり送信を続けます
    for ($lotsOfRecipients as $recipient) {
     ...

     $mailer->send( ... );
    }

    // ログの内容を吐き出します
    // NOTE: EchoLogger はリアルタイムで吐き出すので、dump() は何も行いません。
    echo $logger->dump();

### Decorator プラグイン

複数の受信者に同じメッセージを送信する必要があるが、メッセージボディ内部で使われる受信者の名前など、バリエーションがささいである場合がよくあります。Decorator プラグインはこれらの小さな違いに対するソリューションを提供することを目的としています。

Decorator プラグインは Swift Mailer の送信プロセスを傍受することで機能します。To: フィールドのメールアドレスを読み込み、テンプレート用の置き換えのセットを探します。

このプラグインの使い方はシンプルですが、動作方法が原因でおそらくはもっとも誤解されがちなプラグインです。ユーザーがやらかす典型的な間違いはプラグインを複数回登録しようとすることです (それぞれの受信者に1回) &#8211; たとえばループ内部に入れてしまうなどです。これは正しくありません。

Decorator プラグインは一度だけ登録されますが、送信に先駆けて、すべての受信者が含まれます。受信者のリストは送信のあいだの必須の置き換えを探すために使われます。

#### Decorator プラグインを使う

Decorator プラグインを使うには、メールアドレスにもとづいた置き換えの連想配列を作り、プラグインを追加するメーラーの `registerPlugin()` メソッドを使います。

最初に、メッセージを送信する相手のメールアドレスにもとづいて置き換えの連想配列を作ります。

>**NOTE**
>置き換えの配列は二次元配列になります。キーはメールアドレスで、値はそのメールアドレスに対する置き換えの連想配列です。メールテンプレートのなかのプレースホルダにこれらがマッチするという条件のもとで、この例に使われている波かっこは選んだ任意の構文のタイプになります。

    [php]
    $replacements = array();
    foreach ($users as $user) {
      $replacements[$user['email']] = array(
        '{username}'=>$user['username'],
        '{password}'=>$user['password']
      );
    }

置き換えの配列を使ってDecorator プラグインのインスタンスを作り、Mailer で登録します。一度だけです！

    [php]
    $decorator = new Swift_Plugins_DecoratorPlugin($replacements);

    $mailer->registerPlugin($decorator);

メッセージを作るとき、ボディ (かつ/もしくは件名) の要素をプレースホルダに置き換えます。

    [php]
    $message = Swift_Message::newInstance()
      ->setSubject('Important notice for {username}')
      ->setBody(
        "Hello {username}, we have reset your password to {password}\n" .
        "Please log in and change it at your earliest convenience."
      )
      ;
  
    foreach ($users as $user) {
      $message->addTo($user['email']);
    }

`$replacements` 配列に並べられているそれぞれの受信者にこのメッセージを送信するとき、彼らは自分たちのためにカスタマイズされたメッセージを受けとるだけです。たとえば、上記のメッセージが受信されたときににはあるユーザーには次のように表示されます。

    Subject: Important notice for smilingsunshine2009

    Hello smilingsunshine2009, we have reset your password to rainyDays
    Please log in and change it at your earliest convenience.

一方で別の使い方ではメッセージを次のように受けとります。

    Subject: Important notice for billy-bo-bob

    Hello billy-bo-bob, we have reset your password to dancingOctopus
    Please log in and change it at your earliest convenience.

Decorator プラグインがこの問題を解決するための手段を提供している一方で、プラグインなしでこの問題に取り組む方法がいろいろあります。私たちは自分達自身でよりよい方法を考えだそうとしており、私たちにはいくつかの (疑う余地のない) アイディアがありますが、先に進んで実装する完璧なソリューションはありません。お見逃しなく。

#### Decorator のために独自の置き換えのルックアップを提供する

配列に置き換えの値を満たすことが Decorator への置き換え情報を提供するためのベストなソリューションにはならないことがあります。即座に置き換えのルックアップを実行するもっとエレガントなアルゴリズムがある場合、自前の実装を提供することができます。

Decorator の自前の置き換えのルックアップ実装を提供することは 配列形式ではなく `Swift_Plugins_Decorator_Replacements` のインスタンスを Decorator プラグインのコンストラクタに渡すという問題です。

Replacements インターフェイスは1つのメソッド: `getReplacementsFor($address)` だけでとてもかんたんに実装できます。

データベースから置き換えを即座に調べたい場合を想像してください。これを行う実装を提供することができます。小さなクラスを作る必要があります。

    [php]
    class DbReplacements implements Swift_Plugins_Decorator_Replacements {
      public function getReplacementsFor($address) {
        $sql = sprintf(
          "SELECT * FROM user WHERE email = '%s'",
          mysql_real_escape_string($address)
        );
    
        $result = mysql_query($sql);
    
        if ($row = mysql_fetch_assoc($result)) {
          return array(
            '{username}'=>$row['username'],
            '{password}'=>$row['password']
          );
        }
      }
    }

必要なことは配列の代わりにあなたのクラスのインスタンスを Decorator プラグインのコンストラクタに渡すことです。

    [php]
    $decorator = new Swift_Plugins_DecoratorPlugin(new DbReplacements());

    $mailer->registerPlugin($decorator);

送信されるそれぞれのメッセージに対して、プラグインは置き換えの配列を探すためにあなたのクラスの `getReplacementsFor()` メソッドを呼び出します。

>**NOTE**
>ルックアップアルゴリズムが大文字小文字を区別する場合、`$address` 引数を適切なものに変換すべきです &#8211; たとえば `strtolower()` 関数を通して引数を渡します。
