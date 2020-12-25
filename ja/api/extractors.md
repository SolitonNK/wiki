# 自動抽出 API

extractor Web APIは、オートエクストラクタ定義へのアクセス、変更、追加、削除のためのメソッドを提供します。自動抽出器とその設定の詳細については、[Auto-Extractors](/#!configuration/autoextractors.md) のセクションを参照してください。

## 定義の構造
Auto-Extractorsは、以下のJSON構造を使用して定義されています。


```
{
	"tag": string,
	"name": string,
	"desc": string,
	"module": string,
	"params": string,
	"args": string,
	"uid": int32,
	"gids": []int32,
	"global": bool,
	"uuid": string,
	"synced": bool,
	"lastupdated": string```
}
```
## リストアップ

オートエクストラクタのリスト化は、`/api/autoextractors`でGETを実行することで行われます。 ウェブサーバは、現在のユーザーがアクセスできるインストール済みのセットを表すJSON構造体のリストを返します。 レスポンスの例は以下の通りです。
```
[
    {
        "accelerated": "",
        "desc": "csv extractor",
        "gids": null,
        "global": false,
        "lastupdated": "2020-01-08T09:45:29.617936398-07:00",
        "module": "csv",
        "name": "csvextract",
        "params": "col1, col2, col3",
        "synced": false,
        "tag": "csv",
        "uid": 1,
        "uuid": "3674b59e-6064-4cf0-8023-4bf444e84625"
    },
    {
        "accelerated": "",
        "args": "-d \"\\t\"",
        "desc": "Bro conn logs",
        "gids": null,
        "global": true,
        "lastupdated": "2020-01-08T14:45:28.514723502-07:00",
        "module": "fields",
        "name": "bro-conn",
        "params": "ts, uid, orig, orig_port, resp, resp_port, proto, service, duration, orig_bytes, dest_bytes, conn_state, local_orig, local_resp, missed_bytes, history, orig_pkts, orig_ip_pkts, resp_pkts, resp_ip_bytes, tunnel_parents",
        "synced": false,
        "tag": "bro-conn",
        "uid": 1,
        "uuid": "bdb2c2f6-5d50-4222-8558-ecbe3a0822aa"
    }
]
```

adminフラグを設定してGETリクエストを実行すると(`/api/autoextractors?admin=true`)、システム上のすべてのextractorのリストが返されます。

## 追加

自動抽出器の追加は、リクエストボディに有効な定義JSON構造体を指定して `/api/autoextractors` にPOSTを発行することで実行されます。 構造体は有効でなければならず、タグに割り当てられた既存の自動抽出器は存在してはいけません。 新しい自動抽出器を追加するPOST JSON構造体の例：

```
{
	"name": "testCSVExt",
	"desc": "testing extractor",
	"module": "csv",
	"params": "a, b, c, src, dst, extra",
	"tag": "test4"
}
```

自動抽出器を追加する際にエラーが発生した場合、ウェブサーバはエラーのリストを返します。成功した場合、サーバは新しい抽出器のUUIDで応答します。

注意: extractorを作成する際に `uuid`, `uid`, `gids`, `synced`, `lastupdated` フィールドを設定する必要はありません（これらは自動で設定されます）。管理者のみが `global` フラグを true に設定することができます。

## 更新

自動抽出器の更新は、`/api/autoextractors` へのPUTリクエストを発行することで実行されます。 構造体は有効でなければならず、同じUUIDを持つ既存の自動抽出器が存在しなければなりません。 変更されていないフィールドはすべて、サーバから返されたオリジナルのものとして含まれなければなりません。 定義が無効な場合、ボディにエラーメッセージを含む200以外の応答が返されます。 構造体は有効であっても、更新された定義を配布する際にエラーが発生した場合は、ボディにエラーのリストが返されます。

## 抽出器の構文のテスト

自動抽出器を追加したり更新したりする前に、構文を確認しておくと便利かもしれません。POSTリクエストを `/api/autoextractors/test` に実行すると、そのリクエストを検証することができます。定義に問題がある場合は、エラーが返されます。

```
{"Error":"asdf is not a supported engine"}
```

新しい自動抽出器を追加する場合、新しい抽出器が同じタグの既存の抽出と競合しないことが重要です。既存の抽出を更新する場合は、これは気になりません。指定したタグに対して抽出が既に存在する場合、テストAPIは、返された構造体に TagExists フィールドを設定します。

```
{"TagExists":true,"Error":""}
```

`TagExists` が true の場合、新しい抽出器を作成しようとしている場合はエラーとして扱われ、既存の抽出器を更新しようとしている場合は無視されるべきです。

## ファイルのアップロード

Autoextractorの定義はTOML形式で表現することができます。この形式は人間が読めるものであり、抽出器の定義を配布するのに便利な方法です。以下に例を示します。

```
[[extraction]]
	tag="bro-conn"
	name="bro-conn"
	desc="Bro conn logs"
	module="fields"
	args='-d "\t"'
	params="ts, uid, orig, orig_port, resp, resp_port, proto, service, duration, orig_bytes, dest_bytes, conn_state, local_orig, local_resp, missed_bytes, history, orig_pkts, orig_ip_pkts, resp_pkts, resp_ip_bytes, tunnel_parents"
```

このファイルを解析してJSON構造体を生成するのではなく、このタイプの定義は `/api/autoextractors/upload` へのPOSTリクエストで送られるマルチパートフォームを使って直接ウェブサーバにアップロードすることができます。このフォームには `extraction` という名前のファイルフィールドが含まれていなければなりません。定義が有効でインストールに成功した場合、サーバは200のレスポンスを返します。

## ファイルのダウンロード

自動抽出器の定義をTOML形式でダウンロードするには、`/api/autoextractors/download`にGETリクエストを発行します。ダウンロードしたい定義ごとに、URLにUUIDをパラメータとして追加します。したがって、UUID ad782c81-7a60-4d5f-acbf-83f70e68ecb0 と c7389f9b-ba52-4cbe-b883-621d577c6bcc の 2 つのエクストラクタをダウンロードしたい場合は、次のようになります：GETリクエストを `/api/autoextractors/download?id=ad782c81-7a60-4d5f-acbf-83f70e68ecb0&id=c7389f9b-ba52-4cbe-b883-621d577c6bcc` に送信します。

現在のユーザーがすべての指定された抽出器へのアクセス権を持っている場合、サーバーはTOML形式の定義を含むダウンロード可能なファイルで応答します。このファイルは、上記のファイルアップロードAPIを使用して、別のGravwellシステムにアップロードすることができます。

## 削除

既存の自動抽出器を削除するには、`/api/autoextractors/{uuid}` に DELETE リクエストを発行します。ここで `uuid` は自動抽出器に関連付けられたUUIDです。自動抽出器が存在しない場合や、自動抽出器の削除にエラーが発生した場合、ウェブサーバは200以外のレスポンスとレスポンスボディのエラーで応答します。

## モジュールのリストアップ

自動抽出器の定義では、有効なモジュールを指定する必要があります。 サポートされているモジュールのリストを取得するAPIは、`/api/autoextractors/engines`にGETリクエストを発行することで実行されます。 結果として得られるのは文字列のリストです。

```
[
	"fields",
	"csv",
	"slice",
	"regex"
]
```
