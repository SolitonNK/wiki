# Search Control

/ api / searchctrlにあるREST API

searchctrlグループは、アクティブな検索に関する情報を照会し、オプションで何らかのアクションを呼び出すために使用されます。現在のところ、すべてのアクティブな検索のクエリ、特定の検索に関する情報の取得、アクティブな検索のバックグラウンド化、アクティブな検索のアーカイブ、および検索の削除/終了を行うことができます。

## 基本的なAPIの概要

ここでの基本的なアクションは、REST URLでGET、DELETE、またはPATCHを実行することです。

## 検索リストを取得する
検索リストを要求すると、Webサーバーは現在のユーザーが閲覧を許可されているすべての検索を返します。jsonパッケージは投稿されていませんが、空のGETは '/ api / searchctrl /'に対して実行されます。 

```
[
        {
                "ID": "181382061",
                "UID": 1,
                "GID": 0,
                "State": "DORMANT",
                "AttachedClients": 0
        },
        {
                "ID": "010985768",
                "UID": 1,
                "GID": 0,
                "State": "DORMANT",
                "AttachedClients": 0
        },
        {
                "ID": "795927171",
                "UID": 1,
                "GID": 0,
                "State": "DORMANT",
                "AttachedClients": 0
        }
]
```

## 特定の検索の情報を取得する
特定の検索のステータスを取得するには、GET the REST url / api / searchctrl /：IDを実行します。

```
WEB GET /api/searchctrl/795927171:
{
        "ID": "795927171",
        "UID": 1,
        "UserQuery": "grep paco | grep chico",
        "EffectiveQuery": "grep paco | grep chico | text",
        "StartRange": "2016-12-22T12:41:27.011080417-07:00",
        "EndRange": "2016-12-22T13:01:27.011080417-07:00",
        "Started": "2016-12-22T13:01:27.01227455-07:00",
        "Finished": "0001-01-01T00:00:00Z",
        "StoreSize": 0,
        "IndexSize": 0
}
```

## Backgrounding a search

検索のバックグラウンド化は、最後のクライアントが行かせても検索を続行してもよいことをWebサーバーに通知するために使用されます。検索をバックグラウンドで行うには、url / api / searchctrl /：ID / background correct IDでPATCHを実行します。コマンドが成功するには、検索がアクティブであるか、またはすでにバックグラウンドになっている必要があり、ユーザーは管理者であるか、または検索にアクセスできる必要があります。

```
WEB PATCH /api/searchctrl/795927171/background:
null
```

## 検索のバックグラウンド化

検索を保存すると、検索結果を保持したいことをWebサーバーに通知します。ウェブサーバがディスクスペースを必要としない（または明示的に削除されていない）限り、バックグラウンド検索は常駐のままです（誰にも接続されていなくても）。保存すると保存された場所に結果が移動され、（適切な権限を持つ）誰かが明示的にそれを要求しない限り、結果は削除されません。検索を保存するには、url / api / searchctrl /：IDでPATCHを実行し、正しいIDを保存します。検索はどのような状態にあっても構いませんが、休止状態になると永続ストレージへの転送を開始します。永続的ストレージへの転送は、瞬間的（永続的ストレージが同じドライブ上にある場合）またはフルコピーが必要です。これはバックグラウンドで独自のゴルーチンで行われるため、実行中は何もブロックされません。

```
WEB PATCH /api/searchctrl/010985768/save:
null
```

## 検索の削除/終了

検索を削除すると、検索が終了し（アクティブなユーザーがキックオフされ）、検索結果に関連付けられている記憶域が直ちに削除されます。検索はどの状態でも削除できます。検索を削除するには、正しいIDで/ api / searchctrl /：IDへのDELETE要求を実行します。成功した場合は200を返し、エラーがあった場合は5XXを返し、ユーザーが検索の変更を許可されていない場合は403を返します。

```
WEB DELETE /api/searchctrl/010985768:
null
```

## 管理API

管理ユーザーは、上記のAPIエンドポイントを使用して、検索に関する情報の取得、検索の削除、検索の読み込み、検索のバックグラウンドへの送信などを行うことができます。

### すべての検索をリストする

システムに存在するすべての検索のリストを取得するために、管理ユーザーは `/api/searchctrl/all`でGETを実行できます。 形式は `/api/searchctrl`から返されるものと同じですが、システム上のすべての検索が含まれます。

```
[
    {
        "AttachedClients": 0,
        "GID": 0,
        "ID": "486574780",
        "State": "DORMANT",
        "StoredData": 9355,
        "UID": 1
    },
    {
        "AttachedClients": 0,
        "GID": 0,
        "ID": "815623546",
        "State": "DORMANT",
        "StoredData": 3536,
        "UID": 4
    },
    {
        "AttachedClients": 0,
        "GID": 0,
        "ID": "525125903",
        "State": "DORMANT",
        "StoredData": 0,
        "UID": 7
    },
    {
        "AttachedClients": 0,
        "GID": 0,
        "ID": "274379984",
        "State": "DORMANT",
        "StoredData": 319,
        "UID": 4
    }
]

```