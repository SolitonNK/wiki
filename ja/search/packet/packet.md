## パケット

パケットパイプラインモジュールは、イーサネット、IPv4、IPv6、TCP、UDPパケットからフィールドを抽出します。 各フィールドは、パケットを選択的にフィルタリングできる演算子をサポートしており、演算子が提供されていない場合は、フィールドが利用可能な場合に抽出されます。

パケットモジュールは、特定のプロトコルへのトラフィックをフィルタリングしたり、分析のためにパケットから特定のフィールドを抽出したりするのに便利です。

いくつかのフィールドモジュールは、ソースとデスティネーションを持つことができるフィールドでフィルタリングすることが望ましい柔軟なセクションを可能にします。 ソースとデスティネーションの両方があるIP、ポート、MACでの選択を可能にするために、特別なフィールドであるポート、IP、MACが利用可能です。 ソースまたはデスティネーションのどちらかが列挙された値にマッチした場合、そのフィールドにはマッチしたコンポーネントが入力されます。 例えば、tcp.Port==80の場合、tcp.SrcPortまたはtcp.DstPortのどちらかが80になるとマッチします。

### サポートされているオプション

* `-e <arg>`: “ -e”オプションは、レコード全体ではなく列挙値に作用します。  たとえば、パケット処理エンジンは、レイヤ2トンネルの分析など、抽出された値を処理できます。

### パケット処理オペレーター

| オペレーター | 名 | 説明
|----------|------|-------------
| == | 等しい | フィールドは等しくなければなりません
| != | 等しくない | フィールドは等しくてはいけません
| < | 未満 | フィールドはより小さい
| > | より大きい | フィールドはより大きくなければなりません
| <= | 以下 | フィールドは以下でなければなりません
| >= | 以上 | フィールドは以上でなければなりません
| ~ | 含む | フィールドはメンバーでなければなりません
| !~ | 含まない | フィールドはメンバーであってはいけません

### パケット処理サブモジュール
パケットプロセッサは、パケット内の特定のフィールドを分割するためのサブモジュールのリストをサポートしています。 各サブモジュールとフィールドは、パケットプロセッサがサブフィールドに基づいてイベントをフィルタリングできるようにする演算子のセットをサポートしています。 以下のサブモジュールが利用可能です:

| サブモジュール | 説明 |
|-----------|-------------|
| eth | イーサネットフレーム |
| ipv4 | IPバージョン4パケット |
| ipv6 | IPバージョン6パケット |
| tcp | TCPパケット |
| udp | UDPパケット |
| icmpv4 | ICMPパケット |
| dot1q | VLANタグ付きフレーム |
| dot11 | 802.11ワイヤレスパケット |
| dot11info | 802.11 情報要素 |
| modbus | modbus / TCPパケット |
| MPLS | マルチプロトコルラベル切り替え |

### パケット処理サブモジュール

#### イーサネット

| パケットタイプ | フィールド | オペレータ | 例 
|-----|-------|-----------|---------
| eth | SrcMAC | == != | eth.SrcMAC==DE:AD:BE:EF:11:22 
| eth | DstMAC | == != | eth.DstMAC != DE:AD:BE:EF:11:22 
| eth | MAC | == != | eth.MAC == DE:AD:BE:EF:11:22 
| eth | Len | > < <= >= == != | eth.Len > 0 
| eth | Type | < > <= >= == != | eth.Type < 5 
| eth | Payload | | eth.Payload

#### VLAN dot1q

| パケットタイプ | フィールド | オペレータ | 例 
|-----|-------|-----------|---------
| dot1q | VLANID | > < <= >= == != | dot1q.VLANID > 1024
| dot1q | Priority | > < <= >= == != | dot1q.Priority < 2
| dot1q | Type | > < <= >= == != | dot1q.Type == 2
| dot1q | DropEligible |  == != | dot1q.DropEligible == true

dot1qパケットサブモジュールは、VLANタグ付きパケットの解析を可能にするように設計されています。

##### 検索例

VLANタグ付きパケットで複数のIPv4アドレスをルーティングするすべてのMACアドレスを表示する検索例:

```
tag=pcap packet dot1q.Drop==false eth.SrcMAC ipv4.SrcIP | unique SrcMAC SrcIP | count by SrcMAC | eval count > 1 | table SrcMAC count
```

#### 802.11 無線LAN

| パケットタイプ | フィールド | オペレータ | 例 
|-----|-------|-----------|---------
| dot11 | Address1 | == != | dot11.Address1==DE:AD:BE:EF:11:22 
| dot11 | Address2 | == != | dot11.Address2 != DE:AD:BE:EF:11:22 
| dot11 | Address3 | == != | dot11.Address3 
| dot11 | Address4 | == != | dot11.Address4 
| dot11 | Type | < > <= >= == ! | dot11.Type == 1
| dot11 | ToDS | == ! | dot11.ToDS == true
| dot11 | FromDS | == ! | dot11.FromDS != false
| dot11 | Payload | | dot11.Payload

#### 802.11 情報要素

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| dot11info | SSID | == != | dot11.SSID != xfinitywifi

#### IPv4

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| ipv4 | Version | == != < > <= >= | ipv4.Version != 0b11 
| ipv4 | IHL | == != < > <= >= | ipv4.IHL == 08 
| ipv4 | TOS | == != < > <= >= | ipv4.TOS < 10 
| ipv4 | Length | == != < > <= >= | ipv4.Length > 0xff 
| ipv4 | ID | == != < > <= >= | ipv4.ID == 0x5 
| ipv4 | Flag | == != < > <= >= | ipv4.Flag == 0b1101 
| ipv4 | FragOffset | == != < > <= >= | ipv4.FragOffset > 3 
| ipv4 | TTL | == != < > <= >= | ipv4.TTL < 2 
| ipv4 | Protocol | == != < > <= >= | ipv4.Protocol != 0x08 
| ipv4 | Checksum | == != < > <= >= | ipv4.Checksum <= 0x1234 
| ipv4 | SrcIP | == != ~ !~ | ipv4.SrcIP ~ 192.168.1.1/16 
| ipv4 | DstIP | == != ~ !~ | ipv4.DstIP !~ 10.10.10.1/8 
| ipv4 | IP | == != ~ !~ | ipv4.IP ~ 192.168.1.0/14
| ipv4 | Payload | | ipv4.Payload

#### IPv6

| パケットタイプ | フィールド | オペレータ | 例 
|-----|-------|-----------|---------
| ipv6 | Version | == != < > <= >= | ipv6.Version == 0x08 
| ipv6 | TrafficClass | == != < > <= >= | ipv6.TrafficClass != 20 
| ipv6 | FlowLabel | == != < > <= >= | ipv6.FlowLabel == 0xDEADBEEF 
| ipv6 | Length | == != < > <= >= | ipv6.Length >= 100 
| ipv6 | NextHeader | == != < > <= >= | ipv6.NextHeader == 0x0800 
| ipv6 | HopLimit | == != < > <= >= | ipv6.HopLimit < 10 
| ipv6 | SrcIP | == != ~ !~ | ipv6.SrcIP != FF02::1 
| ipv6 | DstIP | == != ~ !~ | ipv6.DstIP !~ FE80::1/64 
| ipv6 | IP | == != ~ !~ | ipv6.IP == FE80::1/64 
| ipv6 | Payload | | ipv6.Payload

#### TCP

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| tcp | SrcPort | == != < > <= >= | tcp.SrcPort > 1024 
| tcp | DstPort | == != < > <= >= | tcp.DstPort <= 1024 
| tcp | Port | == != < > <= >= | tcp.Port == 80
| tcp | SeqNum | == != < > <= >= | tcp.SeqNum > 0xffff 
| tcp | AckNum | == != < > <= >= | tcp.AckNum < 112345 
| tcp | Window | == != < > <= >= | tcp.Window < 1024 
| tcp | [SYN/ACK/FIN/RST/PSH/URG/ECE/CWR/NS] | ==true, != true | tcp.SYN == true 
| tcp | Checksum | == != < > <= >= | tcp.Checksum != 0x1234 
| tcp | Urgent | == != < > <= >= | tcp.Urgent==0b111010101010101 
| tcp | DataOffset | == != < > <= >= | tcp.DataOffset > 96 
| tcp | Payload | ~ !~ | tcp.Payload ~ "HTTP"

#### UDP

| パケットタイプ | フィールド | オペレータ | 例 
|-----|-------|-----------|---------
| udp | SrcPort | == != < > <= >= | udp.SrcPort > 0xfff 
| udp | DstPort | == != < > <= >= | udp.DstPort < 1024 
| udp | Port | == != < > <= >= | udp.Port == 53 
| udp | Length | == != < > <= >= | udp.Length > 100 
| udp | Checksum | == != < > <= >= | udp.Checksum != 0x1234 
| udp | Payload | ~ !~ | udp.Payload


#### ICMP V4

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| icmpv4 | Type | == != < > <= >= | icmpv4.Type < 0x10 
| icmpv4 | Code | == != < > <= >= | icmpv4.Code ==0x2 
| icmpv4 | Checksum | == != < > <= >= | icmpv4.Checksum == 1024 
| icmpv4 | ID | == != < > <= >= | icmpv4.ID == 4 
| icmpv4 | Seq | == != < > <= >= | icmpv4.Seq > 100
| icmpv4 | Payload | == != ~ !~ | icmpv4.Payload

#### ICMP V6

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| icmpv6 | Type | == != < > <= >= | icmpv6.Type < 0x10 
| icmpv6 | Code | == != < > <= >= | icmpv6.Code != 0x2
| icmpv6 | Checksum | == != < > <= >= | icmpv6.Checksum == 1024 
| icmpv6 | Payload | == != ~ !~ | icmpv6.Payload

#### Modbus

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| modbus | Transaction | == != < > <= >= | modbus.Transaction==0x120
| modbus | Protocol | == != < > <= >= | modbus.Protocol==1
| modbus | Length | == != < > <= >= | modbus.Length > 0
| modbus | Unit | == != < > <= >= | modbus.Unit == 2
| modbus | Function | == != < > <= >= | modbus.Function == 0x05
| modbus | Exception | == != | modbus.Exception == false
| modbus | ReqResp | | modbus.ReqResp
| modbus | Payload | | modbus.Payload

たとえば、次のコマンドは、tumblr のすべての DNS クエリを検索します:

```
tag=pcap packet udp.DstPort==53 udp.Payload | grep -e Payload "tumblr" | text
```

`udp.DstPort == 53`コンポーネントは、UDPポート53宛てのパケットでのみ一致することを指定し、`udp.Payload`コンポーネントは、各パケットのペイロード部分を列挙値に抽出することを指定します。   次に、`grep`モジュールを使用してペイロードから単語" tumblr "を検索し、結果を表示のために`text`レンダラーに送信します。

#### MPLS

パケット検索モジュールはMPLSヘッダーをデコードでき、選択的なフィルタリングが可能です。次のMPLSフィールドが使用可能です。

| パケットタイプ | フィールド | オペレータ | 例 
|-----|-------|-----------|---------
| mpls | Label | == != < > <= >= | mpls.Label==0x10
| mpls | TrafficClass | == != < > <= >= | mpls.TrafficClass==4
| mpls | StackBottom | == != | mpls.StackBottom==true
| mpls | TTL | == != < > <= >= | mpls.TTL>1
| mpls | Payload | == != ~ !~ | mpls.Payload~foo

たとえば、次のコマンドは、MPLSヘッダーと5のトラフィックラベルを含むすべてのトラフィックをフィルターします。

```
tag=pcap packet mpls.Label==5 mpls.TrafficClass mpls.Payload | grep -e Payload "HTTP" | count by TrafficClass | table TrafficClass count
```

注：複数のレイヤーがある場合、MPLSパッケージモジュールは最初のMPLSレイヤーのみを参照します。[packetlayer](#!search/packetlayer/packetlayer.md)モジュールを使用して、ペイロードを参照して追加のレイヤーをデコードする必要があります。

<!---
### ICS-specific protocols

Gravwellには、Modbus、Ethernet/IP、およびCIP用の基本的なプロトコルクラッカーが含まれています。Ethernet/IPとCIPは複雑なため、基本的なデコーディングしか利用できませんが、ベースラインの確立や異常の検出に役立ちます。

| パケットタイプ | フィールド | オペレータ | 例
|-----|-------|-----------|---------
| modbus | Transaction | == != < > <= >= | modbus.Transaction != 0
| modbus | Protocol | == != < > <= >= | modbus.Protocol != 0
| modbus | Length | == != < > <= >= | modbus.Length > 1
| modbus | Unit | == != < > <= >= | modbus.Unit != 255
| modbus | Function | == != < > <= >= | modbus.Function == 0x0f
| modbus | Exception | ==true, !=true | modbus.Exception == true
| modbus | ReqResp | | modbus.ReqResp
| modbus | Payload | | modbus.Payload
#| enip | Command | == != < > <= >= | enip.Command > 0
#| enip | Length | == != < > <= >= | enip.Length > 5
#| enip | SessionHandle | == != < > <= >= | enip.SessionHandle != 0
#| enip | Status | == != < > <= >= | enip.Status != 0
#| enip | Options | == != < > <= >= | enip.Options == 0x02
#| enip | CommandSpecific | | enip.CommandSpecific
#| enip | Payload | | enip.Payload
#| enip | SenderContext | | enip.SenderContext
#| cip | Response | ==true, !=true | cip.Response == true
#| cip | Service | == != < > <= >= | cip.Service == 0x02
#| cip | ClassID | == != < > <= >= | cip.ClassID == 0x00
#| cip | InstanceID | == != < > <= >= | cip.InstanceID == 0x01
#| cip | Status | == != < > <= >= | cip.Status != 0
#| cip | AdditionalStatus | | cip.AdditionalStatus
#| cip | Data | | cip.Data
-->
