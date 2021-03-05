# 検索のwebsocket

websocketのURL: /api/ws/search

このページでは、検索のためのwebsocketプロトコルについて説明します。"grep foo"の検索を初期化し、エントリーデータの検索を実施する際にクライアントとサーバ間でやりとりされるJSONデータの全容例は、 [Websocket検索の例](websocket-search-example.md)ページで見ることができます。

## PingとPongによるkeepalive

検索websocketは、クエリのチェック、検索の送信、検索結果と検索統計の受信に使用されます。 検索websocketについて、メッセージの"タイプ"(Type)を処理するためにはRoutingWebsocket システムを使用することが想定されています。`/api/ws/search` は、起動時に以下のメッセージのような"サブタイプ"(SubType)または"タイプ"(Type)の登録されていることを想定しています。：PONG, parse, search, attach

注：メッセージの"タイプ"(Type)は、旧式な名前として"SubProto"と呼ばれることがあります。これは今後変更された方が良いのですが、このAPIを使用して開発する場合、"SubProto"は、メッセージであくまで送信される"タイプ"(Type)の値を指しており、RFC Websocketサブプロトコル仕様のことではないことに注意してください。

PONGタイプはキープアライブシステムであり、クライアントは定期的にPINGPONGリクエストを送信する必要があります。

これは、ユーザが検索プロンプトか何かに向かって取り組んでいるときに、その裏側で接続が正常に保たれてるかどうかをユーザに伝えるために使うことができます。Websocket 自体が生きているかどうかを調べるだけなので、ユーザがそれをする時には全く必要ないかもしれません。

## 検索の"parse"

"parse"のタイプは、検索バックエンドを実際に呼び出すことなく、クエリが適正かどうかを手早く確認するために使用されます。

適正なクエリのリクエストとレスポンスの場合では、次のような JSON が得られます:

フロントエンドからのリクエスト:
```json
{
        SearchString: "tag=apache grep firefox | regex "Firefox(<version>[0-9]+) .+" | count by version""
}
```

バックエンドからのレスポンス:
```
{
        GoodQuery: true,
        ParseQuery: "tag=apache grep firefox | regex "Firefox(<version>[0-9]+) .+" | count by version"",
        ModuleIndex: 0,
}
```

不適正なクエリのリクエストとレスポンスの場合では、次のような JSON が得られます:

フロントエンドからのリクエスト:
```
{
        SearchString: "tag=apache grep firefox | MakeRainbows",
}
```

バックエンドからのレスポンス:
```
{
        GoodQuery: false,
        ParseError: "ModuleError: MakeRainbows is not a valid module",
        ModuleIndex: 1,
}
```

## 検索の初期化
すべての検索はウェブソケットを介して初期化され、その開始時に "parse"、"PONG"、"search"、"attach "のサブタイプを指定することが必須です。 

この手続きは、websocket の確立時に以下の JSON を送信することで行われます:
```
{"Subs":["PONG","parse","search","attach"]}
```

SearchString メンバには、検索を呼び出す実際のクエリが含まれていなければなりません。

SearchStartとSearchEndには、クエリが動作する時間範囲を指定する必要があります。 範囲指定の時刻は、"2006-01-02T15:04:05.9999999Z07:00 "のようなRFC3339Nano形式の書式です。

適正なクエリを持つ検索リクエストの例としては、次のような JSON があげられます:
```
{
       SearchString: "tag=apache grep firefox | nosort",
       SearchStart:  "2015-01-01T12:01:00.0Z07:00",
       SearchEnd:    "2015-01-01T12:01:30.0Z07:00",
       Background:   false,
}
```

//検索がクールであれば、サーバはYay/Nayと新しいサブタイプに応答します。
//searchStart と searchEnd はRFC3339Nano 形式の文字列で指定されます

適切なクエリに対するレスポンスは例えば次のようなJSONになります:
```
{
        SearchString: "tag=apache grep firefox | nosort",
        RenderModule: "text",
        RenderCmd:    "text",
        OutputSearchSubproto:  "searchSDF8973",
        OutputStatsSubproto:   "statsSDF8973",
        SearchID:              "skdlfjs9098",
		SearchStartRange:      "2015-01-01T12:01:00.0Z07:00",
        SearchEndRange:        "2015-01-01T12:01:30.0Z07:00",
        Background:            false,
}
```

エラーのレスポンスのJSONは次のようになります:
```
{
        Error: "Search error: The parameter "ChuckTesta" is invalid",
}
```

適正な検索リクエストへのレスポンスでは、クライアントは検索ACKを付してレスポンスします。Ack レスポンスは true か false のどちらかです。 false レスポンスは、フロントエンドが理解できないレンダリングモジュールをバックエンドがリクエストした場合に使用されます。これは、例えば、フロントエンドとバックエンドの間でバージョン不一致がある場合に起こります。

次の JSON は、前の応答例と対になる、true のACK の例を表します:
```
{
       Ok: True,
       OutputSearchSubproto: "searchSDF8973"
}
```

ACK が送信されると、バックエンドは検索を開始し、新しいサブタイプの検索結果の提供を開始します。 元の"search"、"parse", "PONG"のサブタイプはアクティブなままで、フロントエンドは新しいクエリをチェックしたり、追加の検索を開始したりするために使用することができます。 しかし、アクティブなクエリとのやりとりはすべて、新しくネゴシエートされた検索固有のサブタイプを介して行われる必要があります。

## 注意
すべての検索は完全に非同期ですが、検索をバックグラウンド状態にすることをリクエストしないままクライアントが切断されたり、接続がクラッシュしたりした場合、アクティブな検索は終了し、データはゴミ箱行きになります。 これは、リソースの枯渇を防ぐためです。 ユーザーはバックグラウンド検索することを明示的に要求しなければなりません。

複数の利用者が一つの検索を活用することができます。 例えば、ボブが始めた検索について、ジャネットがアタッチして結果を見ることができます。 バックグラウンドではない検索は、すべての利用者から切断された場合にのみ終了し、クリーンアップされます。 つまり、ボブが検索を開始してジャネットがアタッチしている時、ボブがブラウザを閉じたり移動したりした場合も、検索は終了しません。 ジャネットはその検索での作業を続けることができます。 しかし、もしさらにジャネットが移動したりブラウザを閉じたりした場合は、検索は終了し、ゴミ扱いされることになります。

## アクティブな検索中に出力される統計情報

統計情報は、統計情報の ID を使ってリクエストされます。

## リクエスト/レスポンスID の参照

リクエストとレスポンスのIDコードの一覧:
```
{
    req: {
        REQ_CLOSE: 0x1,
        REQ_ENTRY_COUNT: 0x3,
        REQ_DETAILS: 0x4,
        REQ_TAGS: 0x5,
        REQ_STATS_SIZE: 0x7F000001, //gets backend "size" value of stats chunks. never used
        REQ_STATS_RANGE: 0x7F000002, //gets current time range covered by stats. rarely used
        REQ_STATS_GET: 0x7F000003, //gets stats sets over all time. may be used initially
        REQ_STATS_GET_RANGE: 0x7F000004, //gets stats in a specific range
        REQ_STATS_GET_SUMMARY: 0x7F000005, //gets stats summary for entire results
        REQ_STATS_GET_LOCATION: 0x7F000006, //get current timestamp for search progress
        REQ_GET_ENTRIES: 0x10, //1048578
        REQ_STREAMING: 0x11,
        REQ_TS_RANGE: 0x12,
		REQ_GET_EXPLORE_ENTRIES: 0xf010,
		REQ_EXPLORE_TS_RANGE: 0xf012,
        SEARCH_CTRL_CMD_DELETE: 'delete',
        SEARCH_CTRL_CMD_ARCHIVE: 'archive',
        SEARCH_CTRL_CMD_BACKGROUND: 'background',
        SEARCH_CTRL_CMD_STATUS: 'status'
    },
    rep: {
        RESP_CLOSE: 0x1,
        RESP_ENTRY_COUNT: 0x3,
        RESP_DETAILS: 0x4,
        RESP_TAGS: 0x5,
        RESP_STATS_SIZE: 0x7F000001, //2130706433
        RESP_STATS_RANGE: 0x7F000002, //2130706434
        RESP_STATS_GET: 0x7F000003, //2130706435
        RESP_STATS_GET_RANGE: 0x7F000004, //2130706436
        RESP_STATS_GET_SUMMARY: 0x7F000005,
        RESP_STATS_GET_LOCATION: 0x7F000006, //2130706438
        RESP_GET_ENTRIES: 0x10,
        RESP_STREAMING: 0x11,
        RESP_TS_RANGE: 0x12,
		RESP_GET_EXPLORE_ENTRIES: 0xf010,
		RESP_EXPLORE_TS_RANGE: 0xf012,
        RESP_ERROR: 0xFFFFFFFF,
        SEARCH_CTRL_CMD_DELETE: 'delete',
        SEARCH_CTRL_CMD_ARCHIVE: 'archive',
        SEARCH_CTRL_CMD_BACKGROUND: 'background',
        SEARCH_CTRL_CMD_STATUS: 'status'
    }
}
```
