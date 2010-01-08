ライブラリをインストールする
===========================

Swift Mailer をインストールする作業はささいなものです。通常は展開されたソースファイルを Web サーバーにアップロードするだけで済みます。

パッケージからインストールする
-----------------------------

ほとんどのユーザーは Swift Mailer の公式サイトからパッケージをダウンロードして Swift Mailer をインストールします。

Swift Mailer を `.tar.gz` もしくは `.zip` ファイルとしてダウンロードする場合インストールはシンプルでアーカイブを展開して Web サーバーにアップロードします。

### ライブラリを展開する

`tar` もしくは 7-Zip などの好きな展開ツールを使ってアーカイブを展開します。

アーカイブを展開するプログラムにアクセスする権限が必要になります。Windows コンピューターでは、7-Zip が機能します。Mac と Linux システムではコマンドラインで `tar` を使うことができます。

ダウンロードしたパッケージを展開するには:

 * アーカイブ管理ソフトウェアの"展開"機能を使います。

ソースコードはアーカイブと同じ名前のディレクトリに置かれます (たとえば Swift-4.0.0-b1)。

次の例は`tar` コマンドを使った Mac OS X と Linux システムのプロセスを示しています。

    chrisbook:build chris$ ls
    Swift-4.0.0-dev.tar.gz
    chrisbook:build chris$ tar xvzf Swift-4.0.0-dev.tar.gz 
    Swift-4.0.0-dev/
    Swift-4.0.0-dev/lib/
    Swift-4.0.0-dev/lib/classes/
    Swift-4.0.0-dev/lib/classes/Swift/
    Swift-4.0.0-dev/lib/classes/Swift/ByteStream/
    Swift-4.0.0-dev/lib/classes/Swift/CharacterReader/
    Swift-4.0.0-dev/lib/classes/Swift/CharacterReaderFactory/
    Swift-4.0.0-dev/lib/classes/Swift/CharacterStream/
    Swift-4.0.0-dev/lib/classes/Swift/Encoder/

      ... etc etc ...

    Swift-4.0.0-dev/tests/unit/Swift/Transport/LoadBalancedTransportTest.php
    Swift-4.0.0-dev/tests/unit/Swift/Transport/SendmailTransportTest.php
    Swift-4.0.0-dev/tests/unit/Swift/Transport/StreamBufferTest.php
    chrisbook:build chris$ cd Swift-4.0.0-dev
    chrisbook:Swift-4.0.0-dev chris$ ls
    CHANGES		LICENSE.GPL	LICENSE.LGPL	README		VERSION		examples	lib		test-suite	tests
    chrisbook:Swift-4.0.0-dev chris$

Git からインストールする
-----------------------

楽に最新バージョンを維持するために Swift Mailer を github.com から直接ダウンロードしてインストールすることが可能です。

Swift Mailer のソースコードは github.com の Git リポジトリで管理されているのでリポジトリからソールを直接得ることができます。

>**NOTE**
>github から Swift Mailer を使うのに Git をインストールする必要はありません。Git をインストールしていない場合、[github][1] に移動して "Download" ボタンをクリックします。

### リポジトリをクローンする

リポジトリは `git clone` コマンドを使って git://github.com/swiftmailer/swiftmailer.git からクローンできます。 

`git clone` コマンドを使う前に `git` コマンドをインストールする必要があります。

リポジトリをクローンするには:

 * 好きなターミナル環境を開きます (コマンドライン)。

 * クローンしたいディレクトリに移動します。

 * `git clone git://github.com/swiftmailer/swiftmailer.git swiftmailer` コマンドを実行します。

ソースコードは "swiftmailer" という名前のディレクトリにダウンロードされます。

次の例は a Linux のような UNIX系システム、BSD もしくは Mac OS X でのプロセスを示しています。

    chrisbook:~ chris$ cd source_code/
    chrisbook:source_code chris$ git clone git://github.com/swiftmailer/swiftmailer.git swiftmailer
    Initialized empty Git repository in /Users/chris/source_code/swiftmailer/.git/
    remote: Counting objects: 6815, done.
    remote: Compressing objects: 100% (2761/2761), done.
    remote: Total 6815 (delta 3641), reused 6326 (delta 3286)
    Receiving objects: 100% (6815/6815), 4.35 MiB | 162 KiB/s, done.
    Resolving deltas: 100% (3641/3641), done.
    Checking out files: 100% (1847/1847), done.
    chrisbook:source_code chris$ cd swiftmailer/
    chrisbook:swiftmailer chris$ ls
    CHANGES		LICENSE.LGPL	README.git	VERSION		docs		lib		test-suite	util
    LICENSE.GPL	README		TODO		build.xml	examples	notes		tests
    chrisbook:swiftmailer chris$

ホストサイトにアップロードする
-----------------------------

必要なのは "lib/" ディレクトリを運用用途の Web ホストにアップロードすることだけです。ほかのフィルとディレクトリはユーザーのサポートのためであって運用には必要ありません。

"lib/" ディレクトリを Web ホストにアップロードするために、FTP、rsync もしくは似たようなソフトウェアが必要になります。

Swift Mailer をアップロードするには:

 * FTP プログラムもしくは rsync/scp を好むのであればコマンドラインを開きます。

 * "lib/" ディレクトリをホスティングアカウントにアップロードします。

これで Swift Mailer を使うのに必要なファイルはホストの PHP がアクセスできるようになります。次の例は Linux もしくは OS X で `rsync` を使ってファイルをアップロードするやり方を示しています。

>**NOTE**
>ファイルを Web 公開ディレクトリに置く必要はありません。これらは PHP スクリプトは これらを"インクルード"できる場所に存在すればいいのです。。

    chrisbook:Swift-4.0.0-dev chris$ rsync -rvz lib d11wtq@swiftmailer.org:swiftmailer
    building file list ... done
    created directory swiftmailer
    lib/
    lib/mime_types.php
    lib/preferences.php
    lib/swift_required.php
    lib/classes/
    lib/classes/Swift/
    lib/classes/Swift/Attachment.php
    lib/classes/Swift/CharacterReader.php
      ... etc etc ...
    lib/dependency_maps/
    lib/dependency_maps/cache_deps.php
    lib/dependency_maps/mime_deps.php
    lib/dependency_maps/transport_deps.php

    sent 151692 bytes  received 2974 bytes  5836.45 bytes/sec
    total size is 401405  speedup is 2.60
    chrisbook:Swift-4.0.0-dev chris$

[1]: http://github.com/swiftmailer/swiftmailer
