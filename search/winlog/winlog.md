# Winlog

winlogプロセッサは、XML形式のWindowsログデータ用の特別な目的の抽出プログラムです。  それはより一般的な[xmlモジュール](/#!search/xml/xml.md)を必要とするよりもむしろWindowsログエントリから多くの一般的なフィールドを抽出するための"ショートカット"を提供します

## サポートされているオプション

* `-e`: “ -e”オプションは、winlogモジュールが列挙値で動作することを指定します。  列挙値を操作すると、アップストリームモジュールを使用してログエントリを抽出した場合に役立ちます。
* `-or`: "-or"フラグは、抽出またはフィルタのいずれかが成功した場合に、winlogモジュールがエントリをパイプラインに沿って続行することを許可することを指定します。

## 処理オペレータ

各winlogフィールドは、高速フィルタとして機能できる一連の演算子をサポートします。  winlogモジュールの場合、すべてのフィールドは文字列として抽出されるので、文字列フィルタのみが利用可能です。

| オペレーター | 名 | 説明
|----------|------|-------------
| == | 等しい | フィールドは等しくなければなりません
| != | 等しくない | フィールドは等しくてはいけません
| ~ | サブセット | フィールドはメンバーでなければなりません
| !~ | サブセットではない | フィールドはメンバーであってはいけません

## データフィールド

以下の形式のログエントリがあるとします:

```
<Event xmlns="http://schemas.microsoft.com/win/2004/08/events/event">
  <System>
    <Provider Name="Microsoft-Windows-Security-Auditing" Guid="{543496D5-5478-49A4-A5BA-3E3B0428E31D}"/>
    <EventID>4689</EventID>
    <Version>0</Version>
    <Level>0</Level>
    <Task>13313</Task>
    <Opcode>0</Opcode>
    <Keywords>0x8020000000000000</Keywords>
    <TimeCreated SystemTime="2018-11-26T20:42:07.323695200Z"/>
    <EventRecordID>1624709</EventRecordID>
    <Correlation/>
    <Execution ProcessID="4" ThreadID="4392"/>
    <Channel>Security</Channel>
    <Computer>MY-PC</Computer>
    <Security/>
  </System>
  <EventData>
    <Data Name="SubjectUserSid">S-1-2-14</Data>
    <Data Name="SubjectUserName">GRAVUSER$</Data>
    <Data Name="SubjectDomainName">WORKGROUP</Data>
    <Data Name="SubjectLogonId">0x3e3</Data>
    <Data Name="Status">0x0</Data>
    <Data Name="ProcessId">0x1384</Data>
    <Data Name="ProcessName">C:\Windows\servicing\TrustedInstaller.exe</Data>
  </EventData>
</Event>
```

以下のフィールドを抽出することができます

| フィールド | XML仕様 |
|-------|----------|
| ProviderName | Event.System.Provider[Name] |
| ProviderGUID | Event.System.Provider[Guid] |
| GUID | Event.System.Provider[Guid] |
| EventID | Event.System.EventID |
| Version | Event.System.Version |
| Level | Event.System.Level |
| Task | Event.System.Task |
| Opcode | Event.System.Opcode |
| Keywords | Event.System.Keywords |
| TimeCreated | Event.System.TimeCreated[SystemTime] |
| EventRecordID | Event.System.EventRecordID |
| ProcessID | Event.System.Execution[ProcessID] |
| ThreadID | Event.System.Execution[ThreadID] |
| Channel | Event.System.Channel |
| Computer | Event.System.Computer |

上記にリストされていないフィールドを指定すると、winlogモジュールは`Event.Data [Name] == <field>`を抽出しようとします。 たとえば、上記の例の`SubjectLogonId（0x3e3）`は、winlogモジュールに"SubjectLogonId"を指定するだけで抽出できます。

## 例

次の例は、上記のサンプルログを参照しています。

プロセスID（4）とユーザー名（GRAVUSER$）を抽出するには:

```
winlog ProcessID SubjectUserName
```

EventID == 4689のセキュリティチャネル上のイベントのみからプロセス名を抽出するには、次の手順を実行します:

```
winlog EventID==4689 Channel==Security ProcessName
```
