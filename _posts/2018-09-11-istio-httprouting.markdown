---
layout: post
title: "Istioでリクエストパスを使ったHTTPルーティングを設定する"
date: 2018-09-11 07:43:36 +0900
comments: false
categories: [Kubernetes, helm, Istio]
image:
  feature: posts/istio-httprouting/title.png
---

Istioにはリクエストパスごとに割り振るサービスを設定することができます。
例えば、`/api/v1/hoge` はAのサービスに、`/api/v2/fuga` はBのサービスに割り振るといった設定です。

これらの機能について、前回の記事『 [helmパッケージ化されたアプリをKubernetes+Istioを使って公開する]() 』で作った環境とhelmパッケージを使って実現方法をまとめていきます。

まず前回のおさらいですが、今の所全てのリクエストが`foolish-penguin-sample`か、`peeking-rabbit-sample`のサービスに流れる設定になっています。

```shell
for i in `seq 0 9` ; do curl 127.0.0.1.xip.io ; done
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss40.png){: .terminal-ss}


もちろんこれは`/hoge/fuga`に対するアクセスにも適用されます。

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss41.png){: .terminal-ss}

今回はこの`/hoge/`配下のに対するリクエストを他のサービスに割り振る設定をしていきます。


シンプルなHTTPルーティングの実現
--------

HTTPルーティングは、VirtualServiceのリソースを使って設定します。

まずはルーティング先のServiceを作ります。
前回作成したsampleのチャートが含まれているフォルダに移動し、新規にデプロイします。
今回も説明の都合上、リリース名を`angry-swan`に固定します。

```shell
cd ./sample
helm install --name=angry-swan .
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss42.png){: .terminal-ss}


次にsample-gatewayのフォルダに移動し、VirtualServerに設定を書き加えます。

```shell
cd ../sample-gateway
vi templates/virtualservice.yaml
vi Chart.yaml
git diff
git add .
git commit -m "Added http routing for angry-swan."
git tag 0.2.0
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss43.png){: .terminal-ss}


書き加えた設定を反映させるため、sample-gatewayをアップグレードします

```shell
helm upgrade exciting-lemur .
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss44.png){: .terminal-ss}


これで期待通りのルーティングになっているか確認していきます。
確認には前回同様のcurlに加えて、今回確認したい`/hoge/fuga`に対するリクエストを実行し、さらに無関係な`/moke`にリクエストを実行して確認していきます。

```shell
for i in `seq 0 9` ; do curl 127.0.0.1.xip.io ; done
curl 127.0.0.1.xip.io/hoge/fuga
curl 127.0.0.1.xip.io/moke
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss45.png){: .terminal-ss}


結果の通り、`/`へのリクエストは`foolish-penguin-sample`か`peeking-rabbit-sample`に、`/hoge/fuga`は`angry-swan-sample`に、`/moke`は`/`のマッチが適用されて`foolish-penguin-sample`か`peeking-rabbit-sample`に振り分けられている事が確認できました。

ポッドに対するログも見ていきます。

```shell
export POD_NAME=$(kubectl get pods --namespace default -l "app=sample,release=angry-swan" -o jsonpath="{.items[0].metadata.name}")
kubectl logs -f $POD_NAME sample | grep -v kube-probe
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss46.png){: .terminal-ss}


ログを読むと`angry-swan-sample`からのリクエストがそのまま`/hoge/fuga`に対して実行されていることが確認できます。
このログのフォローも`Ctrl+c`で終了する事ができます。


パスのリライトを含んだHTTPルーティング
-------

サービスに対してリクエストをルーティングする際、パスを書き換えて実行する機能がIstioには備わっています。
例えば、`/api/hoge/v1/fuga`はAのサービスにルーティングするけど、その際にリクエストパスを`/api/fuga`に変更してサービスに渡すといった事ができるようになります。

今回は例の通り、`/api/hoge/v1/fuga`に対するリクエストを`angry-swan-sample`に、パスを`/api/fuga`に変更しつつ渡す設定をします。

リライトの設定はVirtualServiceに対して入れていきます。

```shell
vi templates/virtualservice.yaml
vi Chart.yaml
git diff
git add .
git commit -m "Added path rewrite from '/api/hoge/v1/fuga' to '/api/fuga'."
git tag 0.2.1
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss47.png){: .terminal-ss}


その後書き加えた設定を反映させるため、sample-gatewayをアップグレードします

```shell
helm upgrade exciting-lemur .
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss48.png){: .terminal-ss}


実際にリクエストを投げて`angry-swan-sample`にリクエストが流れていることを確認します。

```
curl 127.0.0.1.xip.io/api/hoge/v1/fuga
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss49.png){: .terminal-ss}


先ほどと同様にポッドに対するログも見ていきます。

```shell
export POD_NAME=$(kubectl get pods --namespace default -l "app=sample,release=angry-swan" -o jsonpath="{.items[0].metadata.name}")
kubectl logs -f $POD_NAME sample | grep -v kube-probe
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss50.png){: .terminal-ss}


ログの最後の一行（最新のリクエストに対するログ）を読むとリクエストパスがリライトされ、`/api/fuga`にリクエストが飛んでいる事が確認できます。

加えてその配下に対するアクセスに対してもリライトされた後に実行されます。

```shell
curl 127.0.0.1.xip.io/api/hoge/v1/fuga/aaaa
kubectl logs -f $POD_NAME sample | grep -v kube-probe
```

![コマンド実行結果]({{ site.url }}/images/posts/istio-httprouting/ss51.png){: .terminal-ss}


上記の例では`/api/hoge/v1/fuga/aaa`に対するリクエストがリライトされ、`/api/fuga/aaa`に飛んでいる事が確認できます。


今回は以上のように、Istioでリクエストパスを使ったHTTPルーティングを設定し、動きを確認する事ができました。
