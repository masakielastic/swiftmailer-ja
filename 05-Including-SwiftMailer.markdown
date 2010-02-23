Swift Mailer をインクルードする (オートロード)
================================================

Swift Mailer はオートローダーを備えているので、インクルードする必要のあるファイルは `lib/swift_required.php` ファイルだけです。

Swift Mailer のオートローダーを使うには:

 *  Swift Mailer を PHP スクリプトにアクセスできるところに置きます (Web ルートである必要はありません)。

 *  `lib/swift_required.php` ファイルを include もしくは require します。

 *  利用可能なコンポーネントを使うには残りのドキュメントにしたがってください。

>**NOTE**
>Swift Mailer のオートローダーはほかのオートローダーと連携するように設計されていますが、独自のオートローダーを使うために Swift Mailer のオートローダーを避ける必要があります。これを行う必要がある場合、`swift_required.php` の代わりに `swift_init.php` をインクルードします。Swift Mailer はこのファイルがセットアップする Dependency Injection なしで動かないので最小のインクルードは `swift_init.php` ファイルです。

    [php]
    <?php

    require_once '/path/to/swift-mailer/lib/swift_required.php';

    /* rest of code goes here */
