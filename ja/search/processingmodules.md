# 検索処理モジュール

検索モジュールは、パススルーモードでデータを操作するモジュールであり、何らかのアクション（フィルター、修正、ソートなど）を実行し、パイプラインの下にエントリを渡すことを意味します。多くの検索モジュールがあり、それぞれが独自の軽量スレッドで動作します。 これは、検索に10個のモジュールがある場合、パイプラインは分散して10個のスレッドを使用することを意味します。各モジュールのドキュメントには、そのモジュールが分散検索の崩壊やソートを引き起こすかどうかが示されています。コラボレーションするモジュールは、分散型パイプラインを強制的にコラボレーションさせます。 検索を開始する際には、最初に崩壊したモジュールの上流にできるだけ多くの並列モジュールを配置するのがベストです。

モジュールの中には、`count` のようにデータを大幅に変換したり、折りたたんだりするものがあります。これらの畳み込みモジュールに続くパイプラインモジュールは、生のデータや以前に作成された列挙値を扱っていない可能性があります。つまり、`count by Src` のようなものは、`10.0.0.0.1 3`のようなエントリを持つデータを折りたたまれた結果に変換します。説明するために、検索 `tag=* limit 10 | count by TAG | raw` を実行するとcountモジュールからの生の出力が表示され、`tag=* limit 10 | count by TAG | table TAG count DATA` を実行するとテーブルモジュールから見たように生のデータが凝縮されていることがわかります。

## ユニバーサルフラグ

いくつかのフラグは、いくつかの異なる検索モジュールに表示され、全体を通して同じ意味を持ちます。

* `-e <ソース名>` は、モジュールが入力データをエントリのデータフィールドからではなく、与えられた列挙された値から読み込もうとすることを指定します。これは[json](json/json.md)のようなモジュールで、JSONでエンコードされたデータがより大きなデータレコードから抽出されている場合に便利です。例えば、以下の検索はHTTPパケットのペイロードからJSONフィールドを読み込もうとします: `tag=pcap packet tcp.Payload | json -e Payload user.email` のように。
* `-r <リソース名>` は [resources](#!resources/resources.md) システム内のリソースを指定します。これは一般的に、[geoip](geoip/geoip.md)モジュールが使用するGeoIPマッピングテーブルなど、モジュールが使用する追加データを格納するために使用されます。
* `-v`は、通常のパス/ドロップロジックを反転する必要があることを示します。 たとえば、[grep](grep/grep.md)モジュールは通常、特定のパターンに一致するエントリを渡し、一致しないエントリを削除します。 `-v`フラグを指定すると、一致するエントリが削除され、一致しないエントリが渡されます。
* `-s` は "strict" モードを示します。モジュールが通常、いくつかの条件のうちどれか1つでも満たされていればエントリをパイプラインの下に進めることを許可する場合、strictフラグを設定すると、すべての条件が満たされた場合にのみエントリが進むことを意味します。例えば、[require](require/require.md)モジュールは、通常、必須の列挙値のいずれか1つを含むエントリを渡しますが、`-s`フラグが使用されている場合、指定されたすべての列挙値を含むエントリのみを渡します。
* `-p` は "permissive" モードを示します。 パターンとフィルタが一致しないときにモジュールが通常はエントリを削除する場合、permissive フラグはモジュールを通過させるように指示します。 [regex](regex/regex.md) と [grok](grok/grok.md) モジュールは permissive フラグが有用な良い例です。

## ユニバーサル列挙値

以下の列挙値は、各エントリで利用可能です。これらは実際には生のエントリ自体のプロパティのための便利な名前ですが、列挙された値の名前として扱うことができます。

* SRC -- エントリーデータのソース。
* TAG -- エントリーに添付されているタグ。
* TIMESTAMP -- エントリーのタイムスタンプ。
* DATA -- 実際の入力データ。
* NOW -- 現在の時刻。

これらはユーザー定義の列挙値と同じように使うことができるので、`table foo bar DATA NOW`が有効です。どこかで明示的に抽出する必要はありません。いつでも利用可能です。

## 検索モジュールのドキュメント

* [abs](abs/abs.md)
* [alias](alias/alias.md)
* [anko](anko/anko.md)
* [base64](base64/base64.md)
* [count](math/math.md#Count)
* [diff](diff/diff.md)
* [enrich](enrich/enrich.md)
* [entropy](math/math.md#Entropy)
* [eval](eval/eval.md)
* [first/last](firstlast/firstlast.md)
* [geoip](geoip/geoip.md)
* [grep](grep/grep.md)
* [hexlify](hexlify/hexlify.md)
* [ip](ip/ip.md)
* [ipexist](ipexist/ipexist.md)
* [iplookup](iplookup/iplookup.md)
* [join](join/join.md)
* [langfind](langfind/langfind.md)
* [length](length/length.md)
* [limit](limit/limit.md)
* [lookup](lookup/lookup.md)
* [lower](upperlower/upperlower.md)
* [maclookup](maclookup/maclookup.md)
* [Math (list of math modules)](math/math.md)
* [max](math/math.md#Max)
* [mean](math/math.md#Mean)
* [min](math/math.md#Min)
* [packetlayer](packetlayer/packetlayer.md)
* [regex](regex/regex.md)
* [require](require/require.md)
* [slice](slice/slice.md)
* [sort](sort/sort.md)
* [split](split/split.md)
* [src](src/src.md)
* [stats](stats/stats.md)
* [stddev](math/math.md#Stddev)
* [strings](strings/strings.md)
* [subnet](subnet/subnet.md)
* [sum](math/math.md#Sum)
* [taint](taint/taint.md)
* [time](time/time.md)
* [unique](math/math.md#Unique)
* [upper](upperlower/upperlower.md)
* [variance](math/math.md#Variance)
* [words](words/words.md)
