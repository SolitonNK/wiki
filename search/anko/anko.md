## Anko

ankoモジュールはevalの補足としてより完全なスクリプト環境を提供します。検索エントリに対してより複雑な操作を可能にしますが、単純なeval式よりもankoスクリプトの開発、テスト、および展開に多くの作業が必要になります。スクリプトは[リソースシステム](#!resources/resources.md)のリソースとして保存されます。

ankoの構文はevalの構文と同じです。どちらも[github.com/mattn/anko](https://github.com/mattn/anko)から派生しており、Gravwell固有のタスク用にいくつかの追加機能が追加されています。

他のモジュールが十分に機能しない状況ではankoを使用することをお勧めします。通常、これは、エントリを以前のエントリと比較する必要がある、エントリを複製する必要がある、エントリからデータを抽出するのに複雑な操作が必要な、またはこれらの組み合わせの状況を意味します。

ドキュメントのこの部分では、ankoモジュールの使い方について簡単に説明しています。より詳細な説明については、[完全なankoモジュールのドキュメント](#!scripting/anko.md)と[Ankoスクリプト言語のドキュメント](#!scripting/scripting.md)を参照してください。

### 構文

`anko <script name> [script arguments]`

Ankoスクリプトはリソースとして保存されています。リソースの名前は、ankoモジュールへの最初の引数として指定する必要があります。スクリプト名の後に、追加の引数がスクリプト自体に渡されます。

### スクリプト例

次のスクリプトは、evalモジュールのドキュメントの例を再フォーマットしたものです。1行のevalの例よりもはるかに読みやすいことに注意してください:

```
func Process() {
	if len(Body) <= 10 {
		setEnum("postlen", "short")
	} else if len(Body) > 10 && len(Body) < 300 {
		setEnum("postlen", "medium")
	} else {
		setEnum("postlen", "long")
	}
}
```

スクリプトが`CheckPostLen`という名前のリソースにアップロードされていると仮定すると、スクリプトは次のように実行できます:

```
tag=reddit json Body | anko CheckPostLen | count by postlen | table postlen count
```

この`Process`関数は、ankoモジュールに到達する検索エントリごとに1回実行され、列挙値の長さを確認して、本体の長さに基づいて`Body`、新しい列挙値`postlen`を設定します。

この例はとても簡単です。`Process`関数のみを実装しています（オプション`Parse`または`Finalize`関数は実装されていません）。より複雑な例については、[完全なankoモジュールのドキュメント](#!scripting/anko.md)を参照してください。