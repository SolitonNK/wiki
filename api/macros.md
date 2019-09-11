# Macros

Web APIは検索マクロにアクセスして作成するためのメソッドを提供します。検索マクロは、検索の解析段階で展開される短い文字列から長い文字列へのマッピングです。

## SearchMacro構造体

WebサーバーはJSON構造体でマクロを返します。これはフィールドの更新にも使用されます。マクロを更新するために構造体を送信すると、すべてのフィールドが更新されるわけではありません（たとえば、LastUpdatedフィールドを手動で変更することは不可能です）。これらのフィールドは大部分は一目瞭然ですが、正確を期すためにここで説明します。

* ID: このマクロを表す一意の整数
* UID: マクロ所有者の整数のUID
* GIDs: マクロへのアクセスが許可されている整数グループIDのリスト
* Name: 検索クエリで入力されたマクロの短い名前
* Expansion: マクロが展開する文字列
* LastUpdated: このマクロが最後に変更された時刻
* Synced:（内部使用のみ）

## マクロの一覧表示

現在のユーザーに属するすべてのマクロのリストを取得するには、GETを実行してください/api/macros。結果は次のようになります。

```
[{"ID":1,"UID":1,"GIDs":null,"Name":"FOO","Expansion":"grep foo","LastUpdated":"2018-10-31T20:56:24.629561628Z","Synced":true}]
```

この例では、UID 1のユーザは、文字列「grep foo」に展開される「FOO」という名前の1つのマクロを持っています。マクロIDは1です。これは他のAPIで使用できます。

### ユーザーIDによるマクロの取得

管理ユーザーは、GETを実行して目的のユーザーIDに/api/users/{uid}/macros置き換え{uid}て、特定のユーザーのマクロのリストを取得できます。管理者以外のユーザーはこのAPIを使用して自分のマクロを取得できます。

### グループIDでマクロを取得する

管理ユーザーまたはグループのメンバーは、/ api / groups / ｛gid｝ / macrosに対してGETを実行することによって、指定されたグループがアクセス権を持つマクロのリストを取得できます。

### すべてのマクロを取得する

管理ユーザーは、/ api / macros / allに対してGETを実行することによって、システム上のすべてのマクロのリストを取得できます。

## 特定のマクロを取得する

特定のマクロの構造は、/ api / macros / {id}に対してGETを実行し、{id}をマクロIDに置き換えることで取得できます。 たとえば、/ api / macros / 1に対するGETは次のようになります。

```
{"ID":1,"UID":1,"GIDs":null,"Name":"FOO","Expansion":"grep foo","LastUpdated":"2018-10-31T20:56:24.629561628Z","Synced":true}
```

## 新しいマクロを作成する

新しいマクロはPOSTを介して作成することができます/api/macros。以下に示すように、要求の本文には、名前フィールドと拡張フィールド、および（オプションで）GIDフィールドを含める必要があります。

```
{"GIDs": [1, 2], "Name": "TEST", "Expansion": "grep test | count"}
```

A successful creation will return the ID of the new macro; in this example the new ID was 2, so a GET on `/api/macros/2` yields the full body of the new macro:

```
{"ID":2,"UID":1,"GIDs":null,"Name":"TEST","Expansion":"grep test | count","LastUpdated":"2018-10-31T21:05:34.231426316Z","Synced":true}
```

## マクロを更新する

マクロを更新することは/api/macros/{id}、作成と同じ本体フォーマットを使用するようにPUTによって行われます。安全のために、何も変更しなくても毎回GID、Name、Expanionの各フィールドに入力するのが最善です。

更新が成功すると、HTTP 200と更新されたマクロが本文に返されます。

## マクロを削除する
マクロの削除は、DELETE onによって実行され/api/macros/{id}ます。成功するとHTTP 200が返されます。