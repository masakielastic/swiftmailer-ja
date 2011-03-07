メッセージを作成する
====================

Swift Mailer でメッセージを作ることはライブラリによって提供されるさまざまな MIME エンティティを利用することで実現できます。ほんの少しの労力で複雑なメッセージをすばやく作れます。

メッセージ作成のクィックリファレンス
------------------------------------

メッセージ作成はメールクライアントで「作成」ボタンをクリックするときに実行する手順と同じように考えることができます
。件名を入力し、受信者を指定し、添付ファイルを追加してメッセージを書きます。メッセージを作るにはつぎの手順を踏みます。

 * `Swift_Message` オブジェクトの `newInstance()` メソッドを呼び出します。

 * `setFrom()` もしくは `setSender()` メソッドで送信アドレス (`From:`) をセットします。

 * `setSubject()` メソッドで件名をセットします。

 * `setTo()`、`setCc()` かつ/もしくは `setBcc()` メソッドで受信者をセットします。

 * `setBody()` メソッドでボディをセットします。

 * `attach()` メソッドで添付ファイルを追加します。

    [php]
    require_once 'lib/swift_required.php';

    //メッセージオブジェクトを作ります
    $message = Swift_Message::newInstance()

      //メッセージの件名をセットします。
      ->setSubject('Your subject')

      //送信者のアドレスを連想配列の形式でセットします。
      ->setFrom(array('john@doe.com' => 'John Doe'))

      //受信者のアドレスを連想配列の形式でセットします
      ->setTo(array('receiver@domain.org', 'other@domain.org' => 'A name'))

      //本文を提供します
      ->setBody('Here is the message itself')

      //オプションとして代替ボディを追加します
      ->addPart('<q>Here is the message itself</q>', 'text/html')

      //オプションとして任意の添付ファイルを追加します
      ->attach(Swift_Attachment::fromPath('my-document.pdf'))
      ;

メッセージの基本
-----------------

メッセージはだれかに送りたいものを入れるコンテナです。知っておくべきメッセージの基本的な側面がいくつかあります。

メールのメッセージは異なる結果を得るために異なる方法で結びつけられるいくつかの相対的にシンプルなエンティティから構成されます。これらのエンティティすべてには同じ基本的なアウトラインがありますが、異なる目的を提供します。メッセージ自身は MIME エンティティとして定義できます。添付物は MIME エンティティです。すべての
MIME パートは MIME エンティティの &#8211; や &#8230; などです。

それぞれの MIME エンティティ &#8211; の基本ユニットはメッセージ自身で、添付物 &#8211; はヘッダーとボディです:

    Header-Name: A header value
    Other-Header: Another value

    The body content itself

MIME エンティティのヘッダーとボディはさまざまな RFC ドキュメントによって定義される厳密な標準にしたがわなければなりません。エンティティを生成するためのエンコーダーと異なるヘッダーの種類を含めて、これらの仕様がさまざまな種類のオブジェクトによって使われることでこれらの仕様がフォローされることを Swift Mailer は保証します。

### メッセージの構造

すべての MIME エンティティのなかで、メッセージを扱う &#8211; `Swift_Message`
はもっとも大きくてもっとも複雑です。更新可能な多くのプロパティが含まれ、ほかの MIME エンティティ &#8211; たとえば &#8211; などの添付物を含むことができます。その内部でネストにできます。

メッセージにはたくさんの異なるヘッダーが含まれます。ヘッダーにはメールクライアントへのメッセージに関する情報があります。これらのヘッダーの大半はユーザーのマジョリティに慣れ親しまれていますが、基本的なものだけを並べます。メッセージのヘッダー (もしくはほかの MIME エンティティ) と直接連携させることが可能ですが、複雑な詳細内容を引き出すために標準的なヘッダーにはアクセサメソッドが用意されています。たとえば、メッセージの日付は厳密なフォーマットで書かれていますが、UNIX のタイムスタンプを `setDate()` メソッドに渡す必要があるだけです。

| ヘッダー | 説明 | アクセサ |
| ------ | ----------- | --------- |
| `Message-ID` | ユニーク ID でこのメッセージを見わけます。 通常はドメインと生成された時刻を含みます。| `getId()` / `setId()` |
| `Return-Path` | バウンスの行き先を指定します (Swift Mailer はほかの用途のためにこれを読み込みます) | `getReturnPath()` / `setReturnPath()` |
| `From` | メッセージの送り主のアドレスを指定します。複数のひとがメッセージを書いたのであれば複数のアドレスが指定可能です。| `getFrom()` / `setFrom()` |
| `Sender` |物理的にメッセージを送信した人物のアドレスを指定します (`From:` よりも優先順位が高いです) | `getSender()` / `setSender()` |
| `To` | 予定された受信者のアドレスを指定します | `getTo()` / `setTo()` |
| `Cc` | メッセージにコピーされる受信者のアドレスを指定します | `getCc()` / `setCc()` |
| `Bcc` | ブラインドコピーされるメッセージを送る受信者のアドレスを指定します。これらのコピーがほかの受信者の目に入ることはありません。 | `getBcc()` / `setBcc()` |
| `Reply-To` | 返信が送信されるアドレスを指定します | `getReplyTo()` / `setReplyTo()` |
| `Subject` | 受信者のメールクライアントに表示される件名の行を指定します | `getSubject()` / `setSubject()` |
| `Date` | メッセージが送信された日付を指定します | `getDate()` / `setDate()` |
| `Content-Type` | メッセージのフォーマットを指定します (通常は text/plain または text/html) | `getContentType()` / `setContentType()` |
| `Content-Transfer-Encoding` | メッセージのエンコーディングスキーマを指定します | `getEncoder()` / `setEncoder()` |

### メッセージオブジェクトに取り組む

メッセージオブジェクトで利用可能なたくさんのメソッドが用意されていますが、これらの小さなサブセットを使う必要があるだけです。通常は`setBody()` メソッドでメッセージのボディをセットする前に `setSubject()`、`setTo()` と
`setFrom()` メソッドを使います。


メソッドを呼び出す方法はシンプルです。関数のように呼び出すだけです。しかし、オブジェクト演算子の "`<![CDATA[->]]>`" を使うことです。メッセージオブジェクトを作り、それを `$message` とすれば、次のように件名をセットしたことになります:

    [php]
    require_once 'lib/swift_required.php';  

    $message = Swift_Message::newInstance();
    $message->setSubject('My subject');

すべての MIME エンティティ (メッセージを含む) には何が送られているのかみたい場合に呼び出すことのできるhave a `toString()`
メソッドが用意されています。たとえば、`<![CDATA[echo
$message->toString();]]>` を実行すれば、つぎのような内容を見ることになります:


    Message-ID: <1230173678.4952f5eeb1432@swift.generated>
    Date: Thu, 25 Dec 2008 13:54:38 +1100
    Subject: Example subject
    From: Chris Corbyn <chris@w3style.co.uk>
    To: Receiver Name <recipient@example.org>
    MIME-Version: 1.0
    Content-Type: text/plain; charset=utf-8
    Content-Transfer-Encoding: quoted-printable

    Here is the message

次の節でメッセージを作るために使うメソッドをよく見てみることにします。

メッセージにコンテンツを加える
------------------------------

Swift Mailer においてリッチコンテンツは `setSubject()`、`setBody()`、`addPart()` や `attach()` などのメソッドを呼び出すことで比較的かんたんにメソッドに追加できます。

### 件名の行をセットする

受信者のメールクライアントに表示される件名の行は `setSubject()` メソッドでセットできます。もしくは `Swift_Message::newInstance()` のパラメータとしてセットできます。

メッセージの件名をセットするには:

 * メッセージの `setSubject()` メソッドを呼び出すもしくはメッセージを作るたびに指定します

    // メッセージを作るときにパラメータとしてセットします
    $message = Swift_Message::newInstance('My amazing subject');

    // もしくはつぎのようにセットします
    $message->setSubject('My amazing subject');

### ボディのコンテンツを設定する

ユーザーがメッセージを開くときメッセージのボディは `setBody()` メソッドを呼び出すことで指定されます。代替ボディは `addPart()` メソッドで含めることができます。

メッセージのボディはユーザーによって読まれるメインパートです。HTML フォーマット (`text/html`) やプレーンテキスト (`text/plain`) でメッセージを送ったり、もしくは両方のバージョンで送り受信者がメッセージを見る方法を選んでもらえるようにしたことがあります。

経験則として、HTML メールを送ろうとする場合、プレーンテキストを好むユーザーが読めるように同じ内容のプレーンテキストをつねに含めておきます。

メッセージのボディをセットするには:

 * メッセージの `setBody()` メソッドを呼び出すか、メッセージを作るたびに指定します

 * `addPart()` メソッドで代替ボディを追加します

受信者のメールクライアントがテキスト vs. HTML を表示するためのプリファレンスを提供する場合、メールクライアントはユーザーが利用可能なパートを表します。ほかのケースではメールクライアントは「ベストな」パートを表示します - HTML を含めていれば通常は HTML になります。

    [php]
    //メッセージを作るときにパラメータとして渡します
    $message = Swift_Message::newInstance('Subject here', 'My amazing body');  

    //もしくはつぎのように後でセットします
    $message->setBody('My <em>amazing</em> body', 'text/html');

    //addPart() メソッドで代替パートを追加します
    $message->addPart('My amazing body in plain text', 'text/plain');

ファイルを添付する
------------------

添付ファイルはメッセージのダウンロード可能な部分で、メッセージの `attach()` メソッドを呼び出すことで追加できます。ディスクに存在しているファイルを追加することができます。もしくは添付ファイルを即座に作ることもできます。

添付ファイルは興味深い問題領域でメッセージがまとまっている方法の背景にあるコンセプトを理解していれば、Swift Mailer から多くの力を得られます。

メールを通じて送られるファイルを "添付物" と言い表してきました &#8211; これらはメッセージに添付されるからです &#8211; メッセージのほかの多くのパートは、これらを添付物として言い表さなくても、実際には「添付されます」。

ファイルの添付は `Swift_Attachment` クラスによって作られ `attach()` メソッドを通じてメッセージに添付されます。画像フォーマット、Word ドキュメントや PDF とスプレッドシートなど "毎日使う" MIME-types に関して、添付ファイルの Content-Type を明示的にセットする必要はありません。セットされていなくても害はありません。より共通性の少ないフォーマットに関しては、 Content-Type をセットすべきです　&#8211; それらのフォーマットはそのうちカバーされるようになります。

### 既存のファイルを添付する

ディスクもしくは URL にすでに存在しているファイルは `Swift_Attachment::fromPath()` を使って１行でメッセージに添付できます。

ローカルに存在するファイルを添付することができます。`allow_url_fopen` ディレクティブが有効であれば、ほかのウェブサイトからファイルを添付することができます。

既存のファイルを添付するには:

 * `Swift_Attachment::fromPath()` で添付ファイルを作ります

 * `attach()` メソッドでメッセージに添付します。

添付ファイルは添付時と同じ名前でダウンロード可能なファイルとして受信者に示されます。

    [php]
    //添付ファイルを作ります
    // * 技術的には Content-Type パラメータを省くことができることにご注意ください
    $attachment = Swift_Attachment::fromPath('/path/to/image.jpg', 'image/jpeg');  

    //メッセージに添付します
    $message->attach($attachment);


    //上記の2つのステートメントは１行に書き換えることができます
    $message->attach(Swift_Attachment::fromPath('/path/to/image.jpg'));


    //php.ini の allow_url_fopen ディレクティブが有効であれば URL からファイルを添付することができます
    $message->attach(Swift_Attachment::fromPath('http://site.tld/logo.png'));

### ファイル名を設定する

通常は添付ファイルの名前を明示する必要はありません。添付ファイルの名前はデフォルトで使われるからです。しかしファイル名をセットしたい場合は Attachment オブジェクトの `setFilename()` メソッドを使うことができます。

添付ファイルの名前を変更するには:

 * `setFilename()` メソッドを呼び出します。

添付ファイルは通常どおりに添付されますが、メール内部で送られるメタデータはファイルを別のものにリネームします。

    [php]
    //添付ファイルを作り setFilename() メソッドを呼び出します
    $attachment = Swift_Attachment::fromPath('/path/to/image.jpg')
      ->setFilename('cool.jpg');


    //流れるようなインターフェイスが実装されているので、1行で書くことができます
    $message->attach(
      Swift_Attachment::fromPath('/path/to/image.jpg')->setFilename('cool.jpg')
    );

### 動的なコンテンツを添付する

PDF ドキュメントや GD を通して作られた画像はディスクに直接書き込まなくても直接メッセージに添付することができます。標準の `Swift_Attachment::newInstance()` メソッドを使います。

コンテンツを動的に添付するには:

 * 通常どおりにコンテンツを作ります。

 * `Swift_Attachment::newInstance()` で添付ファイルを作り、名前と Content-Type と一緒にコンテンツのソースデータを指定します。

 * `attach()` メソッド添付ファイルをメッセージに追加します。
添付ファイルは受信者にダウンロード可能なファイルとして指定したファイル名と Content-Type と一緒に表示されます。

>**NOTE**
>とにかくいつもファイルをディスクに書き込みたいのであれば、メモリの使用量が少なくてすむ `Swift_Attachment::fromPath()` メソッドで添付すべきです。

    [php]
    //通常の方法でファイルの内容を作りますが、ディスクに書き込みません
    $data = create_my_pdf_data();

    //データから添付ファイルを作ります
    $attachment = Swift_Attachment::newInstance($data, 'my-file.pdf', 'application/pdf');  

    //ファイルをメッセージに添付します
    $message->attach($attachment);


    //添付ファイルをビルドするためにメソッドチェーンを代わりに使うことができます
    $attachment = Swift_Attachment::newInstance()
      ->setFilename('my-file.pdf')
      ->setContentType('application/pdf')
      ->setBody($data)
      ;

### 開封を変更する

添付ファイルは望むのであればデスクトップに保存できるファイルとして表示されます。
添付ファイルの setDisposition() メソッドを使うことで添付ファイルにインラインに表示させることができます。

添付ファイルをインラインに見えるようにするには

 * `setDisposition()` メソッドを呼び出します。

メールクライアントが表示方法をわかっているのであれば、添付ファイルはメール内容を表示するウィンドウの範囲で表示されます。

>**NOTE**
>ZIP アイルなど表示できない種類のファイルのためにインラインの添付ファイルを作ると、メールクライアントは通常の添付ファイルとしてあらわします。

    [php]
    //添付ファイルを作り setDisposition() メソッドを呼び出します
    $attachment = Swift_Attachment::fromPath('/path/to/image.jpg')
      ->setDisposition('inline');


    //流れるようなインターフェイスが実装されているので、1行で書けます
    $message->attach(
      Swift_Attachment::fromPath('/path/to/image.jpg')->setDisposition('inline')
    );

### インラインメディアファイルを埋め込む

メッセージと一緒に画像もしくはほかのコンテンツを含めたいことがよくあります。リモートリソースにリンクする HTML でこれを実現することはかんたんですが、通常は、このアプローチはメールクライアントによってブロックされます。Swift Mailer はメディアをメッセージに直接埋め込むことを可能にします。

メールクライアントは通常リモートリソースからのダウンロードをブロックします。このテクニックはメールを開いたのがだれなのか追跡するための手段として乱用されていたからです。HTML メールを送り、メッセージに画像を含めたい場合、とることのできる別のアプローチは画像を直接埋め込むことです。

Swift Mailer はメッセージへのファイル埋め込みを極限まで効率化します。`embed()` メソッドを呼び出すことでファイルを埋め込みます。このメソッドの戻り値は HTML の `src` もしくは `href` 属性として使うことができます。

添付ファイルのように、既存のファイルを利用可能にしなくても動的にコンテンツを生成することが可能です。

埋め込まれたファイルは HTML 属性の「なかで参照することのできるユニーク ID をもつ特別な種類の添付ファイルとしてメールに送信されます。 埋め込みファイルをサポートしないメールクライアントはそれらのファイルを添付ファイルとして表示することがあります。

画像に関するの共通の作業を説明しましたが、理論的には任意の表示可能な (再生可能な) 種類のメディアでも動きます。しかしながら (ビデオなど)ほかの種類のメディアのサポートに関してはメールクライアントによります。

#### 既存のファイルを埋め込む

ファイルがディスクもしくは URL にすでにあれば、`Swift_EmbeddedFile::fromPath()` メソッドを使うことで、1行でメッセージに埋め込むことができます。

ローカルに存在するファイルを埋め込むことができます。`allow_url_fopen` ディレクティブが有効であれば、ほかのウェブサイトからファイルを埋め込むこともできます。

存在しているファイルを埋め込むには:

 * `Swift_Message::newInstance()` でメッセージオブジェクトを作ります。

 * ボディを HTML としてセットし、`embed()` メソッドでメッセージの正しいポイントでファイルを埋め込みます。

`src` 属性として使われる HTML においてファイルはメッセージンラインで表示されます。

>**NOTE**
>`Swift_Image` と `Swift_EmbeddedFile` はお互いが単なるエイリアスです。`Swift_Image` はセマンティックな目的のために用意されています。

-

>**NOTE**
>お望みであれば2段階でファイルを埋め込むことができます。`embed()` メソッドの戻り値をキャプチャし、それを `src` 属性として使います。

    [php]
    //メッセージを作ります。
    $message = Swift_Message::newInstance('My subject');

    //ボディをセットします。
    $message->setBody(
    '<html>' .
    ' <head></head>' .
    ' <body>' .
    '  Here is an image <img src="' . //ファイルを埋め込みます
         $message->embed(Swift_Image::fromPath('image.png')) .
       '" alt="Image" />' .
    '  Rest of message' .
    ' </body>' .
    '</html>',
      'text/html' // Content-Type を HTML としてセットします
    );

    //php.ini の allow_url_fopen ディレクティブに true がセットされている場合 URL からファイルを埋め込むことができます
    $message->setBody(
    '<html>' .
    ' <head></head>' .
    ' <body>' .
    '  Here is an image <img src="' .
         $message->embed(Swift_Image::fromPath('http://site.tld/logo.png')) .
       '" alt="Image" />' .
    '  Rest of message' .
    ' </body>' .
    '</html>',
      'text/html'
    );


    // embed() メソッドのコードを置くとインラインが扱いにくくなります。
    // 2ステップでこれをおこなうのはかんたんです
    $cid = $message->embed(Swift_Image::fromPath('image.png'));

    $message->setBody(
    '<html>' .
    ' <head></head>' .
    ' <body>' .
    '  Here is an image <img src="' . $cid . '" alt="Image" />' .
    '  メッセージの残り' .
    ' </body>' .
    '</html>',
      'text/html' //Content-Type を HTML としてマークします
    );

#### 動的なコンテンツを埋め込む

GD など実行時に生成される画像はそれらをディスクに書き出さずにメッセージに直接埋め込むことができます。標準の `Swift_Image::newInstance()` メソッドを使います。

動的に作られたコンテンツを埋め込むには:

 * `Swift_Message::newInstance()` でメッセージオブジェクトを作ります。

 * ボディを HTML としてセットし、`embed()` メソッドでメッセージの正しいポイントでファイルを埋め込みます。ファイルの名前と Content-Type を指定する必要があります。

ID が `src` 属性として使われる HTML によってファイルはメッセージインラインで表示されます。

>**NOTE**
>`Swift_Image` と `Swift_EmbeddedFile` の関係はお互いがたんなるエイリアスです。`Swift_Image` はセマンティックな目的のために存在します。

-

>**NOTE**
>お望みであれば2つの段階でファイルを埋め込むことができます。`embed()` メソッドの戻り値をキャプチャし、それを `src` 属性として使います。

    [php]
    //通常の方法でファイルの内容を作りますが、それらをディスクには書き込みません
    $img_data = create_my_image_data();

    //メッセージを作ります
    $message = Swift_Message::newInstance('My subject');

    //ボディをセットします
    $message->setBody(
    '<html>' .
    ' <head></head>' .
    ' <body>' .
    '  Here is an image <img src="' . //ファイルを埋め込みます
         $message->embed(Swift_Image::newInstance($img_data, 'image.jpg', 'image/jpeg')) .
       '" alt="Image" />' .
    '  Rest of message' .
    ' </body>' .
    '</html>',
      'text/html' //Content-Type を HTML としてマークします
    );


    // embed() のコードを置くとインラインが扱いにくくなります。
    // 2つのステップでこれをおこなうのはかんたんです
    $cid = $message->embed(Swift_Image::newInstance($img_data, 'image.jpg', 'image/jpeg'));

    $message->setBody(
    '<html>' .
    ' <head></head>' .
    ' <body>' .
    '  Here is an image <img src="' . $cid . '" alt="Image" />' .
    '  Rest of message' .
    ' </body>' .
    '</html>',
      'text/html' //Content-Type を HTML としてマークします
    );

メッセージに受信者を追加する
----------------------------

`setTo()`、`setCc()` と `setBcc()` メソッドを通じてメッセージ自身の範囲で受信者が指定されます。どこでメッセージが送られたのかわかるように Swift Mailer はメッセージからこれらの受信者を読み込みます。

メッセージの受信者はつぎのタイプの1つです:

 * `To:` 受信者 &#8211; 1番目の受信者 (必須)

 * `Cc:` 受信者 &#8211; メッセージのコピーを受信します (オプション)

 * `Bcc:` 受信者 &#8211; ほかの受信者から隠します
   (オプション)

それぞれのタイプは1つもしくは複数のアドレスを含みます。受信者のアドレスの一覧だけを表示することは可能です。もしくは受信者の本名を提供することでアドレスを個人化することができます。

>**SIDEBAR**
>アドレスの構文
>
>単独のメールアドレス (たとえば `From:` アドレス) を参照することだけがお望みであれば、単なる文字列を使うことができます。
>
>      [php]
>      $message->setFrom('some@address.tld');
>
>名前を含めたいのであれば連想配列を使わなければなりません。
>
>     [php]
>     $message->setFrom(array('some@address.tld' => 'The Name'));
>
>複数のアドレスを含めたいのであれば、配列を使わなければなりません。
>
>     [php]
>     $message->setTo(array('some@address.tld', 'other@address.tld'));
>
>連想配列と非連想配列の構文を組み合わせることで同じリストのなかで個人化 (名前とアドレス) と非個人化したアドレスを混ぜることができます。
>
>     [php]
>     $message->setTo(array(
>       'recipient-with-name@example.org' => '受信者の名前その1',
>       'no-name@example.org', //これはキーバリューのペアではないことにご注意ください
>       'named-recipient@example.org' => '受信者の名前その2'
>     ));

### `To:` 受信者を設定する

`To:` 受信者はメッセージに必須項目でメッセージの 
`setTo()` もしくは `addTo()` メソッドでセットされます。

`To:` 受信者をセットするには、`new Swift_Message( ... )` もしくは
`Swift_Message::newInstance( ... )`　のどちらを使ってメッセージオブジェクトを作り、アドレスの完全な配列で
`setTo()` メソッドを呼び出すか、受信者を1人ずつ追加するには `addTo()` メソッドを呼び出します。

`setTo()` メソッドはこの章の最初のほうで説明したさまざまなフォーマットでのインプットを受け入れます。`addTo()` メソッドは1つもしくは2つのパラメータのどちらをとります。1番目のパラメータはメールアドレスで2番目のオプションパラメータは受信者の名前です。

`To:` 受信者はメッセージヘッダーのなかで見ることが可能で、ほかの受信者に見えます。

>**NOTE**
>すべてのアドレスを見せずにそれぞれの受信者にメッセージをデリバリするために `batchSend()` メソッドを実行したいときは `setTo()` メソッドも使うべきです。このシナリオでは Swift Mailer はプロセス送信のあいだにメッセージ上で内部の変換を実行します。

-

>**NOTE**
>`setTo()` メソッドを複数回呼び出しても新しい受信者は追加されません &#8211;
>それぞれの呼び出しは以前の呼び出しをオーバーライドします。受信者を1人ずつ追加したいのであれば、`addTo()` メソッドを使います。

    [php]
    //すべての受信者を一度にセットするには setTo() メソッドを使います
    $message->setTo(array(
      'person1@example.org',
      'person2@otherdomain.org' => 'Person 2 Name',
      'person3@example.org',
      'person4@example.org',
      'person5@example.org' => 'Person 5 Name'
    ));

    //受信者を1人ずつ追加するには addTo() メソッドを使います
    $message->addTo('person1@example.org');
    $message->addTo('person2@example.org', 'Person 2 Name');

### `Cc:` 受信者を設定する

`Cc:` 受信者はメッセージの `setCc()` もしくは `addCc()` メソッドでセットされます。

`Cc:` 受信者をセットするには、`new Swift_Message( ... )` もしくは
`Swift_Message::newInstance( ... )` のどちらを使ってメッセージオブジェクトを作り、`setCc()` メソッドを呼び出します。アドレスの完全な配列もしくは、受信者を1人ずつ追加するには `addCc()` メソッドを使います。

`setCc()` メソッドはこの章の最初のほうで説明したさまざまなフォーマットでのインプットを受け入れます。`addCc()` メソッドは1つもしくは2つのパラメータのどちらかをとります。1番目のパラメータはメールアドレスで2番目のオプションパラメータは受信者の名前です。

`Cc:` 受信者はメッセージヘッダーのなかで見ることが可能で、ほかの受信者に見えます。

>**NOTE**
>`setCc()` メソッドへの複数の呼び出しは新しい受信者を追加しません &#8211; それぞれの呼び出しは以前の呼び出しをオーバーライドします。`Cc: ` 受信者を1人ずつ追加したいのであれば、`addCc()` メソッドを使います。

    [php]
    //すべての受信者を一度に設定するには setCc() メソッドを使います
    $message->setCc(array(
      'person1@example.org',
      'person2@otherdomain.org' => 'Person 2 Name',
      'person3@example.org',
      'person4@example.org',
      'person5@example.org' => 'Person 5 Name'
    ));

    //受信者を1人ずつ追加するには  addCc() メソッドを使います
    $message->addCc('person1@example.org');
    $message->addCc('person2@example.org', 'Person 2 Name');

### `Bcc:` 受信者を設定する

`Bcc:` 受信者はほかのひとに知られることなくメッセージのコピーを受けとり、`setBcc()` もしくは `addBcc` メソッドによってセットされます。

`Bcc:` 受信者をセットするには、`new Swift_Message( ... )` もしくは
`Swift_Message::newInstance( ... )` かどちらかのメッセージオブジェクトを作り、then call the
`setBcc()` メソッドを呼び出します with a complete array of addresses、もしくは `addBcc()` メソッドを使います。

`setBcc()` メソッドはこの章の最初のほうで説明したさまざまなフォーマットでの入力を受け入れます。`addBcc()` メソッドは1つもしくは2つのパラメータのどちらかをとります。1番目はメールアドレスで2番目のオプションパラメータは受信者の名前です。

個別の `Bcc:` 受信者はメッセージヘッダーのなかでかれらのアドレスを見るだけとなります。ほかの受信者 (ほかの `Bcc:`
受信者を含む) にはアドレスは見えません。

>**NOTE**
>あきらかではない受信者でバルクメールを送信するためのメカニズムとして `Bcc:` 受信者を使うべきではありません。代わりに `batchSend()` メソッドを実行すべきです。

-

>**NOTE**
>`setBcc()` メソッドへの複数の呼び出しは新しい受信者を追加しません &#8211; それぞれの呼び出しは以前の呼び出しをオーバーライドします。`Bcc: ` の受信者を1人ずつ追加したいのであれば、`addBcc()` メソッドを使います。

    [php]
    //一度にすべての受信者を設定するには setBcc() メソッドを使います
    $message->setBcc(array(
      'person1@example.org',
      'person2@otherdomain.org' => 'Person 2 Name',
      'person3@example.org',
      'person4@example.org',
      'person5@example.org' => 'Person 5 Name'
    ));

    //受信者を1人ずつ追加するために addBcc() メソッドを使います
    $message->addBcc('person1@example.org');
    $message->addBcc('person2@example.org', 'Person 2 Name');

送信者の詳細を指定する
-----------------------

メールは送信者に関する情報を含まなければなりません。通常、これは `From:` アドレスによって管理されますが、ほかにもオプションがあります。

送信者の情報は最大で3つの場所に含まれます:

 * `From:` &#8211; メッセージを書いたひとのアドレス (必須)

 * `Sender:` &#8211; メッセージを送った1人のアドレス (オプション)

 * `Return-Path:` &#8211; バウンスが向かうアドレス (オプション)

メッセージオブジェクトの `setFrom()` を使うことで `From:` アドレスをつねに含めなければなりません。ほかに指定しないかぎり、Swift Mailer はこれをデフォルトの `Return-Path:` として使います。

`Sender:` アドレスが存在するのはメールを送信したひとがかならずしもメールを書いたひとではないからです。これは `From:` アドレスよりも優先順位が高く、ほかに指定されていないかぎり `Return-Path:` として使われます。unless otherwise specified.

### `From:` アドレスを設定する

`From:` アドレスは必須でメッセージの `setFrom()` メソッドでセットされます。

`From:` アドレスは実際にメールを書いたひとと通常はそれを送ったひとを指定します。

大半のひとが理解しないことは複数のひとがメールを書いた場合の複数の `From:` アドレスです &#8211;
たとえば、メールが委員会によってまとめられた場合をあげることができます。

`From:` アドレスをセットするには:

 * メッセージオブジェクトの `setFrom()` メソッドを呼び出します。

The `From:` アドレスはメッセージヘッダーで見ることが可能で、受信者が見ることになります。

>**NOTE**
>複数の `From:` アドレスをセットすれば、物理的にメッセージを送る人を示す `Sender:` アドレスをセットしなければなりません。

    [php]
    //単独の From: アドレスをセットします
    $message->setFrom('your@address.tld');

    //名前を含む From: アドレスをセットします
    $message->setFrom(array('your@address.tld' => 'Your Name'));

    //複数のひとがメールを書く場合、複数の From: アドレスをセットします
    $message->setFrom(array(
      'person1@example.org' => 'Sender One',
      'person2@example.org' => 'Sender Two'
    ));

### `Sender:` アドレスを設定する

`Sender:` アドレスはメッセージを送る相手を指し示し、メッセージオブジェクトの `setSender()` メソッドでセットされます。

`Sender:` アドレスをセットするに:

 * メソッドオブジェクトの `setSender()` メソッドを呼び出します。

メッセージヘッダーのなかで見える `Sender:` アドレスは受信者に見えます。

ほかのものが指定されないかぎり、このアドレスが `Return-Path:` として使われます。

>**NOTE**
>複数の `From:` アドレスを設定すると 誰がメッセージを送ったのかを示すために `Sender:` アドレスを設定しなければなりません。

複数の送信者アドレスをセットすべきではありません。1つのメッセージを送るのに複数人を指定できないからです。

    [php]
    $message->setSender('your@address.tld');

### `Return-Path:` (Bounce) アドレスを設定する

`Return-Path:` アドレスはバウンス通知が送信される行き先を指定し、メッセージの `setReturnPath()` メソッドで設定されます。

用意できる `Return-Path:` は1つだけで、個人の名前を含めなくてもかまいません。

`Return-Path:` アドレスを設定するには:

 * メッセージオブジェクトの `setReturnPath()` メソッドを呼び出します。

バウンス通知はつぎのメールアドレスに送られます。

    [php]
    $message->setReturnPath('bounces@address.tld');

受信通知をリクエストする
-------------------------

メールが開かれたときにアドレスに送られる受信通知をリクエストすることは可能です。受信通知をリクエストするには `setReadReceiptTo()` メソッドとアドレスをセットします。

受信通知をリクエストするには:

 * 受信者に遅らせたいアドレスはメッセージオブジェクトの `setReadReceiptTo()` メソッドでセットします。

メールが開かれたときに、メールクライアントがサポートしていれば、通知がそのアドレスに送られます。

>**NOTE**
>受信通知は受信者の大半には機能しません。多くのメールクライアントはこれらを自動的に無効にするからです。受信通知を送るこのようなクライアントはリクエストされたことをユーザーに伝えます。

    [php]
    $message->setReadReceiptTo('your@address.tld');

文字集合を設定する
------------------

メッセージの文字集合 (とこれは MIME の一部) は `setCharset()` メソッドで設定できます。Swift_Preferences クラスと連携してグローバルのデフォルトの文字集合である UTF-8 を変更することもできます。

Swift Mailer のデフォルトの文字集合はオーバーライドされないかぎり UTF-8 です。UTF-8 は大半の事例でうまくゆきます。世界中の大半の文字に加えて標準の US キーボード文字をすべて含むからです。

これはメッセージ (もしくは MIME パート) がどんな文字集合で書かれているのか知っているとしても、絶対に必要不可欠です。
さもなければメッセージがめちゃめちゃに表示される可能性があります。

Swift Mailer で文字集合を変更できる場所は2ヶ所あります。

 * Swift_Preferences クラス
 * それぞれ個別のメッセージかつ/もしくは MIME パート

メッセージオブジェクトの文字集合を設定するにはつぎのようになります。

 * `Swift_Preferences::setCharset()` を呼び出してグローバルな UTF-8 を変更します。

 * メッセージもしくは MIME パートのうえで `setCharset()` メソッドを呼び出します。

    [php]
    //アプローチ 1: グローバル設定を変更します (おすすめ)
    Swift_Preferences::getInstance()->setCharset('iso-8859-2');

    //アプローチ 2: メッセージの setCharset() メソッドを呼び出します。
    $message = Swift_Message::newInstance()
      ->setCharset('iso-8859-2');

    //アプローチ 3: ボディをセットするときに文字集合を指定します。
    $message->setBody('My body', 'text/html', 'iso-8859-2');

    //アプローチ 4: それぞれのパートごとに文字集合を指定します。
    $message->addPart('My part', 'text/plain', 'iso-8859-2');

行の長さを設定する
------------------

行の長さはメッセージの `setMaxLineLength()` メソッドを使うことで変更できます。1000文字未満に保つべきです。

デフォルトでは、Swift Mailer のメッセージの１行の文字数は78文字です。これにはプレーンテキストのターミナルでメッセージを見やすくするための歴史的な理由があります。

メッセージの行の最大長を変更するには:

 * メッセージオブジェクトの `setMaxLineLength()` メソッドを呼び出します。

指定された行の長さよりも長い行は単語のあいだで折りたたまれます。

>**NOTE**
>RFC 2822 にしたがい、1000文字よりも長い最大長を設定すべてではありません。複数の SMTP サーバーのあいだでメッセージがトランスポートされるときにメッセージの一部の切り詰めなど意図しない副作用を引き起こす可能性があるからです。

    [php]
    $message->setMaxLineLength(1000);

メッセージの優先順位を設定する
------------------------------

`setPriority()` メソッドでメッセージの優先順位を変更できます。優先順位を送ることはメールの送信方法を変更することはありません &#8211; これは純粋に受信者に示すためだけの設定です。

メッセージの優先順位は それがもつ重要性を受信者に示す方法です。Swift Mailer では `setPriority` メソッドを呼び出すことで優先順位を設定することができます。このメソッドは1と5のあいだの整数値をとります:

 * Highest
 * High
 * Normal
 * Low
 * Lowest

メッセージの優先順位をセットします。

 * メッセージオブジェクトの `setPriority()` メソッドと1から5のあいだの整数を使って優先順位をセットします。

    [php]
    //"High" 優先順位を示します
    $message->setPriority(2);

