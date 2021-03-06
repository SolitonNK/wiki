## Src

ソースモジュールは、ソースに基づいてエントリをフィルタリングするために使用されます。  これは、すべてのエントリが持つユニバーサルメタデータ項目です。  このモジュールは特定の場所から出ているエントリを見るのに役立ちます。  SrcはIPとサブネットをフィルタリングできます。

ソースモジュールは、IP、サブネット、整数、ベース16整数、および16バイトハッシュとして指定された値を変換/処理できます。 エントリソースフィールドは、非常に柔軟である必要があります。
注：ソースフィールドは、クエリを高速化するために、アクセラレーション/インデックスシステムで使用できます。 ただし、直接等価一致のみが加速システムを呼び出します。 サブネットによるフィルタリングまたは否定の使用は、アクセラレータを使用しません。

### 使用例

特定のソースからのエントリを削除します:

```
tag=syslog,apache,pcap src != 192.168.1.1 | count by TAG | chart count by TAG
```

特定のサブネットからのエントリのみを選択します:

```
tag=syslog,apache,pcap src ~ 192.168.1.0/24 | count by SRC | chart count by SRC
```

特定のサブネットからエントリを削除します

```
tag=syslog src !~ 192.168.0.0/16
```

整数IDを表すsrcを持つエントリのみを選択します

```
tag=syslog src == 1
```

16進数でエンコードされたIDを表すsrcでエントリを削除します

```
tag=syslog src != 0xfeadbeef
```

srcを16進文字列として含むエントリを検索します

```
tag=syslog src == 1234567890ABCDEF0011223344556677
```
