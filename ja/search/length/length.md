## Length

`length`モジュールは、エントリデータまたは列挙値のいずれかの長さ（バイト単位）を算出します。  構文は次のとおりです:

```
length [-t target] [source]
```

`source`パラメータが動作する上の任意の列挙値の名前です。  ソースが指定されていない場合、lengthは生のエントリデータを使用します。  既定では、データまたは列挙値の長さは "length"という列挙値に書き込まれます。  `target`を指定すると、代わりにその名前の列挙値に長さが書き込まれます。

### サポートされているオプション

* `-t <target>`: 算された長さをデフォルトの "length"という名前ではなく、指定された列挙値に書き込みます。

### 使用例

| Command | 説明 |
|---------|-------------|
| length | エントリデータの長さをバイト数で取得し、その結果をデフォルトの列挙値`length`に格納します。 |
| length -t foo | エントリデータの長さを計算し、`length`の代わりに指定の名前（ここでは`foo`)の列挙値に結果を格納します。  |
| length Payload | 列挙値`Payload`の長さを、`length`という名前の列挙値に格納します。   |
| length -t foo Payload | 列挙値`Payload`の長さを、列挙値`foo`に保存します。  |
