# 4d-tips-netkit
4D NetKitまとめ

## 基本情報

### [4D NetKit](https://github.com/4d/4D-NetKit)とは

サードパーティ製Webサービスと連携することを支援するためのライブラリ。汎用的な設計を銘打っており，ソースコードも公開されていますが，v19 R6の時点でサポートされているプロトコルは，**OAuth 2.0 API**のみ，サポートされているサードパーティ製Webサービスは[**Microsoft Graph API**](https://docs.microsoft.com/en-us/graph/overview)のみ，となっています。

|バージョン|備考|
|:-:|:-|
|19 R3|[OAuth2Provider](https://blog.4d.com/4d-and-microsoft-365-access/), IMAP, SMTP, POP3|
|19 R4||
|19 R5||
|19 R6|[Office365](https://blog.4d.com/microsoft-365-send-emails/) Mail.Send|

**注記**: GitHubのリポジトリにあるのは， *master* および *19RX* というブランチだけです。毎回のフィーチャーリリースで19R3, 19R4, 19R5…といったブランチやスナップショットが作成されるわけではありません。

### OAuthとは

OAuthは，ペーシック認証（ユーザー名とパスワードの入力を求めること）に代わる**ログインの仕組み**です。[Exchange Web Services (EWS) ](https://docs.microsoft.com/ja-jp/exchange/client-developer/exchange-web-services/explore-the-ews-managed-api-ews-and-web-services-in-exchange), IMAP, SMTP, POP3など，ペーシック認証をサポートするテクノロジーはレガシーとされており，現行の[Microsoft Graph API](https://docs.microsoft.com/ja-jp/graph/overview)は全面的に[OAuth 2.0を採用しています](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-v2-protocols)。

OAuthでログインすると，有効期限付きの**トークン**が発行され，以後，ユーザー名とパスワードの代わりに認証トークンを使ってログインセッションを管理します。4Dの**IMAP/POP3/SMTPトランスポーター**は，v18 R6以降，いずれもベーシック認証と[トークン認証](https://blog.4d.com/connect-your-mail-transporters-with-an-oauth-2-0-token/)の両方をサポートしています。

### v19 R3とv19 R6のメール送信は何が違うのか

v19 R3では，OAuth 2.0の認証トークンを取得し，IMAPやSMTPでメールを送信することができるようになりました。しかし，Office 365アカウントは，SMTPのセキュリティ設定を緩和する必要があります。そこで，v19 R6では，**Graph APIによる新しいメール送信**（Mail.Send）のサポートが追加されました。認証にOAuth 2.0を使用する点はどちらも同じですが，v19 R6の`Office365`クラスは，SMTPの代わりに[Microsoft Graphのメール送信API](https://docs.microsoft.com/ja-jp/graph/api/user-sendmail?view=graph-rest-1.0&tabs=http)を呼び出すようになっており，Office 365アカウントのセキュリティ設定を変更しなくてもメールが送信できます。

### Office 365アカウントの設定

Office 365アカウントでSMTPを使用するためには，管理者アカウントでログインし，**認証済みSMTP**を有効に設定する必要があります。

`SMTP AUTH`が許可されていない場合，トークン認証には成功しても，`SmtpClientAuthentication`が無効にされているために，APIの呼び出しが不成功に終わります。

> 5.7.139 Authentication unsuccessful, SmtpClientAuthentication is disabled for the Tenant. Visit https://aka.ms/smtp_auth_disabled for more information. [TYCP286CA0104.JPNP286.PROD.OUTLOOK.COM]

https://docs.microsoft.com/ja-jp/exchange/clients-and-mobile-in-exchange-online/authenticated-client-smtp-submission#enable-smtp-auth-for-specific-mailboxes

### 対応バージョン

**4D NetKit**は，4Dアプリの *Components* フォルダーに標準で組み込まれています。v19 LTS（19.x）には組み込まれていません。ただし，19 R5以前のバージョンであれば，フィーチャーリリースに限定された新しい機能を使用しているわけではないため，プロジェクトの *Components* フォルダーにインストールすれば，互換レベルで動作するようです。19 R6では `4D.HTTPRequest` クラスが採用され，LTSおよび19 R5以前のバージョンでは実行時にエラーが返されるようになりました。ただし，新しいコマンドを使用しているわけではなく，`4D`クラスストアの新しいプロパティをコールしているため，シンタックスチェックとコンパイルはいずれも成功します。

**注記**: 本来，v19.xとv19 Rxは，同一のメジャーバージョン番号に属するため，v19 R6はメジャーアップグレードに該当せず，再コンパイルも必要ないはずでした。しかしApple Siliconターゲットの改良に伴い，__例外的に19 R6はv19.xとv19 Rxシリーズの後継世代として扱われます__。LTSおよび19 R5以前のバージョンでプロジェクトを開くことはできますが，R5/R6を跨ぐたびに再コンパイルが必要となります。

### ライセンス

4D NetKitは4Dアプリに組み込まれており，追加ライセンスは不要ですが，OAuthのコールバック処理にコンポーネントのWebサーバーを使用しているため，アクセストークンの取得には**Web Application Expansion**のライセンスが必要です。

OAuthでは，初回の認証でアクセストークンとリフレッシュトークンが発行されます。いずれのトークンにも有効期限が設定されています。ほとんどの場合，アクセストークンの有効期限は`10`分から`30`分です。有効なアクセストークンは何度でも使用することができます。リフレッシュトークンの期限はずっと長く，数週間から数ヶ月は有効です。リフレッシュトークンを使用することにより，アクセストークンとリフレッシュトークンを更新することができます。

初回の認証では，ブラウザからリダイレクトを処理するためにWebサーバーを使用します。以後のトークン管理はRESTful APIで処理されるため，Webサーバーは必要ありません。リフレッシュトークンの期限が切れてしまった場合，初回の認証からやり直す必要があります。

### コンポーネントアーキテクチャー

コンポーネントはプロジェクトモードで開発されており，`cs.NetKit`名前空間でクラスが公開されています。バイナリモードのホストからでもインスタンス化できるように`New OAuth2 provider` `New Office365 provider`という共有プロジェクトメソッドが用意されています。

### エラーハンドリング

4D NetKitは，`_4D THROW ERROR`という内部コマンドをエラー処理の中で使用しています。このコマンドの使い方はドキュメントに説明されていませんが，コンポーネント内で呼び出された場合，標準のエラーをホストに「投げる」効果があるようです。

---

### Microsoft Office365

OAuthのトークン認証をする場合，対応しているWebサービスのサイトでアプリを登録するというステップを踏むことになります。Office365のOAuth APIを使用するのであれば，Microsoft Azureプラットフォームにアプリを登録することから始めます。

## アプリの登録

[www.office.com](https://www.office.com)にログインします。

<img width="634" alt="1" src="https://user-images.githubusercontent.com/1725068/186809085-1cd0806d-aeb9-437d-93ee-d2f7539f3722.png">

[portal.azure.com](https://portal.azure.com/#home)にアクセスし，**Azure Active Directoryの管理**の「ビュー」ボタンをクリックします。

<img width="634" alt="3" src="https://user-images.githubusercontent.com/1725068/186809401-8e29cc1f-fa93-47b8-98a7-dd1b01ec9e3b.png">

サイドバーから「アプリの登録」を選択します。

<img width="634" alt="5" src="https://user-images.githubusercontent.com/1725068/186810015-2f1d0b71-9192-4378-9b62-72e13b514676.png">

「新規登録」ボタンをクリックします。

<img width="634" alt="1" src="https://user-images.githubusercontent.com/1725068/186811065-25cc7ecb-75d6-4367-8f5b-c1144f96a4f8.png">

* 名前: 認証画面などのメッセージに表示されるアプリケーション名を入力します。

<img width="611" alt="3" src="https://user-images.githubusercontent.com/1725068/186811085-789fe5ef-0aad-49bc-8f67-fcf087758b5f.png">

* サポートされているアカウントの種類: 対象ユーザーを「すべてのMicrosoftアカウントユーザー」に設定します。

<img width="610" alt="4" src="https://user-images.githubusercontent.com/1725068/186811124-e52b23b2-f342-441d-80db-2752c332591b.png">

**注記**: 対象ユーザーを個人用のMicrosoft アカウントユーザー（`PersonalMicrosoftAccount`）に設定するとエラーが返されます。

> AADSTS9002331: Application 'a035cd95-0c13-4b9f-a2b0-64b69c44fed5'(TEST) is configured for use by Microsoft Account users only. Please use the /consumers endpoint to serve this request.

* プラットフォーム: 通常はWebプラットフォームを選択します。シングルページアプリケーションを選択すると認証エラーになります。

> AADSTS9002325: Proof Key for Code Exchange is required for cross-origin authorization code redemption.

デスクトップアプリケーション（パブリッククライアント）をプラットフォームにすることもできます。その場合，リダイレクトURIはポート番号もチェックされます。対照的にWebプラットフォームを選択し，リダイレクトURIをループバックアドレスに設定した場合，ポート番号がマニフェストで設定したものとは一致しなくてもリダイレクトを処理することができます。

**注記**: プラットフォームをデスクトップアプリケーションに設定した場合，パブリッククライアントフローを有効にすることもできます。パブリッククライアントは`clientSecret`が省略されていてもトークン認証が成功します。

<img width="644" alt="スクリーンショット 2022-08-30 7 53 38" src="https://user-images.githubusercontent.com/1725068/187317103-c24d7f2e-1790-47bf-a668-1f66390f01d7.png">

* リダイレクトURI: プラットフォームを選択し，`http://127.0.0.1:50993/authorize`と入力します。シングルページアプリケーションまたはWebプラットフォームを選択した場合，`http:`プロトコルでIPリテラルループバックアドレス`127.0.0.1`を設定することはできないので，一旦，`http://localhost:50993/authorize`と入力してから後でマニフェストを書き換えます。デスクトップアプリケーション（パブリッククライアント）であれば，そのまま入力することができます。

**注記**: URLの末尾にスラッシュを追加した場合，APIを呼び出すときもスラッシュを省略せずに追加する必要があります。

<img width="602" alt="2" src="https://user-images.githubusercontent.com/1725068/186811155-81573bd6-c76e-4759-8f95-583576e44069.png">

「登録」ボタンをクリックします。

サマリー画面が表示されるので，下記の項目を書き留めます。

* アプリケーション (クライアント) ID
* ディレクトリ (テナント) ID

「証明書またはシークレットの追加」をクリックし，新しいクライアントシークレットを発行します。

<img width="560" alt="a" src="https://user-images.githubusercontent.com/1725068/186812161-d69e9131-ef55-4351-86c6-e38be76aab61.png">

この画面に再度アクセスしたときには表示されないので，ここで**必ず**「値」を書き留めます。

<img width="634" alt="b" src="https://user-images.githubusercontent.com/1725068/186812259-e1fd6dac-a6b8-4c32-ba40-0cb5668de111.png">

サイドバーから「マニフェスト」を選択します。

<img width="634" alt="m" src="https://user-images.githubusercontent.com/1725068/186812493-7dfdd872-8dcc-4163-ae35-57a9adcc3a96.png">

マニフェストにはアプリの完全な仕様はJSON形式で記述されています。ポータル画面を操作する代わりに直接マニフェストを編集することができます。

リダイレクトURLを書き換えてから「保存」ボタンをクリックします。

<img width="634" alt="w" src="https://user-images.githubusercontent.com/1725068/186812996-2cffbf3f-d56f-42af-a4a2-a10426951045.png">

#### リダイレクトURI

リダイレクトURIは，[`https:`または`http://localhost`から始まる文字列でなければなりません](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/reply-url)。`http://127.0.0.1:50993/authorize/`と例題には記述されていますが，Webプラットフォームの場合，そのようなURLを登録することができないようになっています。

<img width="398" alt="url" src="https://user-images.githubusercontent.com/1725068/186433291-589ceec7-617d-4739-b96c-19fa587cf8ae.png">

その一方で「リダイレクトURIで`localhost`ではなくIPリテラルループバックアドレス`127.0.0.1`を使用するよう」ドキュメントは推奨しており，対策として（`http:`の場合は）「[アプリケーションマニフェストで`replyUrlsWithType`属性を変更する](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/reference-app-manifest#replyurlswithtype-attribute)」ことが提案されています。つまり，入力画面の制約を乗り越えるため，直接，設定JSONファイルを編集する必要があります。

これをせずに`localhost`設定のままIPリテラルループバックアドレス`127.0.0.1`を使用しようとすると，エラーになります。

> AADSTS50011: The redirect URI 'http://127.0.0.1:50993/authorize/' specified in the request does not match the redirect URIs configured for the application 'c92bf2a4-2388-4702-b0f1-4a13ff2fe5ed'. Make sure the redirect URI sent in the request matches one added to your application in the Azure portal. Navigate to https://aka.ms/redirectUriMismatchError to learn more about how to fix this.

### APIのアクセス許可

OAuth 2.0の認証システムは，プラットフォームのAPI全般に共通ですが，**スコープ**の外にあるAPIは利用できないことになっています。スコープは，サービス名・アプリケーション名・読み書き・リソースの種類などの区分で定義されており，権限が設定できるようになっています。

サイドバーから「APIのアクセス許可」を選択します。

<img width="634" alt="access" src="https://user-images.githubusercontent.com/1725068/186814265-0534c143-6f41-4441-9f58-afd26bd34642.png">

デフォルトで`User.Read`というスコープが設定されています。

アクセス許可を追加し，Microsoft Graphを選択します。

<img width="634" alt="graph" src="https://user-images.githubusercontent.com/1725068/186814362-b59bc4a7-e0ae-4367-ad41-adff7918dc09.png">

「委任されたアクセス許可」をクリックします。

<img width="634" alt="スクリーンショット 2022-08-26 12 57 59" src="https://user-images.githubusercontent.com/1725068/186814505-cee00e38-d8f4-41a3-b568-8da0323f0731.png">

IMAPと入力し，APIを登録します。

<img width="634" alt="スクリーンショット 2022-08-26 12 58 42" src="https://user-images.githubusercontent.com/1725068/186814892-20d9dde4-4a08-4833-a7f1-34760a14ce4d.png">

同じ要領とSMTPとPOPも登録します。

[Office365のアカウント設定](#office-365アカウントの設定)が必要なSMTPの代わりにメール送信APIを使用するのであれば，Mailも登録します。

<img width="634" alt="mail" src="https://user-images.githubusercontent.com/1725068/186816274-613379ae-c813-4b83-a783-512f66a0f92d.png">

<img width="634" alt="スクリーンショット 2022-08-26 13 02 38" src="https://user-images.githubusercontent.com/1725068/186816679-6c6a6517-fd45-4472-8283-17b53452abc6.png">

4Dの**IMAP/POP3/SMTPトランスポーター**でトークン認証を使用したり，Microsoft GraphのAPIを使用してメールを送信したりするための準備ができました。

## OAuthでメール送信

#### ポイント

* トークン認証を使用しているため，**ユーザー名とパスワードを管理する必要がない**。この例題では対応していませんが，実装においては，アクセストークン・リフレッシュトークン・それぞれの有効期限を管理することになります。

* アクセストークンがない，あるいは有効期限を過ぎている: `OAuth2Provider.getToken()`でブラウザが表示され，ユーザーにアプリ連携の承諾を求めます。

* アクセストークンがある: 保存したトークンオブジェクトを`OAuth2Provider.getToken()`をコールする**前に**`OAuth2Provider.token`プロパティにセットします。

**注記**: インスタンス化した`OAuth2Provider`オブジェクトをそのままデータベースのオブジェクト型フィールドに保存することができます。プロパティはもちろんのこと，クラス関数もレコードから再現できます。その場合，保存されるのは関数名だけなので，保存時ではなく，実行時のソースコードに基づき，関数が評価されます。

トークンを管理するためのテーブルを作成します。

<img width="150" alt="table" src="https://user-images.githubusercontent.com/1725068/186934010-48d08775-4648-4927-92c8-ab58d59ee812.png">

認証情報を管理するためのクラスを作成します。

```4d
Class constructor($name : Text)
	
	Case of 
		: ($name="4D OAuth Client")
			
			This.name:="Microsoft"
			This.clientId:="…"
			This.clientSecret:="…"
			This.redirectURI:="http://127.0.0.1:50993/authorize"
			This.permission:="signedIn"
			This.scope:=New collection("https://graph.microsoft.com/.default")
			This.scope.unshift("offline_access")
			
	End case 
```

## OAuth SMTP

[(Archived) Authenticate to the Microsoft Graph API in signedIn mode and send an email with SMTP](https://github.com/4d/4D-NetKit#archived-authenticate-to-the-microsoft-graph-api-in-signedin-mode-and-send-an-email-with-smtp
)の例題を使用します。

```4d
var $token : Object

$account:="keisukemiyako@4djapan.onmicrosoft.com"
$type:="office365"

var $eOAuth2 : cs.OAuth2Entity
var $esOAuth2 : cs.OAuth2Selection

$esOAuth2:=ds.OAuth2.query("account == :1 and type == :2"; $account; $type)  //triple = because we have a wildcard

If ($esOAuth2.length=0)
	
	$param:=cs.App.new("4D OAuth Client")
	
	$param.redirectURI:="http://127.0.0.1:50994/authorize"  //see explanation in the header comment
	$param.scope:="offline_access https://outlook.office.com/SMTP.Send"  //need offline_access to obtain refresh_token
	$oAuth2:=cs.NetKit.OAuth2Provider.new($param)
	
	$eOAuth2:=ds.OAuth2.new()
	$eOAuth2.account:=$account
	$eOAuth2.type:=$type
	$eOAuth2.provider:=$oAuth2
	
Else 
	$eOAuth2:=$esOAuth2.first()
	$oAuth2:=$eOAuth2.provider  //contains functions and tokens
End if 

$token:=$oAuth2.getToken()

If ($token#Null)
	
	$token.token.refreshToken:=$token.token.refresh_token
	
	$eOAuth2.save()  //for next time
	
	// Set the email's content and metadata
	$email:=New object
	$email.subject:="My first mail"
	$email.from:=$account
	$email.to:="keisuke.miyako@4D.com"
	$email.textBody:="Test mail \r\n This is just a test e-mail \r\n Please ignore it"
	
	// Configure the SMTP connection
	$parameters:=New object
	$parameters.accessTokenOAuth2:=$token.token.access_token
	$parameters.authenticationMode:=SMTP authentication OAUTH2
	$parameters.host:="smtp.office365.com"
	$parameters.user:=$account
	$parameters.logFile:=System folder(Desktop)+"smtp.log"
	
	// Send the email 
	
	$smtp:=SMTP New transporter($parameters)
	$statusSend:=$smtp.send($email)
	
Else 
	$eOAuth2.drop()
End if 
```

## OAuth Mail Send

[Office365](https://github.com/4d/4D-NetKit#office365
)の例題を使用します。

```4d
var $token : Object

$account:="keisukemiyako@4djapan.onmicrosoft.com"
$type:="office365"

var $eOAuth2 : cs.OAuth2Entity
var $esOAuth2 : cs.OAuth2Selection

$esOAuth2:=ds.OAuth2.query("account == :1 and type == :2"; $account; $type)  //triple = because we have a wildcard

If ($esOAuth2.length=0)
	
	$param:=cs.App.new("4D OAuth Client")
	
	$oAuth2:=cs.NetKit.OAuth2Provider.new($param)
	
	$eOAuth2:=ds.OAuth2.new()
	$eOAuth2.account:=$account
	$eOAuth2.type:=$type
	$eOAuth2.provider:=$oAuth2
	
Else 
	$eOAuth2:=$esOAuth2.first()
	$oAuth2:=$eOAuth2.provider  //contains functions and tokens
End if 

$token:=$oAuth2.getToken()

If ($token#Null)
	
	$token.token.refreshToken:=$token.token.refresh_token
	
	$eOAuth2.save()  //for next time
	
	$office365:=cs.NetKit.Office365.new($oAuth2; New object("mailType"; "JMAP"))
	
	$user:=$office365.user.getCurrent()
	
	If ($user#Null)
		
		$email:=New object
		$email.from:=$user.mail
		$email.sender:=$user.mail
		$email.subject:="My first mail"
		$email.textBody:="Test mail \r\n This is just a test e-mail \r\n Please ignore it"
		$email.htmlBody:="<html><body><h1>Test email </h1> This is just a test email <br /> Please ignore it</body></html>"
		$email.to:="keisuke.miyako@4D.com"
		
		// Send the email 
		
		$status:=$office365.mail.send($email)
		
	End if 
	
Else 
	$eOAuth2:=$esOAuth2.first()
	$oAuth2:=$eOAuth2.provider  //contains functions and tokens
End if 
```

初回はアプリ連携の承諾をユーザーに求めるためにブラウザが起動します。

<img width="426" alt="auth" src="https://user-images.githubusercontent.com/1725068/186937808-d9f44b97-7b64-49bb-bd1e-7c832ca11bf3.png">

連携を承諾するとリダイレクトURLにページが移動します。

<img width="574" alt="accept" src="https://user-images.githubusercontent.com/1725068/186937911-234ce7df-d93c-467f-9496-a375c44ed174.png">

**参考**: どうしても認証に失敗し，APIのアクセス許可が適正であるように思えるのに下記のエラーる場合，アプリケーションを削除してやり直すと問題が解決することがあります。

> Authorization Request Denied. Insufficient privileges to complete the operation.

## Gmail

4D NetKitはOffice365に特化されたコンポーネントですが，OAuth 2.0は共通の仕様なので，コードを少し書き換えることで他社のサービスにも対応することができます。

4D純正コンポーネントを改造する場合，ライセンス公開の規約により，別名でコンポーネントを公開する必要があります。オリジナルのコンポーネントをアンインストールすることはできないので，クラスの名前空間も`NetKit`とは別のものにしなけれななりません。

コンポーネントのWebサーバーを開始した場合，ローカルホストのリスナーソケットはWebサーバーを終了した後もコンポーネントに占有されたままになるため，他のコンポーネントで同じポート番号を使用することはできません。4D NetKitのHTTPサーバーは，リダイレクトURLのポート番号または`50993`を使用します。オリジナルとカスタム版のNetKitを併用する場合，それぞれのコンポーネントに渡すリダイレクトURLのポート番号を別々にすると良いでしょう。Microsoft Graph APIは，プラットフォームをWebに指定した場合，リダイレクトURLのポート番号をチェックしない仕様になっているため，設定とは違うポート番号でもリダイレクトURLを処理することができます。→[ローカルホストの例外](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/reply-url#localhost-exceptions)

## ダウンロード

カスタム版のNetKitは[こちら](https://github.com/miyako/4d-component-KMNetKit)に公開されています。

---

GmailをOAuthでトークン認証し，SMTPで送信するためには，`https://mail.google.com/`スコープのアクセス権を取得する必要があります。2020年以降，そのような[制限付きスコープ]((https://developers.google.com/terms/api-services-user-data-policy))の取得にはGoogleによるチェックが必要となりました。未確認のアプリは認証画面の前に警告メッセージが表示されます。リストを理解した上で続行することもできます。

<img width="407" alt="" src="https://user-images.githubusercontent.com/1725068/187013750-c6aa8541-9199-4035-b9fb-33fc18e75c45.png">

制度改定前に登録されたアプリはブロックされ，続行することはできません。

---

## トピック

* [ソースは公開されているがフリーウェアではない](https://discuss.4d.com/t/4d-netkit-custom-provider/22350)
* [Googleに接続するためにはどこを書き換えれば良いか](https://discuss.4d.com/t/newfeature-connect-your-mail-transporters-with-an-oauth-2-0-token/17349/16)
* [Gmail APIのスコープ](https://discuss.4d.com/t/oauth-2-0-uses-of-specifics-scopes-with-4ds-smtp-transporter/23744/2)
