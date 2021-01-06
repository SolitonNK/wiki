## Sort

デフォルトでは、Gravwell検索パイプライン内のすべてのものが一時的にソートされています。  Webインターフェースは、検索バックエンドでの一時的ソートの生の力を維持しながら、いくつかの追加のソート機能を提供します。

しかし、sortモジュールはユーザーが他の値でソートすることを可能にします。  これは情報を整理するのに非常に役立ちます。  たとえば、dnsmasqデーモンから要求されたトップドメインのテーブルを表示するクエリは、次のようになります:

```
tag=syslog grep dnsmasq | regex ".*query\[A\]\s(?P<dnsquery>[a-zA-Z0-9\.\-]+)" | count by dnsquery | sort by count desc | table dnsquery count
```

構文は`sort [by sortparam] [asc/desc]`です。  `sortparam`は並べ替えの基準となるパラメーターで、` time`、 `tag`、` src`、または列挙値のいずれかです。  sortパラメーターはオプションです。 指定されない場合、デフォルトで「時間」になります。  もう1つのパラメーターは、昇順（`asc`）または降順（`desc`）のどちらかでソートの方向を選択します。  指定されていない場合、デフォルトでは時間の降順でソートされ、他のすべてのパラメーターの昇順でソートされます。

ソート呼び出しの例:

| コマンド | 説明 |
|---------|-------------|
| `sort` | 時間の降順で並べ替え |
| `sort by tag` | エントリタグで昇順に並べ替え |
| `sort by count asc` | "count"という列挙値で昇順に並べ替え |
| `sort asc` | 時間の昇順で並べ替え |
| `sort by src desc` | 入力元で降順に並べ替え |

注：並べ替えモジュールはパイプラインを折りたたみ、すべてのレンダラーの2次的な一時検索を制限できます。  つまり、Webインターフェースでの概要グラフとタイムスライスの選択は、一時的ではないソートされた検索には影響しません。  並べ替え後のパイプラインモジュールが非一時的に順序付けられたデータを期待するように注意する必要があります。