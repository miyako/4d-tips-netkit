# 4d-tips-netkit
4D NetKitまとめ

## 基本情報

#### 対応バージョン

**4D NetKit**は，4Dアプリの *Components* フォルダーに標準で組み込まれています。v19 LTS（19.x）には組み込まれていません。ただし，19 R5以前のバージョンであれば，フィーチャーリリースに限定された新しい機能を使用しているわけではないため，プロジェクトの *Components* フォルダーにインストールすれば，互換レベルで動作するようです。19 R6では `4D.HTTPRequest` クラスが採用され，LTSおよび19 R5以前のバージョンでは実行時にエラーが返されるようになりました。ただし，新しいコマンドを使用しているわけではなく，`4D`クラスストアの新しいプロパティをコールしているため，シンタックスチェックとコンパイルはいずれも成功します。

**注記**: 本来，v19.xとv19 Rxは，同一のメジャーバージョン番号に属するため，v19 R6はメジャーアップグレードに該当せず，再コンパイルも必要ないはずでした。しかしApple Siliconターゲットの改良に伴い，__例外的に19 R6はv19.xとv19 Rxシリーズの後継世代として扱われます__。LTSおよび19 R5以前のバージョンでプロジェクトを開くことはできますが，R5/R6を跨ぐたびに再コンパイルが必要となります。
