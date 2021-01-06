# The Gravwell GUI

ほとんどのユーザーは、Web GUIを通じてGravwellと対話します。このページでは、いくつかの高レベルのメニューとインターフェースの概念を説明します。

* [検索インターフェイス](queries/queries.md)
* [ダッシュボード](dashboards/dashboards.md)
* [ラベルとフィルタリング](labels/labels.md)
* [リソース管理](#!resources/resources.md)
* [オートエクストラクタの管理](#!configuration/autoextractors.md)
* [キットの管理](#!kits/kits.md)
* [高度なGUIのユーザー設定](#!configuration/gui.md)

## GUI入門

ログイン後、デフォルトでは以下のような検索ページが表示されます。

![](searchpage.png)

上部にあるアイコン(メインメニュー、ヘルプ、通知、ユーザープロファイルのラベル)は、Gravwell GUI内でいつでも見ることができます。

## メインメニュー

左上の"ハンバーガー"メニューをクリックすると、メインメニューが開きます:

![](menu.png)

このメニューは、ダッシュボード、クエリライブラリ、プレイブックなど、Gravwellのすべての主要な機能にアクセスするために使用されます。メニュー内のいくつかの項目は、実際にはサブメニューであり、追加オプションを表示するために拡張することができることに注意してください:

![](menu-expanded.png)

これらのサブメニュー内のアイテムは、通常、トップレベルのアイテムよりも使用頻度が低くなります。

注意: これらのスクリーンショットには、管理者専用の管理ツールを含む「管理者」サブメニューが含まれており、管理者としてフラグを立てられたユーザーのみが見ることができます。

## 通知

画面右上のベル型の通知アイコンをクリックすると通知表示が表示される:

![](notifications.png)

通知の"スヌーズ"ボタンをクリックすると、アイコンに表示されているカウンターからその通知が削除されます。

通知の種類によっては、「削除」アイコンをクリックすると通知が完全に消去される場合があります。通知の中には、永続的で削除できないものもあれば、システム全体を対象としたもので管理者のみが削除できるもの、現在のユーザーを対象としたものでそのユーザーが削除できるものもあります。ユーザーが削除できない通知の上で「削除」をクリックしても害はないことに注意してください。

オフラインのインデクサーのような重大な通知は、通知ベルを警告サインに変更します:

![](notif-warn.png)

## ユーザー設定

画面右上のユーザープロファイルアイコンを選択すると、小さなドロップダウンメニューが表示されます:

![](user-dropdown.png)

### アカウント

「アカウント」を選択すると、以下のような設定ページが表示されます。ここでは、メールアドレス、表示名、パスワードを変更することができます。画面下部にある「すべてのセッションをログアウトする」ボタンをクリックすると、すべてのクライアントマシンであなたのアカウントのすべてのアクティブなセッションが削除されます。

![](account-prefs.png)

### インターフェースと外観 

環境設定ページの 2 番目のタブ、"Interface & Appearance "には、Gravwell のユーザーインターフェースをカスタマイズするためのオプションがあります。" Interface theme "ドロップダウンは、GUI全体のカラースキーム(常に人気のあるダークモードを含む)を選択するため、特に興味深いものです。

" チャートテーマ "ドロップダウンは、チャートを描画するときに使用される異なるカラーパレットを選択します。エディタテーマとフォントサイズオプションは、オートメーションスクリプトの作成や他のいくつかの場所で使用されるGravwellの組み込みテキストエディタの外観を制御します。

![](interface-prefs.png)

### 設定

3つ目のタブ、"Preferences "では、Gravwellのデフォルトの動作を変更することができます。

![](general-prefs.png)

「ホームページ」ドロップダウンメニューでは、ログイン後に表示されるページを選択するか、メインメニューの横にあるGravwellアイコンをクリックした後に表示されるページを選択します。デフォルトでは、新しい検索ページが表示されますが、代わりにダッシュボード、キット、またはプレイブックのリストを表示するように選択することもできます。

「検索グループの可視化」オプションでは、すべての検索結果を特定のグループと共有することができます; これは共同作業をするのに便利な方法です。スクリーンショットでは、ユーザーは "foo "という名前のグループを選択しています。そのグループのすべてのメンバーは、このユーザーが今後実行する検索にアクセスできるようになります。

" Advanced Preferences" セクションは、ほとんどのユーザーが無視できます。「開発者モード」を選択すると、JSON環境設定を手動で編集することができます(詳細は [this page](!#configuration/gui.md) を参照してください)。

### メールサーバー

最後のタブ、"Email Server "は、スケジュールされたスクリプトを介して自動化された電子メールアラートを行う予定のユーザーにとって非常に重要です。電子メールを送信する前に、有効なSMTP設定で設定する必要があります。

![](email-prefs.png)

「Server」はSMTPサーバー、「Port」はSMTPに使用するポート、「Username」と「Password」はそのサーバーの認証を行います。"Use TLS "は、サーバーがTLS接続を期待している場合に有効にする必要があります。サーバーが自己署名証明書を使用している場合には、"Disable TLS certification validation "オプションが用意されています。

フィールドが入力されたら、「設定の更新」をクリックして保存し、「設定のテスト」をクリックしてテストメールを送信します。