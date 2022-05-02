Sourcetreeでgitのパスワードを変更した時

gitのクライアントアプリとして大変便利な「Soucetree」ですが、リモートサーバーのパスワードを変更した後、パスワードの設定を変えるのがすごく大変！
セキュリティ対策でパスワードを年に何回か変えることがあるかと思いますが、そのたびにネットで検索するのはどうにも。

それよりも、ネットで英語サイトを含めて調べてもなかなか解決策が出てこない。

パスワードを変更した場合、リモートサーバーのパスワードを変更すると、プッシュなどアクションを起こした時にパスワード入力画面が出てくると説明している記事がありますが、出てきません。

今回表示されたログは下記の通り

git -c diff.mnemonicprefix=false -c core.quotepath=false fetch --prune origin
Access denied
Access denied
Access denied
Access denied
FATAL ERROR: Server sent disconnect message
type 2 (protocol error):
"Too many authentication failures for XXXXXXXX"
fatal: The remote end hung up unexpectedly
Completed with errors, see above.
 Windowsでの対策
 
C://Users/NAME/AppData/Local/Atlassian/SourceTree
※NAMEはそれぞれの環境により異なります。

上記にアクセスして「passwd」の中身を消す。もしくはファイルごと消してみてください。

その後、リモートサーバーに対してアクションを起こすとパスワード入力画面が表示されます。

※参考サイトはこちら

 

Macでの対策
こちらは簡単です。「KeyChainAccess」に保存されているSourcetreeのパスワードを削除してください。

その後、リモートサーバーに対してアクションを起こすとパスワード入力画面が表示されます。

 

☆ご注意ください
ファイルを消す手順があります。環境により異なる場合があります。あくまで自己責任にてお願いします。