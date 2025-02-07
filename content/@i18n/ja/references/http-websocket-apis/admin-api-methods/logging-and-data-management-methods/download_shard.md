---
html: download_shard.html
parent: logging-and-data-management-methods.html
blurb: レジャー履歴の特定のシャードをダウンロードします。
labels:
  - データ保持
---
# download_shard
[[ソース]](https://github.com/XRPLF/rippled/blob/master/src/ripple/rpc/handlers/DownloadShard.cpp "Source")

サーバーに対し、外部ソースから特定の[履歴レジャーデータのシャード](history-sharding.html)をダウンロードするように指示します。`rippled`サーバーで[履歴シャードが保管されるように設定する](configure-history-sharding.html)必要があります。[新規: rippled 1.1.0][]

_`download_shard`メソッドは、権限のないユーザーは実行できない[管理メソッド](admin-api-methods.html)です。_

外部ソースからHTTPSを使用してシャードが[lz4圧縮](https://lz4.github.io/lz4/) [tarアーカイブ](https://en.wikipedia.org/wiki/Tar_(computing))として提供される必要があります。アーカイブには、NuDB形式のシャードディレクトリとデータファイルが含まれている必要があります。

通常、このメソッドを使用してシャードをダウンロードしてインポートすれば、ピアツーピアネットワークからシャードを個別に取得するよりも短い時間で取得できます。また、サーバーから提供される特定範囲のシャードまたはシャードのセットを選択する場合にもこのメソッドを使用できます。

### 要求フォーマット

要求フォーマットの例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "command": "download_shard",
  "shards": [
    {"index": 1, "url": "https://example.com/1.tar.lz4"},
    {"index": 2, "url": "https://example.com/2.tar.lz4"},
    {"index": 5, "url": "https://example.com/5.tar.lz4"}
  ]
}
```

*JSON-RPC*

```json
{
  "method": "download_shard",
  "params": [
    {
      "shards": [
        {"index": 1, "url": "https://example.com/1.tar.lz4"},
        {"index": 2, "url": "https://example.com/2.tar.lz4"},
        {"index": 5, "url": "https://example.com/5.tar.lz4"}
      ]
    }
  ]
}
```

<!-- MULTICODE_BLOCK_END -->


要求には以下のフィールドが含まれます。

| `Field`    | 型      | 説明                                                  |
|:-----------|:--------|:------------------------------------------------------|
| `shards` | 配列 | ダウンロードするシャードとダウンロード元を記述したShard Descriptorオブジェクト（以下の説明を参照）のリスト。 |

`validate`のフィールドは廃止予定であり、今後予告なしに削除される可能性があります。`rippled`は全てのシャードの検証を実行します。[更新: rippled 1.6.0][]

`shards`配列の各**Shard Descriptorオブジェクト**には以下のフィールドが含まれています。

| `Field` | 型     | 説明                                                      |
|:--------|:-------|:----------------------------------------------------------|
| `index` | 数値 | 取得するシャードのインデックス。本番環境のXRP Ledgerでは、最も古いシャードのインデックスは1であり、このシャードにはレジャー32750～32768が含まれています。次のシャードのインデックスは2であり、このシャードにはレジャー32769～49152が含まれています。 |
| `url` | 文字列 | このシャードをダウンロードできるURL。このURLは`https://`か`http://`かで始まり`.tar.lz4`（大文字小文字の区別なし）で終わる必要があります。このダウンロードを提供するWebサーバーは、信頼できる認証局（CA）によって署名された有効なTLS証明書を使用する必要があります。（`rippled`はオペレーティングシステムのCAストアーを使用します。） [更新: rippled 1.7.0][] |

### 応答フォーマット

処理が成功した応答の例:

<!-- MULTICODE_BLOCK_START -->

*WebSocket*

```json
{
  "result": {
    "message": "downloading shards 1-2,5"
  },
  "status": "success",
  "type": "response"
}
```


*JSON-RPC*

```json
200 OK

{
  "result": {
    "message": "downloading shards 1-2,5",
    "status": "success"
  }
}
```


<!-- MULTICODE_BLOCK_END -->

この応答は[標準フォーマット][]に従っており、正常に完了した場合は結果に次のフィールドが含まれます。

| `Field`   | 型     | 説明                                                    |
|:----------|:-------|:--------------------------------------------------------|
| `message` | 文字列 | この要求に対応して実行されたアクションを説明するメッセージ。 |

**ヒント:** サーバーで使用可能なシャードを確認するには、[crawl_shardsメソッド[]を使用します。または、シャードストアーとして設定されたロケーションのサブフォルダー（`rippled.cfg`の`[shard_db]`の`path`パラメーター）を調べます。フォルダーには、シャードの番号に対応する名前が付いています。これらのフォルダーの1つに、シャードが未完了であることを示す`control.txt`ファイルが含まれていることがあります。

### 考えられるエラー

- いずれかの[汎用エラータイプ][]。
- `notEnabled` - サーバーでシャードストアーを使用するように設定されていません。
- `tooBusy` - サーバーはすでに、ピアツーピアネットワークから、または以前の`download_shard`要求の結果として、シャードをダウンロード中です。
- `invalidParams` - 要求で1つ以上の必須フィールドが省略されていたか、または指定されたフィールドのデータタイプが誤っています。

<!--{# common link defs #}-->
{% include '_snippets/rippled-api-links.md' %}
{% include '_snippets/tx-type-links.md' %}
{% include '_snippets/rippled_versions.md' %}
