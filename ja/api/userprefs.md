## user preferences
### ユーザー設定

ユーザー設定APIは、ログイン間およびデバイス間で持続するためのGUI設定を格納するために使用されます。

/api/users/{id}/preferencesを取得すると、大量のJSONが返されます。管理者は任意のユーザー設定を要求でき、ユーザーは自分のセッションのみを要求できます。設定が存在しない場合は、nullを返します。

/api/users/preferencesをGETすると、すべてのユーザー設定が返されます。

ユーザー設定を更新するには、/api/users/{id}/preferencesを入力します。設定が存在しない場合は、とにかく提供されているJSONで更新してください。このAPIにPOSTは発生しません。PUTメソッドのペイロードはJSON BLOBになります。

GETとPUTが唯一の関連メソッドです。各ユーザーは本質的に唯一の好みJSONブロブを持つべきです。

/api/users/{id}/preferencesをDELETEすると、設定が削除されます（管理者または自分専用の場合）。


例ではGETでJSONが返されました:

```json
{
     "foo": "bar",
	 "bar": "baz"
}
```

## クライアントからの例
### 設定を要求する
```
WEB GET /api/users/5/preferences:
{
        "Name": "TestPref2",
        "Value": 57005,
        "DataData": "bW9yZSBpbXBvcnRhbnQgZGF0YQ=="
}
WEB GET /api/users/1/preferences:
{
        "Name": "TestPref1",
        "Val": 1234567890,
        "Data": "some important data",
        "Things": 3.1415
}
```
### すべての設定を要求する（管理者として）
```
WEB GET /api/users/preferences:
[]
```

```
WEB REQ PUT /api/users/1/preferences:
{
        "Name": "TestPref1",
        "Val": 1234567890,
        "Data": "some important data",
        "Things": 3.1415
}
```
```
WEB REQ PUT /api/users/5/preferences:
{
        "Name": "TestPref2",
        "Value": 57005,
        "DataData": "bW9yZSBpbXBvcnRhbnQgZGF0YQ=="
}
```
### 存在しないユーザーにする

404が見つかりません

### 非管理者として他の人を押したり引いたりする

403は禁止されます

### 設定を削除する
```
WEB REQ DELETE /api/users/5/preferences:
```
### 他の人を削除しようとしています

403は禁止されます