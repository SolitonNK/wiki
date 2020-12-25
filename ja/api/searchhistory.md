# 検索履歴

REST APIは`/api/searchhistory`にある。

検索履歴APIを使用して、あるユーザー、グループ、またはその組み合わせが起動した検索のリストを引き出すことができます。結果は、誰が検索を開始したのか、どのグループがその検索を所有しているのか、いつ開始されたのか、検索を表す2つの文字列 (ユーザーが実際に入力したものとバックエンドが処理したもの) などの基本的な情報を提供します。

## 基本APIの概要

ここでの基本的な動作は、`/api/searchhistory/{cmd}/{id}`に対して、`cmd`は検索のセットを、`id`はその検索に関連するIDを表し、`/api/searchhistory/{cmd}/{id}`をGETすることです。 例えば、UID 1 のユーザが所有する全ての検索が欲しい場合は `/api/searchhistory/user/1` で GET を実行し、GID 4 のグループが所有する全ての検索が欲しい場合は `/api/searchhistory/group/4` で GET を実行します。単に `/api/searchhistory/user/1` をGETするだけで、現在のユーザの履歴を返すことができます。

「all」コマンドを使って、特定のUIDがアクセスできるすべての検索を要求することができます。これは、そのUIDが所有するすべての検索と、そのUIDが所属しているグループが所有するすべての検索を要求することを意味しています。例えば、`/api/searchhistory/all/1`をGETすると、UID 1のユーザがこれらのグループのメンバーであれば、GID 1,2,3,4のグループが所有する検索を返すことができます。返される結果は、JSON形式のSearchLog構造体のリストになります。

JSONの例:
```
[
        {
                "UID": 1,
                "GID": 2,
                "UserQuery": "grep stuff",
                "EffectiveQuery": "grep stuff | text",
                "Launched": "2015-12-30T23:30:23.298945825-07:00"
        },
        {
                "UID": 1,
                "GID": 2,
                "UserQuery": "grep stuff | grep things | grep that | grep this | sort by time",
                "EffectiveQuery": "grep stuff | grep things | grep that | grep this | sort by time | text",
                "Launched": "2015-12-30T23:31:08.237520376-07:00"
        }
]
```

### 管理者のクエリ

adminユーザとして`/api/searchhistory?admin=true`へのGETリクエストは、すべてのユーザの検索結果を返します。