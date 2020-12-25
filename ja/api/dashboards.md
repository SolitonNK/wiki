# ダッシュボード・ストレージ API

ダッシュボードAPIは、基本的にはGUIがダッシュボードをレンダリングするために使用するjson blobを管理するための一般的なCRUD APIです。ダッシュボード上に存在する検索は GUI によって起動され、バックエンド/フロントエンド/ウェブサーバはそれらが何であるかの概念を持っていません。

## ダッシュボードの作成

ダッシュボードを追加するには、 **/api/dashboards** に対して以下のフォーマットのペイロードを持つ **POST** リクエストが発行されます。
Dataプロパティは、実際のダッシュボードを作成するためにGUIが使用するJSONです。

```
{
        "Name": "test2",
        "Description": "test2 description",
				"UID": 2,
				"GIDs": [],
		"Data": {
				"A": "A2",
				"B": "B2",
				"C": 12610078956637388,
				"D": false
		}
}
```
リクエストから「UID」パラメータが省略された場合、リクエストしたユーザのUIDがデフォルトとなるはずです。
ウェブサーバのレスポンスには、新しく作成されたダッシュボードのIDが含まれています。

## ダッシュボードの取得

### 現在のユーザのすべてのダッシュボードの取得

ユーザーは、 **/api/dashboards** で **GET** リクエストを発行することで、（所有者またはグループ経由で）アクセス権を持つすべてのダッシュボードを取得することができます。

```
[
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
                "Created": "2016-12-18T23:28:08.250051418Z",
				"GUID": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]


```

### 特定のダッシュボードの取得
特定のIDを取得するには、ダッシュボードのURLの末尾にそのIDを記述します。

```	## Listing Dashboards
GET /api/dashboards/2:
```

また、IDではなくGUIDで特定のダッシュボードを取得することも可能です: `GET /api/dashboards/d28b6887-ad55-479e-8af3-0cbcbd5084b1`.

### 管理者が全ユーザーのダッシュボードを取得

すべてのダッシュボードを取得するには、ユーザーは管理者でなければならず、 **/api/dashboards/all** に **GET** リクエストを発行しなければなりません。このリクエストが非管理者ユーザーによって発行された場合、そのユーザーがアクセスできるすべてのダッシュボードを返す必要があります (これは **/api/dashboards** への GET と同じ意味になります)。

```
WEB GET /api/dashboards/all:
[
       {
                "ID": 1,
                "Name": "test1",
                "UID": 1,
                "GIDs": [],
                "Description": "test1 description",
                "Created": "2016-12-18T23:28:07.679322121Z",
				"GUID": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
                "Data": {
                        "A": "A",
                        "B": "B",
                        "C": 57646075230342348,
                        "D": true
                }
        },
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
                "Created": "2016-12-18T23:28:08.250051418Z",
								"GUID": "55bc7236-39e4-11e9-94e9-54e1ad7c66cf",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }

]	]
```

## ダッシュボードの更新
ダッシュボードの更新(データを変更したり、名前、説明、GIDリストなどを変更する)は、 **/api/dashboards/ID** に **PUT** リクエストを発行することで行われます。
この例では、ユーザー(UID 3)がグループ3に対してダッシュボード(ID 2)にアクセスする権限を追加したいと考えています。

```
GET /api/dashboards/2:
[
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
							   "GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Created": "2016-12-18T23:28:08.250051418Z",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```

これで、ユーザーはターゲット・ダッシュボードを取得し、修正を行い、PUTリクエストを投稿しました。
```
WEB PUT /api/dashboards/2:
[
        {
                "ID": 2,
                "Name": "marketoverview",
                "UID": 3,
                "GIDs": [3],
                "Description": "marketing group dashboard",
				"GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Created": "2016-12-18T23:28:08.250051418Z",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```

サーバーは、更新されたダッシュボードの構造で更新要求に応答します。

注：必要に応じて、GUID をダッシュボード ID の代わりに使用することができます。

## ダッシュボードの削除
ダッシュボードを削除するには、URL **/api/dashboards/ID** に **DELETE** メソッドを指定してリクエストを発行します。

## ユーザーが所有するすべてのダッシュボードの取得
ユーザーが明示的に所有するダッシュボードを取得するには、 **/api/users/UID/dashboards** に **GET** リクエストを発行してください。 これには、ユーザーがグループ・メンバーシップを通じてアクセスできるダッシュボードは含まれません。
```
WEB GET /api/users/1/dashboards:
[
                {
                "ID": 4,
                "Name": "dashGroup2",
                "UID": 5,
                "GIDs": [
                        3
                ],
                "Description": "dashGroup2",
                "Created": "2016-12-28T21:37:12.703358455Z",
								"GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Data": {
                        "A": "A2",	      "timeframe": {
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```

注：必要に応じて、GUID をダッシュボード ID の代わりに使用することができます。

## すべてのグループのダッシュボードを取得する
特定のグループがアクセスできるすべてのダッシュボードを取得するには、 **/api/groups/GID/dashboards** に **GET** リクエストを発行します。 これは、ダッシュボードが複数のグループにいることができるという点で、通常のUnixのパーミッションから少し逸脱しています（つまり、グループはダッシュボードを所有しているのではなく、ダッシュボードはグループのメンバーのようなものです）。

```
WEB GET /api/groups/2/dashboards:
[
        {
                "ID": 2,
                "Name": "marketoverview",
                "UID": 3,
                "GIDs": [3],
                "Description": "marketing group dashboard",
				"GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Created": "2016-12-18T23:28:08.250051418Z",
                                "Data": {"5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
  				"Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        },
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
                "Created": "2016-12-18T23:28:08.250051418Z",
								"GUID": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",	 
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```

### 管理者が全ユーザーのダッシュボードを取得
すべてのダッシュボードを取得するには、ユーザーは管理者でなければならず、 **/api/dashboards/all** に **GET** リクエストを発行しなければなりません。このリクエストが非管理者ユーザーによって発行された場合、そのユーザーがアクセスできるすべてのダッシュボードを返す必要があります (これは **/api/dashboards** への GET と同じ意味になります)。

```
WEB GET /api/dashboards/all:
[
        {
                "ID": 1,
                "Name": "test1",
                "UID": 1,
                "GIDs": [],
                "Description": "test1 description",
                "Created": "2016-12-18T23:28:07.679322121Z",
								"GUID": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
                "Data": {
                        "A": "A",
                        "B": "B",
                        "C": 57646075230342348,
                        "D": true
                }
        },
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
                "Created": "2016-12-18T23:28:08.250051418Z",
								"GUID": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```
