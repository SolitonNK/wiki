## CSV

csvモジュールは、カンマ区切り値からデータを抽出してフィルタ処理するように設計されています。  csvモジュールは、コンマを含む可能性のある列の引用符、カンマのエスケープ、または空白で列を囲む機能など、csv値に関連するいくつかの追加規則に対応するように設計された拡張フィールドモジュールと考えることができます。

数値フィールドオフセットの指定は頻繁に使用すると面倒になることがあるため、[namedfields](#!search/namedfields/namedfields.md)モジュールはユーザーがアップロードしたリソースを使用してフィールドインデックスにわかりやすい名前を割り当てます。

### 抽出フィールドの指定

フィールドは、ゼロの基数からデータへのインデックスを指定することによって抽出されます。  インデックスは、角括弧で囲まれた正の整数を使用して指定されます。複数のディレクティブを指定することで複数の列を抽出できます。  列抽出インデクスは順番に指定する必要はありません。

抽出された列は`as <name>`、フィールドインデックス値の直後にディレクティブを追加することによって名前を変更できます。  たとえば、データから6列目を "uri"という名前の列挙値に抽出するには、抽出ディレクティブは次のようになります。`5 as uri`  名前変更ディレクティブが指定されていない場合、抽出された値にはインデックスと一致する名前が付けられます。  抽出された列は、同等性または含まれた値に基づいてエントリをすばやくフィルタリングすることを可能にするフィルタもサポートします。  フィルタは、名前変更ステートメントの前に指定する必要があります。  1列目の値が "stuff"である場所のみをエントリが通過できるようにする列ディレクティブの例を次に示します`0=="stuff"`。1列目の値が "stuff"と等しくないエントリのみを許可し、1列目のフィールドの名前を "things"に変更する`0 != "stuff" as things`。

重要: 列抽出索引は、10進数、8進数、または16進数として指定できます。  適用されるデフォルト名は、索引の元のテキスト値です。[0xA]の抽出ディレクティブは "0xA"という名前の11番目の列を抽出し、[010]は9番目の列を抽出して "010"という名前を適用します。

重要: " - "、 "。"、スペースなどの特殊文字を含むフィルター値や抽出名を指定するには、値を二重引用符で囲みます。

### サポートされているオプション

* `-e <arg>`: “ -e”オプションは、レコード全体ではなく列挙値に作用します。
* `-s` : -s”オプションは、csvモジュールが厳密モードで動作することを指定します。  いずれかの列指定を満たすことができない場合、そのエントリーはドロップされます。たとえば、0番目、1番目、2番目のフィールドが必要で、エントリに2列しかない場合は、strictフラグによってエントリが削除されます。

### フィルタリング演算子

csvモジュールは同等性に基づくフィルタリングを可能にします。  等価（ "等しい"、 "等しくない"、 "含む"、 "含まない"）を指定するフィルタが有効になっている場合、フィルタの指定に失敗したエントリはすべて削除されます。  フィールドが等しくない "！="として指定され、そのフィールドが存在しない場合、そのフィールドは抽出されませんが、エントリは完全にはドロップされません。

| オペレーター | 名 | 説明 |
|----------|------|-------------|
| == | 等しい | フィールドは等しくなければなりません
| != | 等しくない | フィールドは等しくてはいけません
| ~ | サブセット | フィールドに値が含まれています
| !~ | サブセットではない | フィールドに値が含まれていません

#### フィルタリング例

```
csv [0] == "foo" [2] != "bar" [3] ~ baz as ID
```

### データ抽出

CSVモジュールは常に抽出された列データから周囲の空白と二重引用符を削除します。  たとえば、次のエントリを検討してください:

```
2018-11-01T12:46:01.764386-06:00,daffodil, 15554,9870f7cd-b7d3-4bb6-8160-f5f146ebc764 , "OK, what sort of language would we have the world speak?
Isabella", "CL", Lucien
```

3列目、4列目、5列目は周囲の空白を含み、5列目はコンマと改行を囲む二重引用符を含みます。  次の引数でCSVモジュールを実行したとします:

```
csv [2] [3] [4] | table 2 3 4
```

出力は次のようになります（引用符または空白の空白がないことに注意してください:

| 2 | 3 | 4 |
|----------|------|-------------|
| 15554 | 9870f7cd-b7d3-4bb6-8160-f5f146ebc764 | OK, what sort of language would we have the world speak?<br>Isabella |


### クエリ例

CSVのhttp.logフィードからURL列を抽出して"url"という名前を付けます。

```
tag=brohttp csv [9] as url
```


CSVファイルのhttp.logフィードからURLと要求者のフィールドを抽出し、URLにスペースが含まれているエントリのみをフィルタリングし、結果をテーブルに出力します。

```
tag=brohttp csv [9] ~ " " as url [2] as requester | table url requester
```


6列目が "もの"であってはならず、4列目が "もの"を含まなければならない4、5、および6列目を抽出します。

```
tag=default csv [5]!=stuff [4] [3]~"things" | table 3 4 5
```


apacheアクセスログからURIを抽出してから、 "DATA"パラメータをCSVとして解析します。

```
tag=apache tag=apache regex "GET\s(?P<base>[^\s\?]+)\?(?P<params>\S+)\s" | regex -e params "DATA\=(?P<dataparam>[^\s&]+)" | csv -e dataparam [0] [1] [2] | table 0 1 2
```