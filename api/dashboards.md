# Dashboard Storage API

BLOBを管理するための一般的なCRUD APIです。ダッシュボードに表示される検索はGUIによって起動され、バックエンド/フロントエンド/ Webサーバーには実際にはそれらの概念がありません。

## ダッシュボードを作成する

ダッシュボードを追加するには、POST要求を/api/dashboardsに発行し、ペイロードを次の形式にします。

'Data'プロパティは、実際のダッシュボードを作成するためにGUIによって使用されるJSONです。

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

"UID"パラメータが要求から省略されている場合は、要求元ユーザのUIDがデフォルトになります。

Webサーバーの応答には、新しく作成されたダッシュボードのIDが含まれています。

## ダッシュボードの取得

### 現在のユーザーのすべてのダッシュボードを取得する

ユーザーは、/api/dashboardsでGETリクエストを発行することで、（所有権またはグループを介して）アクセス権を持つすべてのダッシュボードを取得できます。
```
[
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
                "Created": "2016-12-18T23:28:08.250051418Z",
				"Guid": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]

```
### 特定のダッシュボードを入手する
特定のIDを取得するには、そのIDをダッシュ​​ボードのURLの最後に配置します。

```
GET /api/dashboards/2:
```
GUIDによって特定のダッシュボードを取得することも可能ですが、これはお勧めできません。
### すべてのユーザーのすべてのダッシュボードを取得する管理者
すべてのダッシュボードを取得するには、ユーザーは管理者である必要があり、/api/dashboards/allにGETリクエストを発行する必要があります。この要求が管理者以外のユーザーによって発行された場合は、アクセス権を持つすべてのダッシュボードを返す必要があります（/api/dashboardsの GETとは異なります）。

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
				"Guid": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
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
				"Guid": "55bc7236-39e4-11e9-94e9-54e1ad7c66cf",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```

## ダッシュボードの更新
ダッシュボードの更新（データの変更または名前、説明、GIDリストなどの変更）は、/api/dashboards/IDに対してPUT要求を発行することによって行われます。ここで、IDは特定のダッシュボードの固有のIDです。

この例では、ユーザー（UID 3）は、グループ3がダッシュボード（ID 2）にアクセスするための許可を追加したいと考えています。

```
GET /api/dashboards/2:
[
        {
                "ID": 2,
                "Name": "test2",
                "UID": 3,
                "GIDs": [],
                "Description": "test2 description",
				"Guid": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
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

ユーザーはターゲットダッシュボードを取得し、変更を加え、PUTリクエストを送信します。
```
WEB PUT /api/dashboards/2:
[
        {
                "ID": 2,
                "Name": "marketoverview",
                "UID": 3,
                "GIDs": [3],
                "Description": "marketing group dashboard",
				"Guid": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
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

サーバーは更新されたダッシュボード構造で更新要求に応答します。

## ダッシュボードを削除する
ダッシュボードを削除するには、url /api/dashboards/IDに対してDELETEメソッドを使用してリクエストを発行します。ここで、IDはダッシュボードの数値IDです。

## ユーザーが所有するすべてのダッシュボードを取得する
ユーザーが明示的に所有しているダッシュボードを取得するには、/api/users/UID/dashboardsでGETリクエストを発行します。これは、そのUIDが特に所有しているダッシュボードのみを返します。これには、ユーザーがグループメンバーシップを通じてアクセスできるダッシュボードは含まれません。
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
				"Guid": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```

## すべてのグループダッシュボードを取得する
特定のグループが/api/groups/GID/dashboardsにGETリクエストを発行するためにアクセスできるすべてのダッシュボードを取得するには、そのグループにタグ付けされたダッシュボードをすべて返します。ダッシュボードは複数のグループに分けることができるという点で、これは通常のUnixの許可から少し逸脱しています（したがって、グループは実際にはダッシュボードを所有するのではなく、むしろダッシュボードはグループのメンバーの一種です）。

```
WEB GET /api/groups/2/dashboards:
[
        {
                "ID": 3,
                "Name": "dashGroup1",
                "UID": 5,
                "GIDs": [
                        2
                ],
                "Description": "dashGroup1",
                "Created": "2016-12-28T21:37:12.696460531Z",
				"Guid": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
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
				"Guid": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```
### すべてのユーザーのすべてのダッシュボードを取得する管理者
すべてのダッシュボードを取得するには、ユーザーは管理者である必要があり、 /api/dashboards/allにGETリクエストを発行する必要があります。この要求が管理者以外のユーザーによって発行された場合は、アクセス権を持つすべてのダッシュボードを返す必要があります（/api/dashboardsの GETとは異なります）。

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
				"Guid": "d28b6887-ad55-479e-8af3-0cbcbd5084b1",
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
				"Guid": "5c6099dc-39e4-11e9-81a7-54e1ad7c66cf",
                "Data": {
                        "A": "A2",
                        "B": "B2",
                        "C": 12610078956637388,
                        "D": false
                }
        }
]
```
