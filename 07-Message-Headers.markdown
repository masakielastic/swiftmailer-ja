メッセージのヘッダー
=====================

メッセージに独自のヘッダーを追加したいもしくはヘッダーを修正/削除したいことがあります。これを実現するにはメッセージの HeaderSet を取り組みます。

ヘッダーの基本
--------------

Swift Mailer &#8211; において、メッセージ自身 &#8211; を含むすべての MIME エンティティ。これらのヘッダーを HeaderSet と呼ばれる単独のオブジェクトに保存します。HeaderSet は `getHeaders()` メソッドで読み取られます。

以前の章で述べたように、Swift Mailer においてメッセージのパーツを形成するものはすべて `Swift_Mime_MimeEntity` のインスタンスで表される MIME エンティティです。これに含まれるのは &#8211; もっとも知られている &#8211; 
メッセージオブジェクトそのもの、添付物、MIME パートと埋め込み画像です。これらの MIME エンティティはそれぞれボディとボディを記述するヘッダーのセットから校正されます。

`Content-Type` など
これらの MIME エンティティの「標準的な」すべてのヘッダーに関して、`setContentType()` や
`getContentType()` などの、これらと取り組むための名前がつけられているメソッドが用意されています。これはヘッダーはライブラリの適度に複雑な領域だからです。それぞれのヘッダーには (とスパムブロッカーによってチェックされるなどの) メールを管理する標準に応じるために満たさなければならない、微妙に異なる必須の構造があります。

次のような MIME エンティティから HeaderSet をフェッチできます:

    [php]
    $message = Swift_Message::newInstance();

    //Message オブジェクトから HeaderSet をフェッチします
    $headers = $message->getHeaders();

    $attachment = Swift_Attachment::fromPath('document.pdf');

    //Attachment オブジェクトから HeaderSet をフェッチします
    $headers = $attachment->getHeaders();

HeaderSet の仕事は Header オブジェクトのインスタンスを保持し管理することです。HeaderSet が由来する MIME エンティティによって HeaderSet の内容は異なります。たとえば、添加物にメッセージにおいてこれらと異なるヘッダーのセットが含まれるからです。

ヘッダーの名前を吐き出すクイックループで HeaderSet に何がを含まれているのかわかります:

    [php]
    foreach ($headers->getAll() as $header) {
      printf("%s<br />\n", $header->getFieldName());
    }

    /*
    Content-Transfer-Encoding
    Content-Type
    MIME-Version
    Date
    Message-ID
    From
    Subject
    To
    */

`toString()` メソッドを呼び出すことでレンダリングされた HeaderSet を吐き出すこともできます:

    [php]
    echo $headers->toString();

    /*
    Message-ID: <1234869991.499a9ee7f1d5e@swift.generated>
    Date: Tue, 17 Feb 2009 22:26:31 +1100
    Subject: Awesome subject!
    From: sender@example.org
    To: recipient@example.org
    MIME-Version: 1.0
    Content-Type: text/plain; charset=utf-8
    Content-Transfer-Encoding: quoted-printable
    */

複雑性がやってくるのは既存のへダーを修正したいときです。この複雑性はそれぞれのヘッダーが微妙に異なるタイプをもつことが可能であるという事実からやってきます (Date ヘッダー、もしくはメールアドレスを含むヘッダー、もしくは &#8230; の上のキーバリューパラメータをもつヘッダーなど)。HeaderSet のそれぞれのヘッダーは `Swift_Mime_Header` のインスタンスです。
これらすべてには共通の機能がありますが、取り組んでいるヘッダーが種類が何であるのか正確に知っていれば、もう少しコントールできるようになります。

`Swift_Mime_Header` で定義されている定数である `TYPE_TEXT`, `TYPE_PARAMETERIZED`, `TYPE_DATE`, `TYPE_MAILBOX`,
`TYPE_ID` と `TYPE_PATH` と  `getFieldType()` メソッドの値を比較することでヘッダーの種類を決定することができます。


    [php]
    foreach ($headers->getAll() as $header) {
      switch ($header->getFieldType()) {
        case Swift_Mime_Header::TYPE_TEXT: $type = 'text';
          break;
        case Swift_Mime_Header::TYPE_PARAMETERIZED: $type = 'parameterized';
          break;
        case Swift_Mime_Header::TYPE_MAILBOX: $type = 'mailbox';
          break;
        case Swift_Mime_Header::TYPE_DATE: $type = 'date';
          break;
        case Swift_Mime_Header::TYPE_ID: $type = 'ID';
          break;
        case Swift_Mime_Header::TYPE_PATH: $type = 'path';
          break;
      }
      printf("%s: is a %s header<br />\n", $header->getFieldName(), $type);
    }

    /*
    Content-Transfer-Encoding: is a text header
    Content-Type: is a parameterized header
    MIME-Version: is a text header
    Date: is a date header
    Message-ID: is a ID header
    From: is a mailbox header
    Subject: is a text header
    To: is a mailbox header
    */

ヘッダーはセットから削除したり、セットの範囲で修正したり、もしくはセットに追加することができます。

次の節では HeaderSet を扱う方法を示し、HeaderSet の範囲に存在する `Swift_Mime_Header` のそれぞれの実装を説明します。

ヘッダーの種類
---------------

すべてのヘッダーは異なるデータ (データ、アドレス、テキスト&#8230;) でモデリングされているので、Swift Mailer には異なる種類の Header が存在します。Swift Mailer はわずかな数のクラスを定義することで 利用可能な MIME ヘッダーをより一般的なグループにまとめようとします。

### テキストのヘッダー

Text ヘッダーはもっともシンプルな Header です。これらはたとえばメッセージの Subject ヘッダーなど、それ &#8211 の範囲に含まれる特別ではない情報を含めてテキスト情報を含みます。

Text ヘッダーに関して特筆することはありませんが、メッセージにカスタムヘッダーを追加する必要があれば、おそらくこれがもっとも使いたいものでしょう。これはご想像どおりテキストをあらわします。テキストにメッセージヘッダーで許可されない文字が含まれる場合 (改行もしくは ASCII ではない文字) ヘッダーはテキストのエンコーディングを配慮します。

Swift Mailer において Text ヘッダー &#8211; を含めて ヘッダーではないものはヘッダーインジェクション攻撃に対して脆弱性を持ちます。Swift Mailer は危険なデータを危険ではない形式にエンコーディングすることでヘッダーインジェクションへの試みを阻止します。

HeaderSet に新しい Text ヘッダーを追加することはかんたんです。HeaderSet の `addTextHeader()` メソッドを呼び出すことでこれを実現できます。

    [php]
    $message = Swift_Message::newInstance();

    $headers = $message->getHeaders();

    $headers->addTextHeader('Your-Header-Name', 'the header value');

`setValue()` メソッドを呼び出すことで、既存の Text ヘッダーの値を変更することができます。

    [php]
    $subject = $message->getHeaders()->getHeader('Subject');

    $subject->setValue('new subject');

`toString()` メソッドを出力するとき、Text ヘッダーは次のような内容を生み出します:

    [php]
    $subject = $message->getHeaders()->getHeader('Subject');

    $subject->setValue('amazing subject line');

    echo $subject->toString();

    /*

    Subject: amazing subject line

    */

ヘッダーに US-ASCII の外側の範囲にある文字が含まれる場合、これらはエンコードされます。メールクライアントがこれらをデコードして戻すので、このことを気にする必要はありません。

    [php]
    $subject = $message->getHeaders()->getHeader('Subject');

    $subject->setValue('contains – dash');

    echo $subject->toString();

    /*

    Subject: contains =?utf-8?Q?=E2=80=93?= dash

    */

### パラメータ化されたヘッダー

パラメータ化されたヘッダーはテキストのコンテキストをフォローするキーバリューのパラメータを含む Text ヘッダーです。メッセージの Content-Type ヘッダーはパラメータ化されたヘッダーです。Content-Type のあとで文字集合の情報を含むからです。

パラメータ化されたヘッダー Text ヘッダーの特殊な種類のヘッダーです。これは Text ヘッダーをフォローする追加情報を許可することで Text ヘッダーを拡張します。ここで説明したメソッドに加えて Text ヘッダーからのすべてのメソッドが利用可能です。

HeaderSet にパラメータ化されたヘッダーを追加することは `addParameterizedHeader()` メソッドによって実現できます。このメソッドは
`addTextHeader()` のようにテキストの値をとりますが、キーバリューパラメータの連想配列、も受け入れます。

    [php]
    $message = Swift_Message::newInstance();

    $headers = $message->getHeaders();

    $headers->addParameterizedHeader(
      'Header-Name', 'header value',
      array('foo' => 'bar')
      );

ヘッダーのテキストの値を変更するには、Text ヘッダーのように `setValue()`
メソッドを呼び出します。

ヘッダーのパラメータを変更するには、ヘッダーの `setParameters()` もしくは `setParameter()`
メソッドを呼び出します (複数形にご注目)。

    [php]
    $type = $message->getHeaders()->getHeader('Content-Type');

    //setParameters() メソッドは連想配列をとります
    $type->setParameters(array(
      'name' => 'file.txt',
      'charset' => 'iso-8859-1'
      ));

    //setParameter() メソッドは $key と $value の2つの引数をとります
    $type->setParameter('charset', 'iso-8859-1');

`toString()` メソッドを通じて出力するとき、パラメータ化されたヘッダーは次の内容を出力します:

    [php]
    $type = $message->getHeaders()->getHeader('Content-Type');

    $type->setValue('text/html');
    $type->setParameter('charset', 'utf-8');

    echo $type->toString();

    /*

    Content-Type: text/html; charset=utf-8

    */

ヘッダーに US-ASCII の範囲の外側にある任意の文字が含まれる場合、Text ヘッダーのように、これらはエンコードされます。メールクライアントがこれらをデコードして戻すのでこのことを気にする必要はありません。同じように、パラメータが ASCII ではない文字を含む場合、これらの文字は安全に送り届けられるようにエンコードされます。

    [php]
    $attachment = Swift_Attachment::newInstance();

    $disp = $attachment->getHeaders()->getHeader('Content-Disposition');

    $disp->setValue('attachment');
    $disp->setParameter('filename', 'report–may.pdf');

    echo $disp->toString();

    /*

    Content-Disposition: attachment; filename*=utf-8''report%E2%80%93may.pdf

    */

### 日付ヘッダー

Date ヘッダーは日付のフォーマットをする RFC 2822 を含みます (すなわち PHP の `date('r')` の戻り値)。これらは日付や時間をメッセージヘッダーとして表すことが必要な場所で使うことができます。

Date ヘッダーによってモデリングされているデータは `time()` もしくは `strtotime()` によって返される UNIX タイムスタンプです。タイムスタンプは `Tue, 17 Feb 2009 22:26:31 +1100` などのRFC 2822 の構造化されフォーマットされた正しい日付を作るために使われます。

このヘッダー型が使われる明らかな場所はメッセージ自身の `Date:` ヘッダーです。

HeaderSet に新しい Date ヘッダーを追加することはかんたんです。HeaderSet の `addDateHeader()` メソッドを呼び出すことで実現できます。

    [php]
    $message = Swift_Message::newInstance();

    $headers = $message->getHeaders();

    $headers->addDateHeader('Your-Header-Name', strtotime('3 days ago'));

Date ヘッダーの既存の値を変更するには `setTimestamp()` メソッドを呼び出します。

    [php]
    $date = $message->getHeaders()->getHeader('Date');

    $date->setTimestamp(time());

`toString()` メソッドを通じて出力するとき、 Date ヘッダーは次のような内容を生み出します:

    [php]
    $date = $message->getHeaders()->getHeader('Date');

    echo $date->toString();

    /*

    Date: Wed, 18 Feb 2009 13:35:02 +1100

    */

### メールボックスヘッダー (電子メールアドレス)

Mailbox ヘッダーは1つもしくは複数のヘッダーを含みます。おそらくは個人化された名前がこれらに添付されます。これらがモデリングされるデータはメールアドレスと名前の連想配列によって表されます。

Swift Mailer において Mailbox ヘッダーはおそらくもっとも複雑なヘッダーのタイプです。以前の章で説明したように、これらがさまざまな形式をとるのことできる入力を配列として受け入れるからです。

より厳密な構文 &#8211; を持つ `Return-Path` を除いて、メッセージのなかでメールアドレスを含むすべてのへダーは &#8211; このヘッダータイプを使います。すなわち `To:`、 `From:` などです。

HeaderSet の `addMailboxHeader()` メソッドを呼び出すことで HeaderSet に新しい Mailbox ヘッダーを追加します。

    [php]
    $message = Swift_Message::newInstance();

    $headers = $message->getHeaders();

    $headers->addMailboxHeader('Your-Header-Name', array(
      'person1@example.org' => 'Person Name One',
      'person2@example.org',
      'person3@example.org',
      'person4@example.org' => 'Another named person'
      ));

Mailbox ヘッダーの既存の値を変更するには `setNameAddresses()` メソッドを呼び出すことで実現できます。

    [php]
    $to = $message->getHeaders()->getHeader('To');

    $to->setNameAddresses(array(
      'joe@example.org' => 'Joe Bloggs',
      'john@example.org' => 'John Doe',
      'no-name@example.org'
      ));

以前の章で説明したように `setNameAddresses()` メソッドとして受け入れられる複雑な入力フォーマットを自分で理解したいとは思わず (名前ではなく 1つもしくは複数のアドレスだけをセットしたいのであれば、`setAddresses()` メソッドを代わりに使うことができます。

    [php]
    $to = $message->getHeaders()->getHeader('To');

    $to->setAddresses(array(
      'joe@example.org',
      'john@example.org',
      'no-name@example.org'
      ));

>**NOTE**
>両方のメソッドとも実際の入力フォーマットを受け入れます。

ヘッダーのなかで単独のアドレスだけが必要であれば、文字列を `setAddresses()` かつ/もしくは
`setNameAddresses()` メソッドへの入力パラメータとして使うことができます。

    [php]
    $to = $message->getHeaders()->getHeader('To');

    $to->setAddresses('joe-bloggs@example.org');

`toString()` メソッドを通じて出力するとき、Mailbox ヘッダーは次のような内容を生み出します:

    [php]
    $to = $message->getHeaders()->getHeader('To');

    $to->setNameAddresses(array(
      'person1@example.org' => 'Name of Person',
      'person2@example.org',
      'person3@example.org' => 'Another Person'
    ));

    echo $to->toString();

    /*

    To: 個人の名前 <person1@example.org>, person2@example.org, 別の個人
     <person3@example.org>

    */

### ID ヘッダー

ID ヘッダーはエンティティの識別子 (もしくはメッセージ) を含みます。もっとも著名な ID ヘッダーはメッセージ自身の  Message-ID ヘッダーです。

ID ヘッダー内部に存在する ID はいくぶんかメールアドレスのように見えます。たとえば、 `<![CDATA[<1234955437.499becad62ec2@example.org>]]>`。@ 記号の左側の部分は通常はユニークで、現在の時刻とランダムファクタにもとづいています。右側の部分は通常はドメインの名前です。

ID ヘッダーの `setId()` メソッドはこの構造にしたがわなければなりません。さもなければ Swift Mailer (`Swift_RfcComplianceException`) によって例外を得ることになります。このことによって、生成されたメールは適切な RFC ドキュメントでコンパイルされスパムとしてブロックされにくくなることが保証されます。

HeaderSet に新しい ID ヘッダーを追加するのはかんたんです。HeaderSet の `addIdHeader()` メソッドを呼び出すことでこれを実現できます。

    [php]
    $message = Swift_Message::newInstance();

    $headers = $message->getHeaders();

    $headers->addIdHeader('Your-Header-Name', '123456.unqiue@example.org');

既存の日付ヘッダーを変更するには `setId()` メソッドを呼び出します。

    [php]
    $msgId = $message->getHeaders()->getHeader('Message-ID');

    $msgId->setId(time() . '.' . uniqid('thing') . '@example.org');

`toString()` を通じて出力するとき、ID ヘッダーは次の内容を生み出します:

    [php]
    $msgId = $message->getHeaders()->getHeader('Message-ID');

    echo $msgId->toString();

    /*

    Message-ID: <1234955437.499becad62ec2@example.org>

    */

### パスヘッダー

Path ヘッダーはとても制限された Mailbox ヘッダーのようです。これらは関連していない名前で単独のメールアドレスを含みます。メッセージの `Return-Path` ヘッダーは Path ヘッダーです。

HeaderSet の
`addPathHeader()` メソッドを呼び出すことで HeaderSet に新しい Path ヘッダーを追加します。

    [php]
    $message = Swift_Message::newInstance();

    $headers = $message->getHeaders();

    $headers->addPathHeader('Your-Header-Name', 'person@example.org');


`setAddress()` メソッドを呼び出すことで Path ヘッダーの既存の値を変更できます。

    [php]
    $return = $message->getHeaders()->getHeader('Return-Path');

    $return->setAddress('my-address@example.org');

`toString()` メソッドを通じて出力するとき、Path ヘッダーは次のような内容を出力します:

    [php]
    $return = $message->getHeaders()->getHeader('Return-Path');

    $return->setAddress('person@example.org');

    echo $return->toString();

    /*

    Return-Path: <person@example.org>

    */

ヘッダーのオペレーション
------------------------

メッセージの中でヘッダーを扱うことは HeaderSet HeaderSet の範囲での個別のヘッダーでのメソッドの使い方を知ることを含みます。

### 新しいヘッダーを追加する

`add..Header()` メソッドによって提供されるメソッドを使うことで HeaderSet に新しいヘッダーを追加できます。

MIME エンティティにヘッダーを追加するには (メッセージなど):

 * `getHeaders()` メソッドを通じてエンティティから HeadSet を取得します。

 * `add..Header()` メソッド群の1つを呼び出して HeadSet にヘッダーを追加します。

追加されたヘッダーがメッセージが送信されるときにあらわれます。

    [php]
    //メッセージにカスタムヘッダーを追加します
    $message = Swift_Message::newInstance();
    $headers = $message->getHeaders();
    $headers->addTextHeader('X-Mine', 'something here');

    //添付物にカスタムヘッダーを追加します
    $attachment = Swift_Attachment::fromPath('/path/to/doc.pdf');
    $attachment->getHeaders()->addDateHeader('X-Created-Time', time());

### ヘッダーを読み出す

ヘッダーは HeaderSet の `get()` と
`getAll()` メソッドを通じて読み取られます。

MIME エンティティから１つもしくは複数のヘッダーを得るには:

 * `getHeaders()` メソッドを通じてエンティティから HeaderSet を得ます。

 * `get()` もしくは `getAll()` メソッドのどちらを呼び出して HeaderSet からヘッダーを得ます。

`get()` メソッドを使うとき、単独のヘッダーはそれに渡された名前(大文字小文字を区別しない)にマッチする値を返します。ヘッダーの名前で `getAll()` メソッドを呼び出すとき、その名前と一緒にヘッダーの配列が返されます。引数なしで `getAll()` メソッドを呼び出すとエンティティに存在するすべてのヘッダーの配列を返します。

>**NOTE**
>これはメッセージのなかで複数回表れるヘッダーに妥当です (たとえば Received ヘッダー)。このために指定した名前ですべてのヘッダーをフェッチする `getAll()` メソッドが用意されています。加えて、`get()` メソッド ゼロからより明確に指定したいヘッダーではじまるオプションの数値インデックスで `get()` メソッドが用意されています。

>**NOTE**
>ヘッダーの内容を修正したいのであれば、ヘッダーの種類が何であるのか細かい内容を知る必要はありません。`getFieldType()` メソッドを呼び出すことでヘッダーの種類をチェックする必要があるかもしれません。

    [php]
    $headers = $message->getHeaders();

    //To: ヘッダーを取得します
    $toHeader = $headers->get('To');

    //「X-Foo」という名前のヘッダーをすべて取得します
    $fooHeaders = $headers->getAll('X-Foo');

    //「X-Foo」という名前の2番目のヘッダーを取得します
    $foo = $headers->get('X-Foo', 1);

    //存在するすべてのヘッダーを取得します
    $all = $headers->getAll();

### ヘッダーが存在するかチェックする

`has()` メソッドを呼び出すことで名前つきヘッダーが HeaderSet に存在するかチェックできます。

ヘッダーが存在するかチェックするには:

 * `getHeaders()` メソッドを通じてエンティティから HeaderSet を得ます。

 * お探しのヘッダーを指定する HeaderSet の `has()` メソッドを呼び出します。
 
ヘッダーが存在していれば、`true` が返され、存在していなければ `false` が返されます。

>**NOTE**
>メッセージのなかで複数回ヘッダーが表れるのは妥当です (たとえば Received ヘッダー)。このために `has()` はゼロからより細かくチェックしたいヘッダーを指定するゼロから始まるオプションの数値インデックスを受け入れます。

    [php]
    $headers = $message->getHeaders();

    //To: ヘッダーが存在しているかチェックします
    if ($headers->has('To')) {
      echo 'To: exists';
    }

    //X-Foo ヘッダーが重複しているかチェックします (すなわち、2番目をチェックします)
    if ($headers->has('X-Foo', 1)) {
      echo 'Second X-Foo header exists';
    }

### ヘッダーを削除する

HeaderSet から Header を削除するには HeaderSet の
`remove()` もしくは `removeAll()` メソッドを呼び出すことで実行します。

既存のヘッダーを削除するには:

 * `getHeaders()` メソッドを通じてエンティティから HeaderSet を得ます

 * 削除したいヘッダーを指定する `remove()` もしくは
   `removeAll()` メソッドを呼び出します

`remove()` メソッドを呼び出すとき、単独のヘッダーが削除されます。`removeAll()` メソッドを呼び出すとき、任意の名前ですべてのヘッダーが削除されます。渡された名前のヘッダーが存在していなければ、エラーになります。

>**NOTE**
>１つのメッセージのなかでヘッダーが複数回表れるのは妥当です (たとえば `Received` ヘッダー)。この理由のために  `remove()` メソッドはオプションの数値インデックスを受けつけ、より細かくチェックしたいヘッダーを指定できます。この理由のために任意の名前をもつすべてのヘッダーを削除する `removeAll()` が用意されています。

    [php]
    $headers = $message->getHeaders();

    //Subject: ヘッダーを削除します
    $headers->remove('Subject');

    //すべての X-Foo ヘッダーを削除します
    $headers->removeAll('X-Foo');

    //2番目の X-Foo ヘッダーだけを削除します
    $headers->remove('X-Foo', 1);

### ヘッダーのコンテンツを修正する

Header の内容を変更するには ヘッダーの種類がどんなものであるのか知る必要があり、適切なセッターメソッドを呼び出す必要があります。すべてのヘッダーには 混在型のパラメータと正しいセッターへのデリゲートを受けつける `setFieldBodyModel()` メソッドも用意されています。

既存のヘッダーを修正するには:

 * `getHeaders()` メソッドを通じてエンティティから headerSet を得ます。

 * HeaderSet の `get()` メソッドを使うことで Header を得ます。

 * Header の適切なセッターメソッドを呼び出すもしくはヘッダーの `setFieldBodyModel()` メソッドを呼び出します。

HeaderSet 内部でヘッダーがアップデートされメッセージが送信されるときに変更が見えるようになります。

    [php]
    $headers = $message->getHeaders();

    //Subject: ヘッダーを変更します
    $subj = $headers->get('Subject');
    $subj->setValue('new subject here');

    //To: ヘッダーを変更します
    $to = $headers->get('To');
    $to->setNameAddresses(array(
      'person@example.org' => 'Person',
      'thing@example.org'
    ));

    //setFieldBodyModel() を使えば正しいメソッドにデリゲートされ
    // ここではsetNameAddresses() が呼び出されます。
    $to->setFieldBodyModel(array(
      'person@example.org' => 'Person',
      'thing@example.org'
    ));
