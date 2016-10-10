---
layout: post
title: "Azure IoT-HUBにRubyからPOSTしてみる"
date: 2016-01-03 15:33:47 +0900
comments: false
categories: 
---


[Azure IoT-Hub](https://azure.microsoft.com/ja-jp/services/iot-hub/) につなぐときに、WindowsでVisualStudioが使えるか、 [azure-iot-sdks](https://github.com/Azure/azure-iot-sdks/tree/master) が使える言語環境でのTipsはちょこちょこ見かけたけど、Linux上のRubyから繋ぐときのサンプルは見かけなかったので備忘録として書いてみるなど。 Azure IoT-HubはHTTPS, AMQPS, MQTTをサポートしいますが、今回はHTTPSでの接続を試みます。


※ この記事を書いている時点でAzure IoT-HUBプレビュー版となっており、アカウント登録することで無料でトライアル利用することができました。


デバイスの登録
--------

デバイスの登録にはWindows環境ではDeviceExplorerが使えますが、今回は[node.js](https://nodejs.org/en/)で動く[iothub-explorer](https://www.npmjs.com/package/iothub-explorer)を使ってコマンドラインでデバイス登録をします。

記事を書いている時点で[iothub-explorer](https://www.npmjs.com/package/iothub-explorer)は1.0.0-preview.9で、これを使うためには[node.jsのv0.12系](https://nodejs.org/dist/latest-v0.12.x/)が必要だったので、ダウンロード、インストールしました。

その後npmコマンドで[iothub-explorer](https://www.npmjs.com/package/iothub-explorer)をインストールします

```sh
$ npm install -g iothub-explorer@latest
```

次に[Azureポータル](https://portal.azure.com)から`iothubownerポリシー`の`Connection String`を取得します。

その後取得した`Connection String`と`登録したいデバイスID`をINPUTとして、下記のコマンドで登録します。

```sh
$ iothub-explorer "{取得したConnectionString}" create {登録したいデバイスID} --connection-string
```

このコマンドの実行結果として出力される`登録したデバイスのConnectionString`を使って、認証情報を生成し、データをパブリッシュします。


認証情報の生成
--------

IoT-Hubの認証にはSASトークンという認証情報を使っています。HTTPSの場合はAuthorizationヘッダに下記のように記載します。

```
Authorization: SharedAccessSignature sig={SIGNATURE}&se={EXPIRY}&sr={IOT_HUB_HOST}/devices/{DeviceID}
```

またINPUTとしては先ほどの`登録したデバイスのConnectionString`を利用します。[Azureポータル](https://portal.azure.com)の`deviceポリシー`のものではありません。

```
HostName={IOT_HUB_HOST};DeviceId={DEVICE_ID};SharedAccessKey={DEVICE_KEY}
```

SASトークンとConnectionStringの間で共通の部分である`{IOT_HUB_HOST}`及び`{DEVICE_ID}`はそのままの値を当てはめることで利用できます。 `{EXPIRY}`及び`{SIGNATURE}`は生成してあげる必要があります。

`{EXPIRY}`についてはこのトークンの有効期限を秒で指定するもののようで、例えば10分間の場合は`UNIXタイム(秒)+600`を指定します。

`{SIGNATURE}`はかなり複雑で、下記の手順で生成します。

1. `{DEVICE_KEY}`をBASE64でデコードする
2. UTF-8の文字列として、`{IOT_HUB_HOST}/devices/{DEVICE_ID}\n{EXPIRY}`という文字列を作成する
3. 1を鍵、2を計算対象の文字列としてHMAC-SHA256でハッシュ化する
4. 3の出力をBASE64エンコードする
5. 4の出力をURLエンコードする

この結果を`{SIGNATURE}`として利用し、Authorizationヘッダを構築します。


データの登録
--------

データの登録は下記のようなHTTPパケットを利用して行うようです。

```
POST https://{IOT_HUB_HOST}/devices/{DEVICE_ID}/messages/events?api-version=2015-08-15-preview HTTP/1.1
Content-Type: application/json
Authorization: SharedAccessSignature sig={SIGNATURE}&se={EXPIRY}&sr={IOT_HUB_HOST}/devices/{DeviceID}
Accept: */*
Host: {IOT_HUB_HOST}
Content-Length: 50

{"name":"web","config":{"url":"hogehogehogehoge"}}
```

Content-Lengthヘッダ及びリクエストボディはサンプルです。

Rubyでベタ書きするとこんな感じ

```ruby
require 'openssl'
require 'cgi'
require 'base64'
require 'time'
require 'net/https'
require 'json'


CONNECTION_STRING = "{デバイス登録時に取得したConnectionString}"
DATA = {name: "web", config: {url: "hogehogehogehoge"}}


CONNECTION_STRING.split(";").each do |elm|
  key = elm[0, elm.index('=')].gsub(/([A-Z])/, '_\1').gsub(/^_/, '').downcase
  self.instance_variable_set(
    "@#{key}".to_sym,
    elm[(elm.index('=') + 1), elm.length]
  )
end

expiry = Time.now.to_i + 60 * 10

raw = OpenSSL::HMAC.digest(
  'sha256',
  Base64.strict_decode64(@shared_access_key),
  "#{@host_name}/devices/#{@device_id}\n#{expiry}".encode(Encoding::UTF_8)
)
signature = CGI.escape(Base64.encode64(raw).strip)

sas_header = format(
  "SharedAccessSignature sig=%s&se=%s&sr=%s",
  signature,
  expiry,
  "#{@host_name}/devices/#{@device_id}"
)

request = Net::HTTP::Post.new(
  "https://#{@host_name}/devices/#{@device_id}/messages/events?api-version=2015-08-15-preview",
  { 
    'Content-Type'  => 'application/json',
    'Authorization' => sas_header
  }
)
request.body = DATA.to_json

response = nil
https = Net::HTTP.new(@host_name, 443)
https.use_ssl = true
https.start {
  response = https.request(request)
}

p response
```

レスポンスコードに204が返ってきますが、データ自体は登録できています。
登録件数は[Azureポータル](https://portal.azure.com)で確認することができます。

