HTTPサービス
===

## ポイント

#### 理解しておきたい用語と概念

* httpd.confの主要なディレクティブ
* キープアライブ
* Apacheの主要モジュール
* 基本認証
* バーチャルホスト
* SSL
* プロキシサーバSquid
* Nginx
* リバースプロキシ

#### 習得しておきたい技術

* Apacheのインストールと設定
* Apacheのモジュールの利用
* 基本認証によるアクセス制御
* ダイジェスト認証によるアクセス制御
* バーチャルホストの設定
* SSLの特徴と利用方法
* プロキシサーバの役割と機能
* Squidのアクセス制御
* Nginxの基本設定

## Apacheの基本的な設定

### Apacheの設定

* メインの設定ファイルはhttpd.conf
    * ソースからインストール
        * /usr/local/apache2/conf/httpd.conf
    * RedHat
        * /etc/httpd/conf/httpd.conf
    * Debian
        * /etc/apache2/apache2.conf
* 機能ごとに複数の設定ファイルに分割し、httpd.confでインクルードして利用する
* 静的モジュールはApacheインストール時に組み込む、動的モジュールはapxsコマンドでインストールする
* 制御用コマンド
    * apachectl
        * RedHat系
    * apache2ctl
        * Devian系
    * サブコマンド
        * start, stop, restart
        * graceful
            * 安全に再起動
        * configtest
            * 設定ファイルの構文チェック
* CGIの利用
    * LoadModuleディレクティブ
        * CGI, PHP実行のためのモジュールをロード
    * ScriptAliasディレクティブ
        * CGIプログラム用のディレクトリを指定

### ディレクティブ

httpd.confの設定項目はディレクティブと呼ばれる

* DocumentRoot
    * web上で公開するファイルのディレクトリを指定
* `ServerRoot ディレクトリ名`
    * httpdが利用するトップディレクトリを指定
* `ServerName サーバのホスト名`
    * Apacheが稼働しているホストのホスト名
* `ServerAdmin メールアドレス`
    * サーバ管理者の連絡先アドレスを指定。エラーページなどに表示される
* `ServerAlias サーバの別名`
    * サーバのエイリアス別名を指定。名前ベースのVirtualHostでのサーバの別名を指定する
* Listen
    * 待受ポート
* User
    * http子プロセスの実行ユーザを指定
* Group
    * http子プロセスの実行グループを指定
* `DirectoryIndex ファイル名 ...`
    * インデックスとして返すファイル名を指定(複数可)
    * 左から順に最初に見つかったファイルが表示される
* Redirect
    * 指定したURLへリダイレクト
    * `Redirect [ステータス] URL-path URL`
        * ステータス
            * `permanent` または `301`
                * リソースが永久に移動
            * `temp` または `302`
                * 一時的なリダイレクト(デフォルト)
    * クライアント側の処理を指示
* Alias
    * ディレクトリとパスを指定してディレクトリのエイリアスを指定
    * サーバ側で処理を実行。Webサーバが移行後の新しいページを直接クライアントへ返す
* ErrorDocument
    * エラー発生時の処理を指定
    * `ErrorDocument エラーコード ファイル名|文字列|URL`
    * ファイル名を指定する場合は先頭に/を記述
* VirtualHost
    * 1台のサーバで2つ以上のwebサイトを管理するバーチャルホスト設定
    * `<VirtualHost IPアドレス[:ポート番号]>...</VirtualHost>`
    * 名前ベースのバーチャルホスト
        * ひとつのIPアドレスに複数のドメイン名を設定
        * `NameVirtualHost`ディレクティブでIPアドレスを設定する
        * SSLはクライアントがSNIに対応していれば使用できる
            * バーチャルホストはHTTPヘッダでどのホスト宛の通信か決まるため、SSLセッションが形成される時点ではどのVirtualHostとの通信化がわからない
            * SNI(Server Name Indication)を使ってSSL接続時にサーバ名を確認する
    * IPベースのバーチャルホスト
        * 複数のIPアドレスに複数のドメイン名を設定
        * `Listen`ディレクティブでIPアドレスを設定
        * SSLはクライアントに依存せずに使用できる

サーバ処理関連のディレクティブ

* `Timeout 秒数`
    * クライアントからリクエストを受け取ってから完了するまでの時間の最大値を指定
* `KeepAlive on|off`
    * ブラウザサーバ間でTCP接続をキープする、KeepAliveの有効無効を指定
    * 1つのTCP接続を使って複数のHTTP処理リクエストをすることができる
* `MaxKeepAliveRequests リクエスト数`
    * KeepAlive時に1つの接続を受け付ける最大リクエスト数を指定
* `KeepAliveTimeout 秒数`
    * KeepAlive時にクライアントからのリクエストを完了してから、コネクションを切断せずに次のリクエストを受け取るまでの最大待ち時間を指定
* `StartServers 子プロセス数`
    * 起動時のプロセス数
* `MaxSpareServers 子プロセス数`, `MinSpareServers 子プロセス数`
    * 待機子プロセスの最大、最小を設定
* `MaxRequestWorkers 子プロセス数`
    * 生成されるhttpd子プロセスの最大数を指定(同時に応答するリクエストの最大数)
* `MaxConnectionsPerChild リクエスト数`
    * http子プロセスが処理するリクエストの最大数を指定

ログ関連のディレクティブ

* `HostnameLookups on|off`
    * クライアントのIPアドレスをログファイルに記載する際、IPアドレスを逆引きホスト名で記録するかどうか指定
* `LogFormat 書式 書式名`
    * アクセスログに使われる書式を定義
* `CustomLog ファイル名 書式名`
    * アクセスログのファイル名と、LogFormatで定義された書式を指定
* `ErrorLog ファイル名`
    * エラーログファイルを指定
* `LogLevel ログレベル`
    * エラーログに記録するログのレベルを指定
* ログファイルは/var/log/httpdディレクトリ以下に格納

外部設定ファイル関連のディレクティブ

* `AccessFileName ファイル名`
    * 外部設定ファイル名を指定
    * デフォルトは.htaccess
* `AllowOverride パラメータ`
    * 外部設定ファイルによるhttpd.confの上書きを許可
    * <Directory>セクション内でのみ使用できる
    * パラメータ
        * `AuthConfig`
            * 認証関係の設定を許可
        * `Limit`
            * Order, Allow, Denyディレクティブの設定を許可
        * `All`
            * すべての設定の変更を許可
        * `None`
            * すべての設定の変更を禁止

例）全てのユーザのホームディレクトリ「/home/*」において、外部設定ファイル「.htaccess」による設定をすべて許可する場合

```
AccessFileName .htaccess
<Directory "/home/*">
AllowOverride All
</Directory>
```

モジュール関連のディレクティブ

* LoadModuleディレクティブ
    * 動的(DSO: Dynamic Shared Object)モジュールをロード
    * apxsコマンド
        * APache eXtenSion tool
        * Apacheの動的モジュールのコンパイルとインストールを行う
    * `LoadModule モジュール名 モジュールのファイル名`
* 主なモジュール
    * `mod_authn_file`
        * .htaccessでのユーザ認証機能を提供
        * データベースからユーザを検索するために利用される
    * `mod_auth_basic`
        * BASIC認証のフロントエンド
    * `mod_auth_digest`
        * ダイジェスト認証のフロントエンド
    * `mod_authz_host`
        * ホストベースのアクセス制御を提供
    * `mod_user`
        * ユーザベースのアクセス制御
    * `mod_access_compat`
        * ホストベースのアクセス制御(互換機能)
        * accessのcompatibility用モジュール
        * 2.3で非推奨となっており旧バージョンとの互換用
    * `mod_so`
        * 動的(DSO)モジュールを組み込む機能を提供
    * `mod_ssl`
        * SSLによる暗号化通信を提供
    * `mod_perl`
    * `mod_php`
* httpdコマンド
    * 静的に組み込まれているモジュールを確認できる
    * httpdコマンドのオプションはapachectlコマンドでも使用することができる
    * `-l` 静的に組み込まれたモジュールを表示
    * `-M` 静的・動的に踏み込まれたモジュールを表示

### クライアントアクセスの認証

基本認証(BASIC認証)

* htttpd.confにユーザ認証設定を追加し、パスワードファイルを用意する
* パスワードが平文で流れる
* 導入に必要な作業
    * htpasswdコマンドを使用しパスワードファイルの作成及びユーザの登録を行う
    * 必要であれば、グループファイルの作成及びグループの登録を行う
    * Apacheの設定ファイルhttpd.confまたは、外部設定ファイル.htaccessでユーザ認証によるアクセス制御を加えたいディレクトリの設定を行う
* htpasswdコマンド
    * BASIC認証のためのユーザ管理のコマンド
    * 書式
        * `htpasswd [オプション] ファイル名 ユーザ名`
    * オプション
        * `-c` パスワードファイルの新規作成
        * `-D` ユーザを削除
    * オプション無しだと、パスワードの変更およびユーザの追加になる

ダイジェスト認証

* チャレンジレスポンス方式の認証
* 盗聴されても直ちにパスワードが漏洩することはない
* 現在ではほとんどのブラウザが対応している
* ユーザに認可領域を指定できる
* htdigestコマンド
    * ダイジェスト認証のためのユーザ管理のコマンド
    * 書式
        * `htdigest [オプション] ファイル名 認可領域 ユーザ名`
    * オプション
        * `-c` パスワードファイルの新規作成
        * `-D` ユーザを削除

ダイジェスト認証の設定例

```
<Directory "/home/test/html">
AuthType Digest
AuthName "private"
AuthUserFile /tmp/htdigest
Require user test1 test2
</Directory>
```

ホストベースのアクセス認証

* Order, Allow, Deny, Requireディレクティブを使ってホスト名ドメイン名でアクセス制御ができる
* Apache2.4では非推奨なのでRequireディレクティブを使う

認証のディレクティブ

* AuthType
    * 認証方式を指定
    * BASIC認証の場合はBasic, ダイジェスト認証の場合はDigestを指定
* AuthName
    * 認可領域名を指定
    * この値は認証の際にメッセージとして表示される
        * 例: `Enter your ID and Password.`
* AuthUserFile
    * 作成したパスワードファイル名を指定
    * BASIC認証では通常.htpasswdが使われる
* AuthGroupFile
    * 作成したグループファイル名を指定
    * BASIC認証では通常は.htgroupが使われる
* Order Deny, Allow
    * Denyディレクティブで広く拒否する範囲を指定し、Allowディレクティブで一部アクセスを許可する範囲を指定する
    * デフォルトすべて許可
* Order Allow, Deny
    * Allowディレクティブで広く許可する範囲を指定し、Denyディレクティブで一部アクセスを拒否する範囲を指定する
    * デフォルト全て拒否
* Require
    * 認証対象とするユーザまたはグループを指定
    * ユーザの場合
        * `Require user ユーザ名 ユーザ名 ....`
    * グループの場合
        * `Require group グループ名 グループ名 ....`
    * 指定したパスワードファイルに登録されているすべてのユーザを認証対象とする
        * `Require valid-user`

Requireディレクティブ

* Apache2.4ではアクセス制御にRequireディレクティブを使う
* 書式
    * `Require [not] エンティティ 値`
* 複数の条件を指定したい場合は、<RequireAll><RequireAny><RequireNone>の3つのディレクティブを使う
* エンティティ
    * `all granted`
        * 全て許可
    * `all denied`
        * 全て拒否
    * `env 環境変数`
        * 指定した環境変数が設定されていると許可
    * `method httpメソッド`
        * 指定したhttpメソッドに合致すると許可
    * `expr 表現`
        * 指定した表現に合致すると許可。条件文を書く
    * `ip`
        * 指定したIPアドレスを許可
    * `user`
        * 指定したユーザを許可
* モジュールが提供するエンティティ
    * mod_authz_core
        * 下記以外
    * mod_authz_host
        * ip, host
    * mod_authz_user
        * user, group, valid-user
    * mod_authzのzは認可、authorization
* 複数の条件をしたい場合のディレクティブ
    * RequireAll
        * すべての条件に合致したら真
    * RequireAny
        * いずれかの条件に合致したら真
    * RequireNone
        * すべての条件に合致しなかったら真

Apache2.2までのアクセス制御設定の例

```
<Directory "/var/www/html/private">
Order Deny,Allow
Deny from all
Allow from 192.168.10.0/24
</Directory>
```

Apache2.4のアクセス制御設定の例(単独条件)

```
Require ip 192.168.0
```

Apache2.4のアクセス制御設定の例(複数条件)

```
<Directory "/var/www/html/private">
<RequireAny>
Require ip 192.168.10
Require group root
</RequireAny>
</Directory>
```

### サーバ情報の取得

* mod_statusモジュールでサーバの稼働状況の情報をブラウザで表示できる
* mod_infoモジュールでサーバの設定情報をブラウザで表示できる

## HTTPS向けのApacheの設定

### SSL

* 公開鍵暗号を使ったセキュリティ技術
* mod_sslモジュールを使用
* ApacheでSSL/TLSを利用する流れ
    * 秘密鍵を作成
    * サーバ証明書を認証局に作成してもらうため、CSR(Certificate Signing Request 証明書の署名要求)を作成する
    * 認証局にCSRを提出し、その後、中間CA証明書とサーバ証明書が認証局より発行される
    * 秘密鍵と中間CA証明書、サーバ証明書をサーバの所定の場所で設置し、ApacheのSSL/TLS用の設定ファイルssl.confでそれぞれのファイルを指定する
    * Apacheを再起動する
* SSLをVirtualHostで利用する
    * 名前ベースの場合、ホスト名を識別する前位にSSL通信を確立しなければならない。クライアントがSNIに対応していればSSLを使用できる
    * IPベースの場合、クライアントに依存せずにSSLを使える

### 証明書

* そのサーバの公開鍵と署名が含まれる
    * 署名は認証局の秘密鍵で行われ、認証局の公開鍵で複合できる
* データの暗号化には共通鍵暗号方式、その鍵の受け渡しには公開鍵暗号方式が使われる
* 自分で認証局を構築するのはオレオレ認証局。当然ルートCA証明書にたどり着けないため、証明書は信頼できない
* サーバ証明書
    * サイトの正当性を証明。サーバの公開鍵は
    * サーバ側に設定
    * 証明書に含まれている情報
        * サーバの公開鍵
            * クライアントとの通信の暗号化に利用
        * 証明書を発行した認証局の署名
            * なりすましを防ぐために利用
    * サーバ証明書を認証局から入手する手順
        * 公開鍵と暗号鍵を作成
        * 公開鍵を認証局(CA)へ送付
        * CAが証明書を発行して返送
        * 返送された証明書をwebサーバにインストール
* 中間CA証明書
    * 自身の公開鍵
    * サーバ側に設定
    * 証明書に含まれている情報
        * 自身の公開鍵
        * 証明書を発行した認証局の署名
* ルートCA証明書
    * 自身の公開鍵
    * ブラウザに内蔵されている

### 関連ファイル

* server.key
    * サーバ秘密鍵
* server.csr
    * 認証局に対する証明書発行要求書
* server.crt
    * サーバ証明書

### opensslコマンド

* SSL証明書や鍵作成に利用
* CSR(サーバ証明書発行要求)
    * `req` を使用
    * `openssl req -new -key 秘密鍵ファイル名 [-out 出力ファイル名]`
* 秘密鍵の作成
    * `genrsa` を使用
    * `openssl genrsa [-out 出力ファイル名] [鍵長]`
* 自己認証局でサーバ証明書を作成
    * `ca` を使用する
    * `openssl ca [-out 出力ファイル名] -infiles CSRファイル名`

### ディレクティブ

ssl.confのディレクティブ

* `ServerTokens`
    * HTTPレスポンスヘッダに出力されるバージョン情報を指定
    * 通常はProd
    * Apacheのバージョンを含めるかどうかも設定できる
* `ServerSignature on|off`
    * エラーメッセージなどのフッタ表示の有効、無効の指定
* `TraceEnable`
    * TRACEメソッドの使用の有効、無効の指定
* `SSLCertificateKeyFile`
    * サーバの秘密鍵のファイルを指定
* `SSLCertificateFile`
    * サーバ証明書のファイルを指定
    * 中間CA証明書があるときは1つにまとめて指定
* `SSLVerifyClient`
    * クライアント認証のレベルを指定
    * requireでクライアント証明書が必須
    * クライアント認証は、認証局にCSR(証明書の署名要求)を提出し、認証局の秘密鍵で署名された証明書を発行してもらう
* `SSLCACertificateFile`
    * クライアント認証に使用するCA証明書のファイルを指定
    * 中間CA証明書とサーバ証明書を1ファイルにまとめて指定
* `SSLCACertificatePath`
    * クライアント認証に使用するCA証明書のファイルが置かれたディレクトリを指定
* `SSLProtocol`
    * 使用可能なSSLプロトコルを指定
    * -をつけると無効化、+をつけると有効化
* `SSLCipherSuite`
    * 使用可能な暗号スイートを指定
    * 鍵長が128bitより大きく、MD5を使用していない暗号スイートを使用する例
        * `SSLCipherSuite HIGH:MEDIUM:!MD5`
* `SSLEngine`
    * SSLの有効無効を指定
    * 有効なコンテキストは、サーバ設定ファイル、またはバーチャルホスト
        * コンテキストは、ディレクティブが有効な範囲

## キャッシュプロキシとしてのSquidの実装

### プロキシサーバの利点

* クライアントからのアクセス制御
* キャッシュによるアクセスの高速化、ネットワークトラフィックの削減

### Squid

* Linuxで最もよく利用されているプロキシサーバ
* スクウィッド

### 設定ファイル

/etc/squid/squid.conf 

* `http_port [IPアドレス:]ポート番号`
    * Squidが利用するポート番号
* `hierarchy_stoplist 文字列`
    * キャッシュを利用しない文字列を指定
* `maximum_object_size`
    * キャッシュされるデータのサイズ
* `maximum_object_size_in_memory`
    * メモリにキャッシュされる最大ファイルサイズ
* `ipcache_size 数`
    * IPアドレスの名前解決をキャッシュする数
* `cache_dir ディレクトリ名`
    * キャッシュディレクトリとパラメータ
    * パラメータは、ディレクトリの最大容量、ディレクトリの数、サブディレクトリの数など
* `cache_mem バイト数`
    * メモリ上のキャッシュサイズ
* `cache_log ファイル名`
    * キャッシュのログ
* `request_header_max_size`
    * HTTPリクエストヘッダの最大サイズ
* `request_body_max_size`
    * HTTPリクエストボディの最大サイズ
* `auth_param 認証方式 program 認証プログラム パスワードファイル`
    * ユーザ認証の方式等を設定
    * パスワードファイル「/usr/local/passwd」に記載されているユーザ全員をBASIC認証の対象とする例
        * `auth_param basic program /usr/local/squid/libexec/ncsa_auth /usr/local/passwd`

### アクセス制御の設定

* BASIC認証によるユーザ認証
    * auth_paramで認証方式(basic)、認証プログラム、パスワードファイルを指定
    * aclでproxy_authを使用して認証対象を定義
    * http_accessでアクセス制御を行う

* acl
    * ホストやプロトコルの集合にACL名をつける
    * ポート番号、IPアドレスなどさあざまなタイプを指定することができる
    * 書式
        * `acl ACL名 ACLタイプ 文字列|ファイル名`
        * ファイル名はダブルクォーテーションで囲う
    * ACLタイプ
        * `src IPアドレス/マスク`
            * クライアントのIPアドレス
        * `dst IPアドレス/マスク`
            * 宛先のIPアドレス
        * `srcdomain ドメイン名`
            * クライアントのドメイン名
        * `dstdomain ドメイン名`
            * クライアントのドメイン名
        * `port ポート番号`
            * 宛先のポート番号
        * `arp`
            * MACアドレス
        * `proto`
            * プロトコル
        * `method メソッド名`
            * HTTPリクエストのメソッド
        * `time S|M|T|W|H|F|A 開始(時:分)-終了(時:分)`
            * 曜日と時間
            * S〜Aは日曜から土曜
            * 月曜日から金曜日の9:00から18:00の間をACL名eigyobiとして定義する場合
                * `acl eigyobi time MTWHF 09:00-18:00`
        * `url_regex 文字列`
            * 正規表現を使ったURL
            * 正規表現のリストが格納されているファイルを指定することもできる。その場合はファイル名をダブルクォーテーションで囲う
                * `acl blacklist url_regex "/var/squid/blacklist_url"`
        * `urlpath_regex 文字列`
            * 正規表現を使ったURL(プロトコルとホスト名を除く)
            * 例）testという文字列を含むURL（プロトコルとホスト名を除く）をACL名Unwantedとして定義する場合
                * `acl Unwanted urlpath_regex test`
        * `proxy_auth ユーザ名`
            * ユーザ認証の対象
            * 例）ユーザhogeを認証対象として、ACL名passwordを定義する場合
            * `acl password proxy_auth hoge`
* http_access
    * アクセス制御を設定する。設定したACLを利用
    * 書式
        * `http_access allow|deny ACL名`
        * 「!」を付加すると「acl名」の内容が反転
        * 上から順に処理され、1つの条件にマッチしたらそれ以降の条件はチェックされない
        * 2つ以上のACL名を指定した場合、AND条件ですべての条件が一致した場合のみアクセス制御が適用される
        * 最終行が「deny」の場合、条件にマッチしなかった通信は全て許可
        * 最終行が「allow」の場合、条件にマッチしなかった通信は全て拒否

ACLの設定例

```
acl all src 0.0.0.0/0.0.0.0
acl localmembers src 192.168.10.0/255.255.255.0
acl SSL port 443
acl OK_port port 80 443
acl CONNECT method CONNECT
acl clients srcdomain test.com
acl eigyobi time MTWHF 09:00-18:00

http_access allow localmembers OK_port
http_access deny !OK_port
http_access deny CONNECT !SSL
http_access allow clients
http_access allow eigyobi
```

## Nginxの実装

### Nginx

* 高速で動作し負荷に強いWebサーバ。リバースプロキシサーバ、メールプロキシサーバの機能も有している
* マスタープロセスと複数のワーカープロセスから構成される

### 設定

/etc/nginx/nginx.conf

* /etc/nginx/, /etc/nginx/conf.d/ 以下に複数のファイルを配置してnginx.confに読み込んで利用
* `-t` nginx.confファイルの構文をチェック

### ディレクティブ

nginx.confのディレクティブ

* 階層を持つことができるものをコンテキストと呼ぶ
* 最上位にある階層はmainコンテキスト
* mainコンテキストの下には、http, server, locationの順に階層構造を形成
* `http {}`
    * httpサーバとしての設定
    * main内で使用
* `server {}`
    * バーチャルホストの設定
    * http内で使用
* `location プレフィックス URIにパスの条件 {}`
    * 条件にマッチするリクエストURLに対する設定
    * server, location内で使用
* `listen IPアドレス:ポート;`
    * リクエストを受け付けるIPアドレスとポート番号
    * server内で使用
* `server_name 名前 ...;`
    * バーチャルホストの名前の指定
    * server内で使用
* `index ファイル名 ...;`
    * インデックスとして返すファイル名の指定
* `root パス;`
    * webで公開するHTMLを保存する最上位のディレクトリの指定
* `proxy_pass URL;`
    * プロキシ先の指定。リバースプロキシの設定
    * location内で使用し、リクエスト転送先であるWebサーバを指定する
* `proxy_set_header フィールド 値;`
    * プロキシ先に送られるリクエストヘッダフィールドの再定義、追加
    * ヘッダフィールドのHostに変数$hostの値を指定する例
        * `proxy_set_header Host $host;`
* `fastcgi_pass IPアドレス:ポート;`
    * FastCGIサーバの指定
    * FastCGIはWebアプリケーション用のインターフェース
* `fastcgi_param パラメータ 値;`
    * FastCGIにわたすパラメータ設定の指定

全体的な設定

```
...
http {
    httpサーバとしての共通の設定
    server {
        バーチャルホストの設定
        location URIのパスの条件 {
            条件にマッチしたリクエストURIに対する設定
        }
        ...
    }
    ...
}
```

「www.example.com」へのリクエスト全てを「192.168.1.10」へ転送する場合

```
server {
    listen 80;
    server_name www.example.com;
    location / {
        proxy_pass http://192.168.1.10/
        proxy_set_header Host $host;
        proxy_pass_header Server;
    }
}
```

### リバースプロキシの設定

* コンテンツをキャッシュしてクライアントに提供
* アクセス元からHTTPヘッダを転送する必要がある
* proxy_set_header

