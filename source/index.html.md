---
title: Blockstor API Reference

language_tabs:
  - javascript
  - nim

toc_footers:
  - <a href='https://github.com/zenywallet/blockstor'>Blockstor Source Code</a>

includes:

search: true
---

# Introduction - はじめに

Blockstor はウォレットサービスのバックエンド向けに設計された Block Explorer です。一般的な Block Explorer は多くの情報を保持し情報表示に関しては多機能ですが、ウォレットサービスには不要な部分が多く含まれています。 Blockstor ではウォレットサービスに不要な部分を排除し、ウォレット向けのデータ構造にすることでレスポンスを高速化しています。

Blockstor は、Webウォレット等のクライアントから直接利用するためのAPIサービスではなく、ユーザーウォレットのアドレス等を管理するサーバー側のウォレットサービスからアクセスすることを想定して実装されています。

# Features - 特徴

### 高速なAPIレスポンス
REST API のクエリはどんなものでもすべて高速に結果を返します。UTXOや送受信履歴は入出力の多いアドレスでも一瞬で返します。

### 容易なアルトコイン対応
Bitcoin 用に設計された Block Explorer やウォレットアプリケーションをアルトコインに対応させる場合、アルトコイン用のハッシュアルゴリズム等の実装が必要であったり、改造に手間がかかることがありますが、Blockstor ではハッシュアルゴリズムの実装は不要で、パラメータ設定のみでBitcoin ベースのアルトコイン対応ができます。

### コイン最大数量は64ビット
コインの最大数量は Satoshi 単位で64ビットまでサポートしており、Bitcoin より発行枚数を増やしたアルトコインに対応できます。
ただし、Javascript の仕様により53ビットの数値までしか扱えないので、Number.MAX_SAFE_INTEGER (9007199254740991) より大きな数値は、数値ではなく数字の文字列に変換されます。

### 効率的なデータ同期
ウォレットサービス側でデータ読み出し位置を管理でき、データの同期を効率的にできます。
Blockstor 内のトランザクション情報は、Genesis Block のトランザクションから順番にシーケンス番号を割り当てて管理しています。
特定のシーケンス番号より新しい情報のみを取得するなどが可能です。
また、Orphan Block となりブロックがロールバックした場合は、巻き戻された位置のシーケンス番号を取得することができます。
その他、シーケンス番号は取引履歴の表示順をソートするなどのためにも利用できます。

### WebSocket ストリーミング
WebSocket によるストリーミングAPIでリアルタイムな情報を受信できます。
確認済みとなったブロックのデータや未確認状態のトランザクションの情報を受信できます。
いずれもアドレスベースに整理されており、ウォレットサービス側で必要なアドレスの情報のみをフィルタリングすることが容易にできます。

### UTXO数の把握
UTXOの数を簡単に取得できます。
マイニングプール等からの受信回数が極めて多いアドレスが増えてきており、一度に送信することができなくなった状態のウォレットがたくさんあるようです。
アドレスの入力トランザクション数の把握しユーザーに警告したり、細分化した入力トランザクションをまとめることで一括で送信できるような状態にする機能等の実現に利用できます。

### TCP接続によるブロックインデックス
ブロックをインデックスする際は、高速なTCP接続とRPC接続の両方をサポートしています。
サービス起動時はTCP接続モードでブロックのインデックスを試し、接続失敗した場合はRPC接続モードにフォールバックします。
ただし、RPC接続モードはTCP接続モードに比べると低速です。すべてのブロックを同期した後はRPC接続モードで動作します。

### BitcoinJS を利用
BitcoinJS を利用し、SegWit アドレスに対応しています。
コインの数量を64ビットまで扱えるようにした BitcoinJS を利用しています。それはWebブラウザ等のクライアント側でも利用できます。

### シンプルなコード
改造がしやすいシンプルなコードです。
取得情報を追加することも可能です。例えばアドレス保有数量のランキングシステムなども簡単に作成できるでしょう。

# Installation - インストール

Blockstor の実行には Node.js が必要です。nvm (Node Version Manager) を使用して Node.js をインストールすることを推奨します。

下記のサイトの手順に従い nvm をインストールする。

[https://github.com/nvm-sh/nvm](https://github.com/nvm-sh/nvm)

> nvm のインストール

```
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.34.0/install.sh | bash
```

nvm をインストールしたら環境変数を読み込むために、一旦 Shell をログアウトして再ログインする。

nvm を使って Node.js をインストールする。

> Node.js のインストール

```
# インストール可能なバージョンを表示
nvm ls-remote

# Node.js バージョン v10 の最新版をインストール
nvm install 10
```

下記サイトから Blockstor をインストールする。

[https://github.com/zenywallet/blockstor](https://github.com/zenywallet/blockstor)

> Blockstor のインストール

```
git clone https://github.com/zenywallet/blockstor.git
cd blockstor
npm install
```

# Blockstor Config - 設定
> デフォルト設定のコピー

```
cp config/default-sample.json5 config/default.json5

```

config フォルダ内の default-sample.json5 を default.json5 にコピーして、設定を編集する。

### パラメータ設定
Parameter |   | Requirements | Description
--------- | - | ------------ | -----------
rpc | host | required | RPC接続先ホスト名
    | port | required | PRC接続先ポート
    | user | required | RPC接続先ユーザー名
    | pass | required | RPC接続先パスワード
tcp | host | required | TCP接続先ホスト名
    | post | required | TCP接続先ポート
    | protocol_version | required | プロトコルバージョン
    | start_string | required | メッセージ開始文字
server | http_port | required | httpポート番号
       | http_path | optional | REST APIのパス(default: "/api")
       | ws_port | required | WebSocketポート番号
       | ws_heartbeat | optional | WebSocket でハートビート機能を 0 - 利用しない。1(default) - 利用する
       | ws_path | optional | WebSocketエンドポイントのパス(default: "/api")
       | ws_deflate | optional | deflate を 0(default) - 無効、1 - 有効
db | rawblocks | optional | rawblock データを Blockstor で 0(default) - 保持しない。1 - 保持する
   | remove_orphan_rawblocks | optional | orphan となったブロックを rawblocks からを 0(default) - 取り除かない。1 - 取り除く
networks | "coin name" | required | BitcoinJS のネットワーク設定
target_network | | required | networksで使用するコインを指定する
apikeys | "some-api-key-value" | optional | APIキーおよびAPIキーの説明を設定を列挙する

> apikey 生成方法

```
$ uuidgen
37e27e0a-ecdf-42fa-8f6b-60b29cf5625d
```

<aside class="warning">
ロールバックするときに orphanブロックの情報が必要になります。rawblock データを保持しない設定の場合、orphanブロックの情報を coind から取得します。接続先 coind を変更すると orphanブロックを持っていない可能性もあるため、接続先 coind は変更しないようにしてください。
</aside>

<aside class="warning">
TCP接続はフルノードの機能を持っていないので、必ず自前で用意した coind に接続してください。
</aside>

<aside class="notice">
http と WebSocket を一つのポート番号で動作させることもできます。その場合、http_port と ws_port に同じポート番号を設定してください。
</aside>

### BitcoinJS パラメータ設定
networks."coin name" のパラメータ説明です。

Parameter |   | Requirements | Description
--------- | - | ------------ | -----------
messagePrefix | | required | メッセージにサインをするときに使用するプレフィックス文字列
bech32        | | optional | bech32プレフィックスアドレス文字列
bech32_extra  | | optional | bech32パラメータ以外にサポートするbech32プレフィックスアドレス文字列のArray
bip32 | public  | required | base58Prefixes EXT_PUBLIC_KEY
bip32 | private | required | base58Prefixes EXT_SECRET_KEY
pubKeyHash  | | required | base58Prefixes PUBKEY_ADDRESS
scriptHash  | | required | base58Prefixes SCRIPT_ADDRESS
scriptHash2 | | optional | base58Prefixes SCRIPT_ADDRESS2 (Monacoin用)
wif     | | required | base58Prefixes SECRET_KEY
wif_old | | optional | base58Prefixes OLD_SECRET_KEY (Monacoin用)

<aside class="notice">
messagePrefix は、core の src/validation.cpp strMessageMagic を参照してください。先頭の数字は、string の文字列長を設定してください。
</aside>

<aside class="notice">
base58Prefixes は、core の src/chainparams.cpp を参照してください。
</aside>

<aside class="notice">
bech32_extra, scriptHash2, wif_old は、BitcoinJS標準のパラメータではありません。
</aside>

# Blockstor Startup - 起動

Node.js で Blockstor を起動する。

> Blockstor の起動方法

```
$ node index.js
2019-07-04 02:55:42 height(blockstor/coind)=0/1617610 delay=1617610 - start
2019-07-04 02:58:36 height(blockstor/coind)=60343/1617612 delay=1557269 - 2015-01-10 07:55:14 (TCP mode)
```

同期中は "(TCP mode)" と表示されていることを確認してください。すべてのブロックを同期した後はRPC接続に切り替わり表示が "(RPC mode)" となります。

synced が表示された場合、Core との同期が済んだことを意味しますが、Core自体の同期が終わっているかどうかは考慮されていません。

接続エラーなど何らかの異常が発生した場合は積極的にプロセスを終了するように設計されています。そのため、永続的な稼働を目指す場合はシェルスクリプト等で再起動するようにしてください。

> ブロックが見つからない場合の例

```
$ node index.js
2019-06-20 23:46:30 height(blockstor/coind)=1596678/1582963 delay=-13715 - start
Error: Block not found
ERROR: getBlock 0000000624710d3282e48c5d49e94f7e30402dc2173ab187d5c87bf107122214
abort
```

> TCP接続できなかった場合の例1

```
$ node index.js
2019-07-04 02:55:42 height(blockstor/coind)=0/1617610 delay=1617610 - start
2019-07-04 03:43:46 height(blockstor/coind)=62200/1617641 delay=1555441 - 2015-01-12 06:24:20 (RPC mode)
```

> TCP接続できなかった場合の例2

```
$ node index.js
2019-06-20 23:51:04 height(blockstor/coind)=1596678/1597102 delay=424 - start
Error: connect ECONNREFUSED 127.0.0.1:9253
ERROR: Cannot connect.
2019-06-20 23:51:35 height(blockstor/coind)=1598990/1598990 delay=0 - synced
2019-06-20 23:51:36 #1598991 0000000d9508b93867eab223121535c56c5901b8665f8798558de220e1dc3a8a 2019-06-14 13:51:17
```

> ロールバックが発生した場合の例

```
$ node index.js
2019-07-03 05:34:18 height(blockstor/coind)=1616696/1616758 delay=62 - start
Error: connect ECONNREFUSED 127.0.0.1:9253
ERROR: Cannot connect.
2019-07-03 05:34:25 height(blockstor/coind)=1616758/1616758 delay=0 - synced
2019-07-03 05:35:37 #1616759 00000008b1c15a3e0a26746db91d39975742c79c9ccd79a0d8073b581efad8b9 2019-07-03 05:34:31
2019-07-03 05:36:40 #1616760 00000003b9252b45781126ff8648779978acc3fcea0949212968c08373013383 2019-07-03 05:36:12
...
2019-07-03 05:53:39 #1616775 00000000a693b69716228f1dcee1a3b00b944939c3e66f799f7e5c1a42af7cf2 2019-07-03 05:52:47
2019-07-03 05:53:55 #1616776 00000007f176023c88cb58d4ecd7a7f7c97bca1519df570e8bce870acfb61c54 2019-07-03 05:53:40
rollback height=1616776
2019-07-03 05:55:36 #1616776 00000004e0c96cc6cace0c5025ce491c454d2f1131e4ec36e694bd13f98572e1 2019-07-03 05:53:39
2019-07-03 05:55:36 #1616777 00000004b85268eca32c37555db1a11231d8af3cbe7ce7843eefc0cd1bcd0b94 2019-07-03 05:54:52
...
2019-07-03 06:04:55 #1616781 00000003df839912f6c74d908160a8d03c7b6633520f4dc349435fe37833960a 2019-07-03 06:03:59
mempool: count=0
```

# REST API

## アドレス情報取得 /addr

アドレスの数量情報を取得する。

### HTTP Request
`GET https://blockstor.bitzeny.jp/api/addr/{addr}`

> アドレス情報取得のサンプル

```javascript
// nodejs
var request = require('request');

request.get({
  url: 'https://blockstor.bitzeny.jp/api/addr/ZenyAddress...xyz',
  json: true
}, function(err, res, result) {
  if(!err) {
    console.log(result);
  }
});

// javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
  if(xhr.readyState != 4 || xhr.status != 200) return;
   console.log(JSON.stringify(xhr.response, null, ' '));
};
xhr.open('GET', 'https://blockstor.bitzeny.jp/api/addr/ZenyAddress...xyz', true);
xhr.responseType = 'json';
xhr.send();
```

```nim
import httpclient, json

let client = newHttpClient()
let res = client.request("https://blockstor.bitzeny.jp/api/addr/ZenyAddress...xyz")
if res.status == Http200:
  echo parseJson(res.body)
```

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
{addr} | required | false | 情報取得対象のアドレス

### Headers

`Content-Type: application/json`

### Response
> アドレス数量情報のサンプル

```json
{
  "err": 0,
  "res": {
    "balance": 182549587641,
    "utxo_count": 19,
    "unconf": {
      "out": 0,
      "in": 10185033090
    }
  }
}
```

Attribute |   |   |   | Description
--------- | - | - | - | -----------
err | | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | | | |
 | balance | | | 数量 (Satoshi)
 | utxo_count | | | UTXO数
 | unconf | | | ※未確認のトランザクションがある場合のみ
 | | out | | 対象アドレスから他のアドレスへ送信中の未確認数量 (Satoshi)
 | | in | | 他のアドレスから対象アドレスへ受信中の未確認数量 (Satoshi)

## 複数アドレス情報取得 /addrs
複数のアドレスの数量情報を取得する。

### HTTP Request
`POST https://blockstor.bitzeny.jp/api/addrs`

> 複数アドレス情報取得のサンプル

```javascript
// nodejs
var request = require('request');

request.post({
  url: 'https://blockstor.bitzeny.jp/api/addrs',
  json: true,
  body: {addrs: ['ZenyAddress1...xyz', 'ZenyAddress2...xyz']}
}, function(err, res, result) {
  if(!err) {
    console.log(JSON.stringify(result, null, '  '));
  }
});

// javascript
var xhr = new XMLHttpRequest();
xhr.onreadystatechange = function() {
  if(xhr.readyState != 4 || xhr.status != 200) return;
   console.log(JSON.stringify(xhr.response, null, '  '));
};
xhr.open('POST', 'https://blockstor.bitzeny.jp/api/addrs', true);
xhr.setRequestHeader('Content-Type', 'application/json');
xhr.responseType = 'json';
xhr.send(JSON.stringify({addrs: ['ZenyAddress1...xyz', 'ZenyAddress2...xyz']}));
```

```nim
import httpclient, json

let client = newHttpClient()
client.headers = newHttpHeaders({"Content-Type": "application/json"})
var postdata = %*{"addrs": ["ZenyAddress1...xyz", "ZenyAddress2...xyz"]}
let res = client.request("https://blockstor.bitzeny.jp/api/addrs", httpMethod = HttpPost, body = $postdata)
if res.status == Http200:
  echo parseJson(res.body).pretty
```

### Request Body
`{addrs: [addr1, addr2, ..., addrN]}`

Field | Field Type | Requirements | Default | Description
----- | ---------- | ------------ | ------- | -----------
addrs | array[string] | required | false | 情報取得対象のアドレスを列挙する

### Headers
`Content-Type: application/json`

### Response
> 複数アドレス数量情報のサンプル

```json
{
  "err": 0,
  "res": [
    {
      "balance": 8112500516522,
      "utxo_count": 2586
    },
    {
      "balance": 50730025701,
      "utxo_count": 665
    }
  ]
}
```

Attribute |   |   |   | Description
--------- | - | - | - | -----------
err | | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | | | | [addr1の数量情報, addr2の数量情報, ..., addrNの数量情報]

※各数量情報の形式は [アドレス情報取得 /addr](#addr) を参照

## UTXO情報取得 /utxo
アドレスのUTXO情報を取得する。

### HTTP Request
`GET https://blockstor.bitzeny.jp/api/utxo/{addr}?gte=seq&gt=seq&lte=seq&lt=seq&limit=num&seqbreak=flag&reverse=flag&unconf=flag`

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
{addr} | required | false | 情報取得対象のアドレス
gte | optional | false | 指定されたシーケンス番号以上のデータを取得する (greater than or equal)
gt | optional | false | 指定されたシーケンス番号より大きなデータを取得する (greater than)
lte | optional | false | 指定されたシーケンス番号以下のデータを取得する (less than or equal)
lt | optional | false | 指定されたシーケンス番号より小さなデータを取得する (less than)
limit | optional | 1000 | 取得するデータ数, 最大50000
seqbreak | optional | 0 | 0 - 同一シーケンス番号を分割しない, 1 - 同一シーケンス番号を考慮せず分割する
reverse | optional | 0 | 0 - シーケンス番号の古い順, 1 - シーケンス番号の新しい順
unconf | optional | 0 | 0 - 未確認の受信トランザクションを含めない, 1 - 未確認の受信トランザクションを含む

### Headers
`Content-Type: application/json`

### Response
> UTXO情報のサンプル

```json
{
  "err": 0,
  "res": [
    {
      "sequence": 2531366,
      "txid": "7e307028d20c985204ad4757a9bacee940d42720e498d12bb92722bf8fbc13ae",
      "n": 0,
      "value": 6250000000
    },
    {
      "sequence": 2552650,
      "txid": "a99564f7653809844b8b7219dd41ad2c8e316608dc68cec5bfd141625464a578",
      "n": 0,
      "value": 6250000000
    },
    ...
  ]
}
```

Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | [ { | |
 | | sequence | Blockstor シーケンス番号
 | | txid | トランザクションID
 | | n | トランザクションIDの格納番号
 | | value | 数量 (Satoshi)
 | }, ... ] | |

## 複数UTXO情報取得 /utxos
複数のアドレスのUTXO情報を取得する。

### HTTP Request
`POST https://blockstor.bitzeny.jp/api/utxos?gte=seq&gt=seq&lte=seq&lt=seq&limit=num&seqbreak=flag&reverse=flag&unconf=flag`

### Request Body
`{addrs: [addr1, addr2, ..., addrN]}`

Field | Field Type | Requirements | Default | Description
----- | ---------- | ------------ | ------- | -----------
addrs | array[string] | required | false | 情報取得対象のアドレスを列挙する

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
gte | optional | false | 指定されたシーケンス番号以上のデータを取得する (greater than or equal)
gt | optional | false | 指定されたシーケンス番号より大きなデータを取得する (greater than)
lte | optional | false | 指定されたシーケンス番号以下のデータを取得する (less than or equal)
lt | optional | false | 指定されたシーケンス番号より小さなデータを取得する (less than)
limit | optional | 1000 | 取得するデータ数, 最大50000
seqbreak | optional | 0 | 0 - 同一シーケンス番号を分割しない, 1 - 同一シーケンス番号を考慮せず分割する
reverse | optional | 0 | 0 - シーケンス番号の古い順, 1 - シーケンス番号の新しい順
unconf | optional | 0 | 0 - 未確認の受信トランザクションを含めない, 1 - 未確認の受信トランザクションを含む

### Headers
`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | | | [addr1のUTXO情報, addr2のUTXO情報, ..., addrNのUTXO情報]

※各UTXO情報の形式は [UTXO情報取得 /utxo](#utxo-utxo) を参照

## アドレス取引履歴取得 /addrlog
アドレスの取引履歴取得を取得する。

### HTTP Request
`GET https://blockstor.bitzeny.jp/api/addrlog/{addr}?gte=seq&gt=seq&lte=seq&lt=seq&limit=num&seqbreak=flag&reverse=flag`

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
{addr} | required | false | 情報取得対象のアドレス
gte | optional | false | 指定されたシーケンス番号以上のデータを取得する (greater than or equal)
gt | optional | false | 指定されたシーケンス番号より大きなデータを取得する (greater than)
lte | optional | false | 指定されたシーケンス番号以下のデータを取得する (less than or equal)
lt | optional | false | 指定されたシーケンス番号より小さなデータを取得する (less than)
limit | optional | 1000 | 取得するデータ数, 最大50000
seqbreak | optional | 0 | 0 - 同一シーケンス番号を分割しない, 1 - 同一シーケンス番号を考慮せず分割する
reverse | optional | 1 | 0 - シーケンス番号の古い順, 1 - シーケンス番号の新しい順

### Headers
`Content-Type: application/json`

### Response
> 取引履歴のサンプル

```json
{
  "err": 0,
  "res": [
    {
      "sequence": 3197963,
      "type": 1,
      "txid": "fbe61cebd32dcec39f77fe2e387983b06eda74ee0a1de39092eb70e35f0252f8",
      "value": 3125000000,
      "height": 1617558,
      "time": 1562203790
    },
    {
      "sequence": 3197958,
      "type": 0,
      "txid": "51c3ad6863639c32ec3562ad3c5c505037f086948223bb646180f9a54dcfc01c",
      "value": 12500013266,
      "height": 1617556,
      "time": 1562203551
    },
    ...
  ]
}
```

Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | [ { | |
 | | sequence | Blockstor シーケンス番号
 | | type | 送受信タイプ 0 - 送信, 1 - 受信, 2 - 受信 (受信アドレスが複数存在する場合)
 | | txid | トランザクションID
 | | value | 数量 (Satoshi)
 | | height | ブロック高
 | | time | 時間 (ブロックの時間)
 | }, ... ] | |

## 複数アドレス取引履歴取得 /addrlogs
複数のアドレスの取引履歴取得を取得する。

### HTTP Request
`POST https://blockstor.bitzeny.jp/api/addrlogs?gte=seq&gt=seq&lte=seq&lt=seq&limit=num&seqbreak=flag&reverse=flag`

### Request Body
`{addrs: [addr1, addr2, ..., addrN]}`

Field | Field Type | Requirements | Default | Description
----- | ---------- | ------------ | ------- | -----------
addrs | array[string] | required | false | 情報取得対象のアドレスを列挙する

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
gte | optional | false | 指定されたシーケンス番号以上のデータを取得する (greater than or equal)
gt | optional | false | 指定されたシーケンス番号より大きなデータを取得する (greater than)
lte | optional | false | 指定されたシーケンス番号以下のデータを取得する (less than or equal)
lt | optional | false | 指定されたシーケンス番号より小さなデータを取得する (less than)
limit | optional | 1000 | 取得するデータ数, 最大50000
seqbreak | optional | 0 | 0 - 同一シーケンス番号を分割しない, 1 - 同一シーケンス番号を考慮せず分割する
reverse | optional | 1 | 0 - シーケンス番号の古い順, 1 - シーケンス番号の新しい順

### Headers
`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | | | [addr1の取引履歴, addr2の取引履歴, ..., addrNの取引履歴]

※各取引履歴の形式は [アドレス取引履歴取得 /addrlog](#addrlog) を参照


## mempool情報取得 /mempool

mempoolの情報を取得する。

### HTTP Request

`GET https://blockstor.bitzeny.jp/api/mempool`

### Headers

`Content-Type: application/json`

### Response
> mempool情報のサンプル

```json
{
  "err": 0,
  "res": [
    {
      "txid": "e8c337bd0f97173ba14818909af4f7475e24022459b7996a0ab6293844befea2",
      "addrs": {
        "ZpwKdjyecgmE2Lbj4c2JbFB23qMiur3wE7": {
          "0": 2125765054
        },
        "ZsuqBgGq2qLAFK5Q2C85d6iHaivZLmjQK9": {
          "0": 9375000991
        },
        "ZpmsQZEBFE1Riwx5zRD14trjwCeu6iiALx": {
          "1": 10000261229
        },
        "ZycSS23vKvkp1RWMm4FBsLWTg5FS9k4chc": {
          "1": 1500504133
        }
      }
    },
    {
      "txid": "d97cf8b287dce38936d45be199c93d7ea4d9316e1d5f0a8bbbdde14e3d360d2d",
      "addrs": {
        "ZgpNLbUPdbEECiZBdRJc84jinkLnbspUxy": {
          "0": 268892966
        },
        "ZxStkdMgdDMohuQk58id95gFPJqgd1k8SL": {
          "1": 74843370
        },
        "ZgvoDe2oBeDKAxD9DkwWZqt6g75Mg6LmuY": {
          "1": 194049366
        }
      }
    }
  ]
}
```

Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中
res | | | [トランザクション情報1, トランザクション情報2, ..., トランザクション情報N]

### トランザクション情報1〜N
Attribute |   |   | Description
--------- | - | - | -----------
txid | | | トランザクションID
addrs | | |
 | アドレス1 | |
 | | "0" | 送信数量
 | | "1" | 受信数量
 | | "2" | 受信数量 (受信アドレスが複数存在する場合)
 | アドレス2 ... | |
 | アドレスN | |
 | | |

※addrs内は、トランザクション内に存在するアドレスがキー名として列挙される。

※送信数量、受信数量の "0", "1", "2" は値がある場合のみ存在する。

※トランザクション内のアドレスに複数送受信がある場合は送信、受信ごとに数量がまとめられる。

## シーケンス位置取得 /marker

シーケンスの読み出し済み位置を取得する。

### HTTP Request

`GET https://blockstor.bitzeny.jp/api/marker/{apikey}`

### Headers

`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 3 - ロールバック中, 4 - ロールバックが発生した, 5 - APIキーが不正
res | | | Blockstor シーケンス番号

※[Blockstor Config - 設定](#blockstor-config) の apikey を設定してください。

## シーケンス位置設定 /marker

シーケンスの読み出し済み位置を設定する。

### HTTP Request

`POST https://blockstor.bitzeny.jp/api/marker`

### Request Body
`{apikey: apikey, sequence: sequence}`

Field | Field Type | Requirements | Default | Description
----- | ---------- | ------------ | ------- | -----------
apikey | string | required | false | apikey を指定する
sequence | number | required | false | 同期済みの Blockstor シーケンス番号を指定する

### Headers

`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 3 - ロールバック中, 4 - ロールバックが発生した, 5 - APIキーが不正, 8 - 指定したシーケンス番号の値が大きく不正
res | | | err が 4 の場合のみ、同期済みの Blockstor シーケンス番号

※[Blockstor Config - 設定](#blockstor-config) の apikey を設定してください。

<aside class="notice">
ロールバックが発生した場合、同期済みの Blockstor シーケンス番号はロールバックが発生した位置に変更されます。
</aside>

## ブロック高取得 /height

現在 Blockstor が同期済みのブロック高を取得する。

### HTTP Request

`GET https://blockstor.bitzeny.jp/api/height`

### Headers

`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗
res | | | 現在のブロック高

## トランザクション送信 /send

トランザクションを送信する。

### HTTP Request

`POST https://blockstor.bitzeny.jp/api/send`

### Request Body
`{rawtx: rawtx}`

Field | Field Type | Requirements | Default | Description
----- | ---------- | ------------ | ------- | -----------
rawtx | string | required | false | トランザクションデータ

### Headers

`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 6 - ビジー
res | | | err が 0, 1 の場合、sendRawTransaction の処理結果 {code: code, message: message}

## トランザクション情報取得 /tx

トランザクションの情報を取得する。

### HTTP Request

`GET https://blockstor.bitzeny.jp/api/tx/{txid}`

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
{txid} | required | false | 情報取得対象のトランザクションID

### Headers

`Content-Type: application/json`

### Response
> トランザクション情報のサンプル

```
{
  "err": 0,
  "res": {
    "ins": [
      {
        "value": 3125000519,
        "addrs": [
          "Zzdn5c3mmiWXH3dDTxFDhZSkSYsozfYEGu"
        ]
      },
      {
        "value": 3125000000,
        "addrs": [
          "Zzdn5c3mmiWXH3dDTxFDhZSkSYsozfYEGu"
        ]
      }
    ],
    "outs": [
      {
        "value": 5353905454,
        "addrs": [
          "ZnzJjnN18N6J1KPNm3fqd8HCTWCfAysRDr"
        ]
      },
      {
        "value": 896094684,
        "addrs": [
          "ZxT6Zreb8zptiyiMJbqEmjBxVYDSwTvJPE"
        ]
      }
    ],
    "fee": 381
  }
}
```

Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 1 - 失敗, 2 - 同期中, 3 - ロールバック中, 6 - ビジー
res | | | err が 1 の場合、getRawTransaction の処理結果 {code: code, message: message}、6 の場合はなし、それ以外の場合は下記
 | ins | | 入力 value, addrs のリスト
 | | value | 数量
 | | addrs | アドレスリスト
 | outs | | 出力 value, addrs のリスト
 | | value | 数量
 | | addrs | アドレスリスト
 | fee | | トランザクション手数料

<aside class="notice">
トランザクション情報取得機能を使用する場合、coind に txindex=1 を指定してください。指定しなかった場合も問題なく動作するようにフォールバックを実装していますが、ブロックからトランザクションデータを取り出すので、少しパフォーマンスが悪くなります。
</aside>

## 検索 /search

アドレスやトランザクションを検索する。

### HTTP Request

`GET https://blockstor.bitzeny.jp/api/search/{keyword}`

### Query Parameters
Parameter | Requirements | Default | Description
--------- | ------------ | ------- | -----------
{keyword} | required | false | 検索文字列

### Headers

`Content-Type: application/json`

### Response
Attribute |   |   | Description
--------- | - | - | -----------
err | | | 0 - 成功, 7 - 検索結果が多すぎます
res | | | err が 7 の場合はなし、それ以外の場合は下記
 | addrs | | アドレスリスト
 | txids | | トランザクションIDリスト

<aside class="notice">
先頭文字からの部分一致で検索できます。
</aside>

# Stream API

### WebSocket Endpoint

`wss://blockstor.bitzeny.jp/api/`

> WebSocket クライアントサンプル

```javascript
// nodejs
var WebSocket = require('ws');

var ws = new WebSocket('wss://blockstor.bitzeny.jp:443/api');
ws.on('message', function(msg) {
  console.log(JSON.stringify(JSON.parse(msg), null, '  '));
});

// javascript
var ws = new WebSocket('wss://blockstor.bitzeny.jp:443/api');
ws.onmessage = function(evt) {
  console.log(JSON.stringify(JSON.parse(evt.data), null, '  '));
}
```

```nim
import "websocket.nim"/websocket, asyncdispatch, json

let ws = waitFor newAsyncWebsocketClient("blockstor.bitzeny.jp", Port(443),
  path = "/api", protocols = @[], ssl = true)

proc read() {.async.} =
  while true:
    let (opcode, data) = await ws.readData()
    if opcode == Opcode.Text:
      echo parseJson(data).pretty

asyncCheck read()
runForever()
```

> ブロックデータサンプル1

```
{
  "height": 1617899,
  "hash": "00000009a9392fd700f6b8554a4c71eff54e8c10c7df34efc1472eb39f86d22d",
  "time": 1562235324,
  "addrs": {
    "ZsWVcJq4G7S1dHfQAidRiANXF9LqbGfLzp": {
      "balance": 17728151916773,
      "utxo_count": 5625,
      "vals": [
        {
          "sequence": 3198538,
          "type": 1,
          "value": 3125000000
        }
      ]
    }
  },
  "txs": {
    "3198538": "b4ae9575f67431ae88510991497f2e38fbe1ac815a54e8439f513c3f979331ae"
  }
}
```

> ブロックデータサンプル2


```
{
  "height": 1617911,
  "hash": "0000000339b3824c29c420da9fca5c133b7fda24224805d6b6a94a29df0b1b70",
  "time": 1562236016,
  "addrs": {
    "ZpgUq4hVqe8y4wrCUcKTwS97jt9sK9XXkw": {
      "balance": 123750178263,
      "utxo_count": 40,
      "vals": [
        {
          "sequence": 3198552,
          "type": 1,
          "value": 3093750228
        }
      ]
    },
    "ZdzzJGbxRLSiim9cWvVHetJVxzPW72n6eP": {
      "balance": 4437532773,
      "utxo_count": 142,
      "vals": [
        {
          "sequence": 3198552,
          "type": 1,
          "value": 31250002
        }
      ]
    },
    "ZcanDiZt7uxjsCrPXvN18qbUKanNBFUwey": {
      "balance": 423450288802,
      "utxo_count": 1,
      "vals": [
        {
          "sequence": 3198553,
          "type": 1,
          "value": 423450288802
        }
      ]
    },
    "ZqXuAvdoacwhDd9JaWvj9AYiPXLMQyqQgj": {
      "balance": 2182066255,
      "utxo_count": 512,
      "vals": [
        {
          "sequence": 3198553,
          "type": 1,
          "value": 3000000
        }
      ]
    },
    "Zd3vfCHzr5pUajEwkNy2jGeenJ1k1wBLXD": {
      "balance": 0,
      "utxo_count": 0,
      "vals": [
        {
          "sequence": 3198553,
          "type": 0,
          "value": 423453289032
        }
      ]
    }
  },
  "txs": {
    "3198552": "8d6037071c6d137856f39bbc806e07f6233978a9c94bbcad998b536684ae978a",
    "3198553": "cb5e8476e4a3f7523a4a6c76f0ba79ba5cf508f2b4f429ecc56712f011c040eb"
  }
}
```

> mempoolデータサンプル

```
{
  "mempool": [
    {
      "txid": "cb5e8476e4a3f7523a4a6c76f0ba79ba5cf508f2b4f429ecc56712f011c040eb",
      "addrs": {
        "Zd3vfCHzr5pUajEwkNy2jGeenJ1k1wBLXD": {
          "0": 423453289032
        },
        "ZcanDiZt7uxjsCrPXvN18qbUKanNBFUwey": {
          "1": 423450288802
        },
        "ZqXuAvdoacwhDd9JaWvj9AYiPXLMQyqQgj": {
          "1": 3000000
        }
      }
    }
  ]
}
```

# Satoshi Conversion - 数量変換

Blockstor が扱う数量の単位はすべて Satoshi である。
コイン数量に変換するには、1 / 100000000 倍する必要がある。javascript と nim の変換関数をここに書いておくが、これより美しいコードが書けると思う。

> Satoshi からコイン数量に変換する関数

```javascript
function conv_coin(uint64_val) {
    strval = uint64_val.toString();
    val = parseInt(strval);
    if(val > Number.MAX_SAFE_INTEGER) {
        var d = strval.slice(-8).replace(/0+$/, '');
        var n = strval.substr(0, strval.length - 8);
        if(d.length > 0) {
            return n + '.' + d;
        } else {
            return n;
        }
    }
    return val / 100000000;
}
```

```nim
import strutils

proc convertCoin(val: string): string =
  if val.len > 8:
    result = val[0..^9] & "." & val[^8..^1]
  else:
    result = (parseFloat(val) / 100000000).formatFloat(ffDecimal, 8)
  result.trimZeros()

let json = parseJson(res.body)
echo convertCoin($json["res"]["balance"])
```

# License - ライセンス

MIT License

[https://github.com/zenywallet/blockstor/blob/master/LICENSE](https://github.com/zenywallet/blockstor/blob/master/LICENSE)

# Acknowledgements - 謝辞

Blockstor は BitZeny 関連サービス開発の有志により、[BitZenyフォーラム](https://bitzeny.info/) ([https://bitzeny.info/](https://bitzeny.info/)) で開発がスタートしました。
BitZeny コミュニティの様々な人たちの活動に感謝するとともに、国産の仮想通貨の発展に貢献できればと考えています。

[@zenywallet](https://twitter.com/zenywallet)
