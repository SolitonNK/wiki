# Search History
## 検索履歴

/ api / searchhistoryにあるREST API

検索履歴ＡＰＩは、ユーザ、グループ、またはそれらの組み合わせが立ち上げた検索のリストを引き出すために使用される。結果には、誰が検索を開始したか、どのグループが検索を開始したか、検索を表す2つの文字列（ユーザーが実際に入力したもの、バックエンドが処理したもの）などの基本情報が表示されます。

## 基本的なAPIの概要

ここでの基本的なアクションは、/ api / searchhistory / {cmd} / {id}に対してGETを実行することです。cmdは、必要な検索セットを表し、idは、その検索に関連するIDを表します。たとえば、すべての検索をUID 1のユーザーが所有する場合は、/ api / searchhistory / user / 1に対してGETを実行します。すべての検索をGID 4のグループが所有する場合は、/ api /に対してGETを実行します。検索履歴/グループ/ 4。

"all" cmdを使用して、特定のUIDがアクセス権を持つALL検索を要求できます。これは、UIDが所有するすべての検索と、彼がメンバーであるグループが所有するすべての検索を私に提供することを意味します。たとえば、/ api / searchhistory / all / 1に対するGETは、UID 1のユーザーがそれらのグループのメンバーである場合、GID 1、2、3、および4のグループが所有する検索を返すことがあります。返された結果は、JSON形式のSearchLog構造のリストになります。

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