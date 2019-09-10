## Src

ソースモジュールは、ソースに基づいてエントリをフィルタリングするために使用されます。  これは、すべてのエントリが持つユニバーサルメタデータ項目です。  このモジュールは特定の場所から出ているエントリを見るのに役立ちます。  SrcはIPとサブネットをフィルタリングできます。

### 使用例

特定のソースからのエントリを削除します:

```
tag=syslog,apache,pcap src != 192.168.1.1 | count by TAG | chart count by TAG
```

特定のサブネットからのエントリのみを選択します:

```
tag=syslog,apache,pcap src == 192.168.1.0/24 | count by SRC | chart count by SRC
```