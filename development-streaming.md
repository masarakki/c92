## Streaming

マストドンでリアルタイム更新されるデータは、すべてWebsocketによって送られてきます。
Websocketの接続を受け付けるのは、専用のNode.jsアプリケーションであるストリーミングサーバ(`streaming/index.js`) です。

### アーキテクチャ

マストドンでは、タイムラインのタブそれぞれが1本のWebsocketのコネクションを張ります。
ストリーミングサーバは、受け取ったタイムラインの属性に合致した新規トゥートを送り続けます。

とはいえ、ストリーミングサーバでSQLを叩いているわけではありません。
ストリーミングサーバは、Redisを購読(`subscribe`)し、Redisに追加されたデータを各クライアント送る、というとてもシンプルな作りになっています。

タイムラインは種類によってそれぞれキーが設定されています。
例えば、ローカルタイムラインは全員同じ `timeline:public:local` というキーを購読し、ホームタイムラインはユーザごとに `timeline:{account_id}` のキーを購読します。

となると、処理のキモはRedisにデータを追加しているところになります。
それはすべてRails側の仕事になっています。
前述の `FanOutOnWriteService` の中には、様々な `deliver_to_xxx` がありましたが、
このdeliverメソッドの内部で `FeedManager` や `FeedInsertWorker` を使ってRedisにデータを追加しています。

### カスタマイズのポイント

表示条件を変えるなどの単純なカスタマイズをするなら、既存のRailsコードを弄るだけで変更できるでしょう。

新規タイムラインを追加するなどの重めのカスタマイズの場合、Redisのキーを考え、Rails側の実装、Stremingの実装、フロントエンドの実装すべて漏れ無くしなくてはなりません。
既存のコードを参考にはできると思いますが、特にRails側が難しくなるのではないかと思います。

さらにタイムラインではない何か、例えば管理者からのお知らせを表示したいとか、メタ情報を動的に変更したいとか、強制リロードさせる制御命令を送りたいとか、そういうものを実装してみても面白いかもしません。