メッセージを送信する
=====================

メッセージ送信のクィックリファレンス
-------------------------------------

メッセージの送信方法は単刀直入です。Transport を作り、Mailer を作るために Transport を使い、メッセージを送るために Mailer を使います。

メッセージを送信するには:

 * 提供された Transports の 1つから Transport を作ります &#8211;
   `Swift_SmtpTransport`、
   `Swift_SendmailTransport`、
   `Swift_MailTransport` もしくはアグリゲート Transports の1つ。

 * コンストラクタパラメータとして Transport を使い `Swift_Mailer` クラスのインスタンスを作ります。

 * メッセージを作成します。

 * Mailer オブジェクトの `send()` もしくは
   `batchSend()` メソッドを通じてメッセージを送ります。

`send()` メソッドを使うと、メッセージはメールクライアントを使ったように送られます。連続した数の受信者を含む整数が返されます。受信者が送信できなければ、ブール値の `false` と同等であるゼロが返されます。メッセージのなかで2つの `To:` 受信者と3人の `Bcc:` 受信者とすべての受信者のデリバリは成功し、5の値が返されます。

`batchSend()` メソッドは a number just like `send()`, except that it does not include all of the
`To:` 受信者のすべてを含みません。それぞれの受信者は is sent a unique copy of the message with only their address in the headers。これは This is advised for newsletter systems.

    [php]
    require_once 'lib/swift_required.php';

    //トランスポートを作成します
    $transport = Swift_SmtpTransport::newInstance('smtp.example.org', 25)
      ->setUsername('your username')
      ->setPassword('your password')
      ;

    /*
    代わりに Sendmail もしくは Mail などの異なるトランスポートを使うことができます:

    //Sendmail
    $transport = Swift_SendmailTransport::newInstance('/usr/sbin/sendmail -bs');

    //Mail
    $transport = Swift_MailTransport::newInstance();
    */

    //あらかじめ作っていた Transport を使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

    //メッセージを作成します
    $message = Swift_Message::newInstance('Wonderful Subject')
      ->setFrom(array('john@doe.com' => 'John Doe'))
      ->setTo(array('receiver@domain.org', 'other@domain.org' => 'A name'))
      ->setBody('Here is the message itself')
      ;
  
    //メッセージを送信します
    $result = $mailer->send($message);

    /*
    メッセージを送るのに `batchSend()` メソッドを使うことができます。

    $result = $mailer->batchSend($message);
    */

### トランスポートの種類

Transport は送信を実際に担当するコンポーネントです。Mailer オブジェクトに Transport オブジェクトを提供する必要があります。利用可能な選択肢がいくつかあります。

通常は水面下で Transport がどのように動くのか知る必要はありません。知っておけばよいのは
インスタンスの作り方と環境に対してどれを使うのかです。

#### SMTP トランスポート

SMTP Transport は (標準化された) Simple Message Transfer
Protocol を通じてメッセージを送ります。これは暗号化と認証を扱います。

SMTP Transport、`Swift_SmtpTransport` はうたがないなくもっともよく使われる Transport です。99% の Web サーバーで動くからです (数字をそろえましたが、おわかりいただけることでしょう)。サーバーに必要なことは正しいポート番号 (通常は25)でリモート (もしくはローカルの) SMTP サーバーに接続する機能です。

メールがほかのドメインに送られる前に SMTP サーバーはしばしユーザー名とパスワードで認証するためのユーザーを必要とします。これは Swift Mailer の SMTP Transport を使うことで用意に実現できます。 

SMTP はプロトコルです &#8211; 言い換えると行われるジョブをコミュニケーションする (すなわちメッセージを送る)「方法」です。SMTP プロトコルはインターネットでメッセージを配信するための基本原理です。このためメッセージを送るためのもっとも「直接的な」メソッドで Swift Mailer を使うときに最強の力とフィードバック (デリバリの失敗)をもたらしてくれます。

一般的に SMTP はリモートサービスとして動くので (すなわちネットワーク/インターネットを通じてコネクトします) サーバーからサーバーに対してきわめてポータブルです。SMTP サーバーのアドレスとポート番号をアプリケーションの設定ファイルに保存し SMTP サーバーの変更に応じて設定を調整します。

SMTP サーバーのなかには &#8211; たとえば Google のように &#8211; セキュリティの理由から暗号化を使います。Swift Mailer は SSL と TLS の暗号化設定を両方ともサポートします。

##### SMTP トランスポートを使う

SMTP Transport はかんたんに使うことができます。大半のコンフィギュレーションのオプションはコンストラクタでセットします。

SMTP Transport を使うにはプログラムコードがどの SMTP サーバーに接続するのかを知っていることが必要です。わからなければ自分が利用しているウェブサイトのホストに聞いてください。多くの人が誰にコネクトすればよいのか Swift Mailer の開発者にたずねてきますが &#8211; Iホスト環境に特有の設定なので、開発者は答えようがありません。

SMTP Transport を使うには:

 * サーバーの名前とオプションのポート番号 (デフォルトは 25) `Swift_SmtpTransport::newInstance()` を呼び出します。

 * Mailer を作るために戻り値を使います。

SMTP サーバーへのコネクションは `send()` もしくは `batchSend()` メソッドを最初に呼び出すと確立されます。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作ります
    $transport = Swift_SmtpTransport::newInstance('smtp.example.org', 25);

    //あらかじめ作っておいた Transport を使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

    /*
    複数のメソッド呼び出しを使うこともできます

    $transport = Swift_SmtpTransport::newInstance()
      ->setHost('smtp.example.org')
      ->setPort(25)
      ;
    */

##### 暗号化された SMTP

SMTP Transport をパラメータとして指定するもしくはメソッドを呼び出すことで SSL もしくは TLS 暗号化を使うことができます。

SMTP Transport で暗号化を使うには:

 * `Swift_SmtpTransport::newInstance()` に第3パラメータとして暗号化設定を渡すもしくは

 * Transport の `setEncryption()` メソッドを呼び出します。

SMTP サーバーへのコネクションは `send()` もしくは `batchSend()` メソッドの最初の呼び出しで確立されます。コネクションは正しい暗号化設定で初期化されます。

>**NOTE**
>SSL もしくは TLS 暗号化を機能させるには OpenSSL Transports ラッパーを PHP にインストールしなければなりません。PHP の `stream_get_transports()` 関数を使うことでtls」かつ/もしくは「ssl」が PHP にインストールされているかチェックできます。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作ります
    $transport = Swift_SmtpTransport::newInstance('smtp.example.org', 587, 'ssl');

    //あらかじめ作っておいた Transport を使い  Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

    /*
    メソッドチェーンを使うこともできます

    $transport = Swift_SmtpTransport::newInstance()
      ->setHost('smtp.example.org')
      ->setPort(587)
      ->setEncryption('ssl')
      ;
    */

##### SMTP とユーザー名とパスワード

サーバーのなかには認証を必須とするものがあります。`setUsername()` と `setPassword()` メソッドでユーザー名とパスワードを提供することができます。

SMTP Transport でユーザー名とパスワードを使うには:

 * `Swift_SmtpTransport::newInstance()` で Transport を作ります

 * Transport の `setUsername()` と `setPassword()` メソッドを呼び出します。

Mailer で `send()` もしくは `batchSend()` メソッドが最初に使われるとき、ユーザー名とパスワードは最初のコネクションで認証するために使われます。

認証が失敗すると `Swift_Transport_TransportException` が投げられます。

>**NOTE**
>認証が失敗し Exception が投げられたことを速く理解する必要があれば、作られた Transport の `start()` メソッドを呼び出します。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作り setUsername() と setPassword() を呼び出します。
    $transport = Swift_SmtpTransport::newInstance('smtp.example.org', 25)
      ->setUsername('username')
      ->setPassword('password')
      ;

    //あらかじめ作っておいた Transport を使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

#### Sendmail トランスポート

Sendmail Transport は`sendmail` などのローカルにインストールした MTA &#8211; とコミュニケーションをとることでメッセージを送ります。

Sendmail Transport、`Swift_SendmailTransport` はリモートサービスに直接接続しません。`sendmail` がインストールされている Linux サーバーを前提に設計されています。Transport はローカルな
`sendmail` プロセスを開始し、メッセージをそれに送ります。通常 `sendmail` プロセスはすばやく反応し、メッセージを送る前にディスクにスプールします。

Transport は歴史的な理由から Sendmail Transport から名づけられました (`sendmail` は長いあいだ「標準的な」メール送信のための UNIX ツールでした)。これは名前にもかかわらず、これは sendmail に関連するラッパーである Exim や Postfix などのほかのメール転送エージェントを使って正しいコマンドラインのフラグではじめられるように、メッセージを送信します。

Sendmail Transport はとても素早く結果を返すので、これは受信者に素早くメッセージを配信しなければならないという共通の誤解がありますが、&#8211; これは事実ではありません。これが遅いということはけっしてありませんが、想定された受信者がメッセージを得ることに関しては SMTP よりも速くというわけではありません。sendmail 自身が SMTP を通じてメッセージを送るので、メッセージはディスクに素早くスプールされるからです。

受信者が拒否されたことを Swift Mailer に通知することに関して、Sendmail Transport は SMPT と同じぐらいスマートであるポテンシャルをもっていますが、ローカルにインストールされた `sendmail` インスタンスのマジョリティにおいて、有益なフィードバックを提供するようにコンフィギュレーションはじゅうぶんに調整されていません。そういうわけなので、Swift Mailer は実際には配布物をサーバーに残す前に実際には失敗しているのにもかかわらず、デリバリの成功を報告するかもしれません。

コマンドラインフラグで指定した異なる2つのモードで Sendmail Transport を実行することができます:

  * "`-bs`" は SMTP モードで動きます。理論的には SMTP Transport のようにふるまいます。

  * "`-t`" はフィードバックなしの piped モードで動きます。理論的には速くなりますが、おすすめではありません。

デリバリの失敗に問題がある場合、Sendmail Transport を一種の非同期 SMTP
Transport &#8211; と考えることができます。代わりに SMTP Transport を試すべきです。Swift Mailer はここでは何もしませんが、仕事をほかの誰か (すなわち
`sendmail`) にゆだねます。

##### Sendmail トランスポートを使う

Sendmail トランスポートを使うにはコマンドをパラメータとして
`Swift_SendmailTransport::newInstance()` を呼び出す必要があります。

Sendmail Transport を使うには `sendmail` もしくは別の MTA が存在する場所を知る必要があります。Swift Mailer
は大半のシステムで機能する `/usr/sbin/sendmail` のデフォルト値を使います。

コマンド全体をパラメータとして指定します (すなわち、コマンドラインフラグを含む)。Swift Mailer は「`-bs`」
(デフォルト) と「`-t`」のオペレーションモードをサポートします。

>**NOTE**
>「`-t`」モードの sendmail を実行すると送信の正否にかかわらず、フィードバックは得られません。理由がないかぎり「`-bs`」を使います。

Sendmail Transport を使うには:

 * 正しいコマンドラインフラグを含めて、コマンドで `Swift_SendmailTransport::newInstance()` を呼び出します。指定されていなければ、デフォルトの `/usr/sbin/sendmail -bs` が使われます。

 * Mailer を作るには返されたオブジェクトを使います。

sendmail のプロセスは `send()` もしくは `batchSend()` への最初の呼び出しではじまります。プロセスのスタートが失敗すれば `Swift_Transport_TransportException` が投げられます。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作ります
    $transport = Swift_SendmailTransport::newInstance('/usr/sbin/exim -bs');

    //あらかじめ作っておいた Transport を使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

#### Mail トランスポート

Mail Transport は PHP 内部の `mail()` 関数にデリゲートすることでメッセージを送ります。

筆者や &#8211; ほかのひとの経験によれば &#8211;  `mail()` 関数はとりわけ予想可能でも助けになるわけでもありません。

とてもよく知られていることは、`mail()` 関数のふるまいは Linux と Windows サーバーのあいだで完全に異なります。Linux では `sendmail` が使われますが、Windows では SMTP を使います。

`mail()` 関数をきちんと機能させるには、`sendmail` もしくは `SMTP` サーバーの位置を指定することで、`php.ini` ファイルのコンフィギュレーションを正しく修正する必要があります。

`mail()` の問題はものごとをシンプルに「しよう」としてかえって複雑にしてしまうことです。貧弱なインターフェイスのデザインが原因です。Swift Mailer の開発者は Mail Transport がほどよい程度の一貫性をもって動くように相当な努力をしてきました。

この Transport を使う場合の重大な欠点は:

  * 予測できないメッセージヘッダー

  * デリバリの失敗に関するフィードバックの欠如

  * リアルタムデリバリフィードバックを必須とするいくつかのプラグインの欠如

最後の手段ですが、私たち開発者は情熱をもってこれを言います！

##### Mail トランスポートを使う

Mail Transport を使うには、`Swift_MailTransport::newInstance()` を呼び出す必要があります。Transport のコンフィギュレーションを修正する必要はないでしょう。

Mail Transport を使うには:

 * `Swift_MailTransport::newInstance()` を呼び出します

 * Mailer を作るために返されたオブジェクトを使います

Message の送信には `mail()` 関数が使われます。

>**NOTE**
>`mail()` 関数は `$additional_parameters` パラメータをとります。デフォルトでは Swift Mailer はこのパラメータに「`-f%s`」をセットします。「%s」は (`sprintf()` 関数を通じて) 送信時に送り主のアドレスに置き換わります。`newInstance()` に引数を渡すことでこのデフォルトをオーバーライドすることができます。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作ります
    $transport = Swift_MailTransport::newInstance();

    //あらかじめ作っておいた Transport を使い Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

### メッセージ送信に利用できるメソッド

Mailer クラスはメッセージオブジェクトを送るために２つのメソッドを提供します &#8211;
`send()` と `batchSend()`。それぞれのメソッドのふるまいは微妙に異なります。

Swift Mailer でメッセージが送られるとき、Mailer クラスは選んだ Transport クラスとコミュニケーションをとります。

メッセージのそれぞれの受信者は Transport によって受け入れられるもしくは拒否されます。たとえば、メールアドレスのドメイン名に到達できなければ、アドレスが処理できないので SMTP Transport はアドレスを拒否することがあります。&#8211; `send()` もしくは `batchSend()` &#8211; のどちらを使うのであっても、Swift Mailer は受け入れられた受信者の数を示す整数を返します。

>**NOTE**
>拒否された受信者を見つけることは可能です &#8211; この章の後の方でこの内容をカバーします。

#### `send()` メソッドを使う

`Swift_Mailer` クラスの `send()` メソッドはデスクトップのメールクライアントとまったく同じロジックを使ってメッセージを送ります。Message にこれを渡し結果を得ます。

`send()` メソッドで Message を送るには:

 * 組み込みの Transport クラスの1つから Transport のインスタンスを作ります &#8211;
   `Swift_SmtpTransport`、
   `Swift_SendmailTransport`、
   `Swift_MailTransport` もしくはアグリゲート Transport の1つ

 * Transport をコンストラクタパラメータとして使うことで `Swift_Mailer` クラスのインスタンスを作ります。

 * Message を作ります。

 * オブジェクトの `send()` メソッドを通じてメッセージを送ります。

メールクライアントを使ったようにメッセージは送られます。返される整数は成功した受信者の数を含む整数が返されます。メッセージが送られた受信者がいなければブール値の `false` と同等のゼロが返されます。メッセージのなかの2人の `To:` 受信者と3人の `Bcc:` 受信者をセットすると すべての受信者のデリバリに成功すれば5の値が返されます。

>**NOTE**
>つぎの例において、1つのメールは `receiver@domain.org` と `other@domain.org` の両方に送られます。両方のアドレスとも `To:` ヘッダーのなかで見えます。`receiver@domain.org` と `other@domain.org` のひとたちがお互いを知られないようにしたいのであれば `batchSend()` メソッドを代わりに使います。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作ります
    $transport = Swift_SmtpTransport::newInstance('localhost', 25);

    //あらかじめ作っておいた Transport を使い Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

    //メッセージを作ります
    $message = Swift_Message::newInstance('Wonderful Subject')
      ->setFrom(array('john@doe.com' => 'John Doe'))
      ->setTo(array('receiver@domain.org', 'other@domain.org' => 'A name'))
      ->setBody('Here is the message itself')
      ;
  
    //メッセージを送ります
    $numSent = $mailer->send($message);

    printf("Sent %d messages\n", $numSent);

    /* Note that often that only the boolean equivalent of the
       return value is of concern (ゼロは FALSE を示します)
   
    if ($mailer->send($message))
    {
      echo "Sent\n";
    }
    else
    {
      echo "Failed\n";
    }

    */

#### `batchSend()` メソッドを使う

`Swift_Mailer`
クラスの `batchSend()` メソッドは個別のメッセージを `To:` フィールドのなかのそれぞれの受信者に送ります。それぞれの受信者は `To:` フィールドのなかの独自のアドレスだけを含むメッセージを受けつけます。

`batchSend()` メソッドで Message を送るには:

 * 組み込みの Transport クラスの1つから Transport を作ります &#8211;
   `Swift_SmtpTransport`、
   `Swift_SendmailTransport`、
   `Swift_MailTransport` もしくはアグリゲート Transport の1つ

 * Transport をコンストラクタパラメータとして使い `Swift_Mailer` クラスのインスタンスを作ります。

 * Message を作ります。

 * Mailer オブジェクトの `batchSend()` メソッドを通じてメッセージを送ります。

メッセージのそれぞれの受信者は `To:` フィールドの自分自身のメールアドレスだけがついてくる異なるコピーを受けとります。成功した受信者の人数と同じ整数が返されます。受信者に送信されなければブール値の `false` と同等のゼロが返されます。メッセージのなかの2人の `To:` 受信者と3人の `Bcc:` 受信者とすべての受信者のデリバリが成功すれば5の値が返されます。

>**NOTE**
>次の例では、2つのメールが送信されます。`receiver@domain.org` と `other@domain.org` のそれぞれに1回ずつです。これらの受信者がお互いを知ることはありません。

    [php]
    require_once 'lib/swift_required.php';

    //Transport を作ります
    $transport = Swift_SmtpTransport::newInstance('localhost', 25);

    //あらかじめ作っておいた Transport を使って Mailer を作ります
    $mailer = Swift_Mailer::newInstance($transport);

    //メッセージを作ります
    $message = Swift_Message::newInstance('Wonderful Subject')
      ->setFrom(array('john@doe.com' => 'John Doe'))
      ->setTo(array('receiver@domain.org', 'other@domain.org' => 'A name'))
      ->setBody('Here is the message itself')
      ;

    //メッセージを送ります
    $numSent = $mailer->batchSend($message);

    printf("Sent %d messages\n", $numSent);

    /* Note that often that only the boolean equivalent of the
       return value is of concern (ゼロは FALSE を示します)

    if ($mailer->batchSend($message))
    {
      echo "Sent\n";
    }
    else
    {
      echo "Failed\n";
    }

    */

### 拒否されたアドレスを見つける

`send()` もしくは
`batchSend()` メソッドへの参照によるパラメータを使うことで、Transport によって拒否されたアドレスのリストを得ることは可能です。

Swift Mailer はそれぞれのアドレスにメッセージを送ることを試みますが、受信者が拒否されれば、配列に追加されます。既存の配列を渡すことが可能で、さもなければ配列が参照によって作られます。

拒否された受信者のリストを集めることはたとえばデリバリがうまくゆかないときに「prune」なメーリングリストが必要な状況などに役立ちます。 

#### 参照で失敗を取得する

`send()` もしくは
`batchSend()` メソッドで参照によるデリバリの失敗を集める方法はシンプルで、メソッド呼び出しに変数の名前を渡すだけです。

参照で送信できなかった受信者を得るには:

 * Mailer クラスの `send()` もしくは
   `batchSend()` メソッドに変数の名前を参照で渡しします。

Transport が受信者の誰かを拒否する場合、犯人のアドレスが参照渡しで提供された配列に犯人のアドレスが追加されます。

>**NOTE**
>変数の名前がまだ存在していなければ、これは空の配列として初期化され、その配列に失敗の情報が追加されます。変数がすでに存在していれば、配列にタイプキャスティングされ、それに失敗の情報が追加されます。

    [php]
    $mailer = Swift_Mailer::newInstance( ... );

    $message = Swift_Message::newInstance( ... )
      ->setFrom( ... )
      ->setTo(array(
        'receiver@bad-domain.org' => 'Receiver Name',
        'other@domain.org' => 'A name',
        'other-receiver@bad-domain.org' => 'Other Name'
      ))
      ->setBody( ... )
      ;
  
    //send() メソッドに変数の名前を渡します
    if (!$mailer->send($message, $failures))
    {
      echo "Failures:";
      print_r($failures);
    }

    /*
    Failures:
    Array (
      0 => receiver@bad-domain.org,
      1 => other-receiver@bad-domain.org
    )
    */
