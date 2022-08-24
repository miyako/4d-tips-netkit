# 4d-tips-netkit
4D NetKitまとめ

## 基本情報

#### [4D NetKit](https://github.com/4d/4D-NetKit)とは

サードパーティ製Webサービスと連携することを支援するためのライブラリ。汎用的な設計を銘打っており，ソースコードも公開されていますが，v19 R6の時点でサポートされているプロトコルは，**OAuth 2.0 API**のみ，サポートされているサードパーティ製Webサービスは[**Microsoft Graph API**](https://docs.microsoft.com/en-us/graph/overview)のみ，となっています。

#### トピック

* [ソース公開されているがフリーウェアではない](https://discuss.4d.com/t/4d-netkit-custom-provider/22350)
* [Googleに接続するためにはどこを書き換えれば良いか](https://discuss.4d.com/t/newfeature-connect-your-mail-transporters-with-an-oauth-2-0-token/17349/16)

* 19 R3 - [OAuth2Provider](https://blog.4d.com/4d-and-microsoft-365-access/)
* 19 R4
* 19 R5
* 19 R6 - [Office365](https://blog.4d.com/microsoft-365-send-emails/)

GitHubのリポジトリにあるのは， *master* および *19RX* というブランチだけです。毎回のフィーチャーリリースで19R3, 19R4, 19R5…といったブランチやスナップショットが作成されるわけではありません。

#### 対応バージョン

**4D NetKit**は，4Dアプリの *Components* フォルダーに標準で組み込まれています。v19 LTS（19.x）には組み込まれていません。ただし，19 R5以前のバージョンであれば，フィーチャーリリースに限定された新しい機能を使用しているわけではないため，プロジェクトの *Components* フォルダーにインストールすれば，互換レベルで動作するようです。19 R6では `4D.HTTPRequest` クラスが採用され，LTSおよび19 R5以前のバージョンでは実行時にエラーが返されるようになりました。ただし，新しいコマンドを使用しているわけではなく，`4D`クラスストアの新しいプロパティをコールしているため，シンタックスチェックとコンパイルはいずれも成功します。

**注記**: 本来，v19.xとv19 Rxは，同一のメジャーバージョン番号に属するため，v19 R6はメジャーアップグレードに該当せず，再コンパイルも必要ないはずでした。しかしApple Siliconターゲットの改良に伴い，__例外的に19 R6はv19.xとv19 Rxシリーズの後継世代として扱われます__。LTSおよび19 R5以前のバージョンでプロジェクトを開くことはできますが，R5/R6を跨ぐたびに再コンパイルが必要となります。
