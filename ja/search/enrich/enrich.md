## Enrich

`enrich` モジュールはパイプラインの各エントリに列挙値を追加することができます。これらの値はモジュールの引数で指定したり、リソースから取得したりすることができます。`enrich` は、"User=admin"のような定数の列挙値を用いてデータに注釈を付けるために用いることができます。 `enrich` は、"User=admin" のような一定の列挙値でデータに注釈を付けることができ、可視化やレポーティング、複合クエリ内でのピボット、非一時的なデータの処理を簡素化することができます。

### サポートされているオプション

* `-r`: オプション。`-r` オプションは引数を必要とし、データを抽出するリソースの名前またはUUIDを指定します。
* `-o`: オプション。指定された既存の列挙値をすべて上書きする。通常は `-r` と組み合わせる。 

### 文字列定数でのエンリッチング

`enrich`の最も単純な使い方は、クエリ内で列挙された値とデータを指定することです。例えば、列挙値 "foo"と "bar"をすべてのエントリに追加するには、列挙値の名前と内容を空白で区切って指定するだけです:

```
tag=jsondata json val | enrich foo bar | table val foo
```

列挙値のペアを複数指定することもできます。例えば、それぞれ異なる内容の列挙値 "foo" と "bar" を作成するには、次のようにします。:

```
tag=jsondata json val | enrich foo "my data" bar "my other data" | table val foo bar
```

### リソースでのエンリッチング

`enrich`モジュールはCSVやルックアップテーブルのリソースからカラムを抽出することができる。リソースを利用する場合、`enrich` は最初の行のみを利用する(CSVリソースのカラムヘッダを除く)。カラムを指定することができ、何も指定しない場合はすべてのカラムが使用される。カラムが既存の列挙値と競合する場合は、`-o` フラグを使った場合のみ上書きされる。

例えば、リソース "data"から "foo"と "bar"のカラムを使用するには:

```
tag=jsondata val | enrich -r data foo bar | table val foo bar
```

さらに、`as` キーワードを使用することで、列挙値の名前をカラム名とは別の名前にすることができます。例えば、カラム "foo" を使って列挙し、その結果の列挙値を "bar" という名前にするには、次のようにします:

```
tag=jsondata val | enrich -r data foo as bar | table val bar
```

注 : `enrich` モジュールでは、リソースの最初の行のみが使用されます(CSVリソースのカラムヘッダを除く)。