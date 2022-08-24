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

OAuthは，ペーシック認証（ユーザー名とパスワードの入力を求めること）に代わるログインメカニズムのひとつです。[Exchange Web Services (EWS) ](https://docs.microsoft.com/ja-jp/exchange/client-developer/exchange-web-services/explore-the-ews-managed-api-ews-and-web-services-in-exchange), IMAP, SMTP, POP3など，ペーシック認証をサポートするテクノロジーはレガシーとされており，現行の[Microsoft Graph API](https://docs.microsoft.com/ja-jp/graph/overview)は全面的に[OAuth 2.0を採用しています](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/active-directory-v2-protocols)。

よりもずにアカウントを認証することができ，IMAP, SMTP, POP3といった**トランスポーター**を使用することができます。

Office 365のアカウントは，IMAPやSMTPでもメールを送信することができますが，アカウントのセキュリティ設定がかなり面倒です。v19 R6では，**Graph APIによる新しいメール送信**（Mail.Send）もできるようになりました。こちらも認証にはOAuth 2.0を使用しますが，SMTPを使用せずにREST APIでメールを送信する点が違っています。

### 対応バージョン

**4D NetKit**は，4Dアプリの *Components* フォルダーに標準で組み込まれています。v19 LTS（19.x）には組み込まれていません。ただし，19 R5以前のバージョンであれば，フィーチャーリリースに限定された新しい機能を使用しているわけではないため，プロジェクトの *Components* フォルダーにインストールすれば，互換レベルで動作するようです。19 R6では `4D.HTTPRequest` クラスが採用され，LTSおよび19 R5以前のバージョンでは実行時にエラーが返されるようになりました。ただし，新しいコマンドを使用しているわけではなく，`4D`クラスストアの新しいプロパティをコールしているため，シンタックスチェックとコンパイルはいずれも成功します。

**注記**: 本来，v19.xとv19 Rxは，同一のメジャーバージョン番号に属するため，v19 R6はメジャーアップグレードに該当せず，再コンパイルも必要ないはずでした。しかしApple Siliconターゲットの改良に伴い，__例外的に19 R6はv19.xとv19 Rxシリーズの後継世代として扱われます__。LTSおよび19 R5以前のバージョンでプロジェクトを開くことはできますが，R5/R6を跨ぐたびに再コンパイルが必要となります。

### コンポーネントアーキテクチャー

コンポーネントはプロジェクトモードで開発されており，`cs.NetKit`名前空間でクラスが公開されています。バイナリモードのホストからでもインスタンス化できるように`New OAuth2 provider` `New Office365 provider`という共有プロジェクトメソッドが用意されています。


## トピック

* [ソースは公開されているがフリーウェアではない](https://discuss.4d.com/t/4d-netkit-custom-provider/22350)
* [Googleに接続するためにはどこを書き換えれば良いか](https://discuss.4d.com/t/newfeature-connect-your-mail-transporters-with-an-oauth-2-0-token/17349/16)

## APIアクセス許可

「APIのアクセス許可」ページに移動し，アクセス許可の追加をクリックします。

<img width="880" alt="a" src="https://user-images.githubusercontent.com/1725068/186438396-5af20d85-fa89-4415-970d-0fd988bb5613.png">

Microsoft Graphをクリックします。

<img width="880" alt="g" src="https://user-images.githubusercontent.com/1725068/186438464-79451b07-3383-407b-8e1f-4a0394b41c3e.png">

「委任されたアクセス許可」をクリックします。

<img width="880" alt="d" src="https://user-images.githubusercontent.com/1725068/186438546-24d1d85a-54a9-4915-af9e-e0a59b3044a8.png">

IMAPと入力し，APIを登録します。

<img width="880" alt="imap" src="https://user-images.githubusercontent.com/1725068/186439186-1405bb4a-c594-4097-a8b2-42e57cefd8cd.png">

同じ要領とSMTPとPOPも登録します。

<img width="880" alt="smtp" src="https://user-images.githubusercontent.com/1725068/186439305-49488d0c-391e-487f-802e-ee35066364c3.png">




## 雑記

リダイレクトURIは，[`https:`または`http://localhost`から始まる文字列でなければなりません](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/reply-url)。`http://127.0.0.1:50993/authorize/`と例題には記述されていますが，そのようなURLを登録することができないようになっています。

<img width="398" alt="url" src="https://user-images.githubusercontent.com/1725068/186433291-589ceec7-617d-4739-b96c-19fa587cf8ae.png">

その一方で「リダイレクトURIで`localhost`ではなくIPリテラルループバックアドレス`127.0.0.1`を使用するよう」ドキュメントは推奨しており，対策として（`http:`の場合は）「[アプリケーションマニフェストで`replyUrlsWithType`属性を変更する](https://docs.microsoft.com/ja-jp/azure/active-directory/develop/reference-app-manifest#replyurlswithtype-attribute)」ことが提案されています。つまり，入力画面の制約を乗り越えるため，直接，設定JSONファイルを編集します。

<img width="880" alt="manifest" src="https://user-images.githubusercontent.com/1725068/186435459-e8ee401e-2022-40d5-91fd-256ce6f9c79f.png">

これをせずに`localhost`設定のままIPリテラルループバックアドレス`127.0.0.1`を使用しようとすると，エラーになります。

> AADSTS50011: The redirect URI 'http://127.0.0.1:50993/authorize/' specified in the request does not match the redirect URIs configured for the application 'c92bf2a4-2388-4702-b0f1-4a13ff2fe5ed'. Make sure the redirect URI sent in the request matches one added to your application in the Azure portal. Navigate to https://aka.ms/redirectUriMismatchError to learn more about how to fix this.

## トピック
