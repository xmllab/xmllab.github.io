---
layout: post
title: "helmパッケージ化されたアプリをKubernetes+Istioを使って公開する"
date: 2018-09-10 07:43:36 +0900
comments: false
categories: [Kubernetes, helm, Istio]
image:
  feature: posts/istio-with-helm/title.png
---

これまでkubernetes上でアプリを動かす時に基本的にhelmにまとめてデプロイしてきましたが、upgradeするたびに瞬断する問題の解決と、カナリアリリースを利用したブルーグリーンデプロイメントを実現するために、既存のhelmを使ったワークロードにIstioを絡めて利用する方法を書いていきます。

IstioはKubernetes上に展開したマイクロサービスの接続をマネージメントしてくれるコンポーネントで、簡単に言うとIngressの高機能版です。

環境
-------

今回のKubernetes環境としてはDocker for Mac with Kubernetesで動きを見ていきます。

![バージョン確認]({{ site.url }}/images/posts/istio-with-helm/ss01.png){: .terminal-ss}


アプリは、[hashicorp/http-echo](https://hub.docker.com/r/hashicorp/http-echo/) を使って構築していきます。これは簡単なWebアプリケーションで、コンテナの起動引数を使ってレスポンスボディを指定できるものになります。

まずはイメージの動作確認を兼ねてdockerコマンドでコンテナを作成・起動してみます。
実行方法はUSAGEの通りです。

```shell
docker run -p 5678:5678 hashicorp/http-echo -text="hello world"
```

起動するとターミナルは下記のような状態になります。

![docker runの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss02.png){: .terminal-ss}

この状態でブラウザで http://localhost:5678 にアクセスすると、起動時に指定した`hello world`が表示されてているはずです。

![ブラウザでの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss03.png)

docker runで動かしたコンテナを停止する場合はは`Ctrl+C`を使用します。

![Ctrl+Cの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss04.png){: .terminal-ss}


helmパッケージの作成とデプロイ
--------

次にこのイメージをkubernetes上に展開するためにhelm chartを作成して、gitのバージョン管理配下に置きます。

```shell
helm create sample
cd sample/
git init
git add .
git commit -m 'first commit'
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss05.png){: .terminal-ss}

作成したhelmパッケージに対し、先ほどdockerコマンドで立ち上げたように[hashicorp/http-echo](https://hub.docker.com/r/hashicorp/http-echo/)を立ち上げる設定をします。編集するファイルは`Chart.yaml`、`values.yaml`、`templates/deployment.yaml`、`templates/NOTES.txt`の４つです。

```shell
vi Chart.yaml
vi values.yaml
vi templates/deployment.yaml
vi templates/NOTES.txt
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss06.png){: .terminal-ss}

編集内容は下記の通りです。

```shell
git diff
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss07.png){: .terminal-ss}

編集が完了したらコミットし、タグを付けます。

```shell
git commit -a -m "bump version to 1.0.0"
git tag 1.0.0
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss08.png){: .terminal-ss}

作成したhelmパッケージをインストールしていきます。今回は説明の都合上、release nameを`foolish-penguin`に固定します。

```shell
helm install --name=foolish-penguin .
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss09.png){: .terminal-ss}

表示されたNOTESを元にポートフォワーディングを設定してインストールされたpodにアクセスできるかどうか確認します。

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss10.png){: .terminal-ss}

ブラウザで http://localhost:5678 にアクセスし、`"hello world v1"`と表示されればOKです。

![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss11.png)

このポートフォワーディングもdocker run同様`Ctrl+C`で終了することができます。

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss12.png){: .terminal-ss}

ちなみに今回はDeploymentを作成しているので、Deploymentに対してポートフォワーディングを設定することもできます。

```shell
kubectl port-forward deployment/foolish-penguin-sample 8081:5678
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss13.png){: .terminal-ss}
![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss14.png)

Istioを利用する場合はServiceに対してルーティングすることになるので、Serviceへもポートフォワーディングを設定して動作を確認しておきます。

```shell
kubectl port-forward svc/foolish-penguin-sample 8082:80
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss15.png){: .terminal-ss}
![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss16.png)

一通り確認し終わったらCtrl+Cで終了し、１つ上のディレクトリまで戻ります。

```shell
cd ..
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss17.png){: .terminal-ss}


Istioのダウンロードとhelmを使ったインストール
--------

次にIstioの最新版をダウンロードし、ローカルに作成されだディレクトリの中へ移動します。
この記事をまとめている時点では1.0.1が最新になっています。
(参考：https://istio.io/docs/setup/kubernetes/download-release/ )

```shell
curl -L https://git.io/getLatestIstio | sh -
cd istio-*/
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss18.png){: .terminal-ss}

helmを実行するためのサービスアカウントをcluster-adminにアップデートし、Istioのインストールを行います。

```shell
kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
helm init --service-account tiller --upgrade
helm install install/kubernetes/helm/istio --name istio --namespace istio-system
```

Istioを構成する様々なリソースがインストールされます。インストールされたリソースは下記のコマンドで確認することができます。

```shell
kubectl -n istio-system get all
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss19.png){: .terminal-ss}


インストールが完了したら１つ上のディレクトリまで戻ります。

```shell
cd ..
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss17.png){: .terminal-ss}


helmパッケージのIstio対応
--------

ベースとなるhelmパッケージを作成し、同様にGitの管理下に置きます。

```shell
helm create sample-gateway
cd sample-gateway/
git init
git add .
git commit -a -m 'first commit'
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss20.png){: .terminal-ss}

その後不要なファイルを削除し、IstioのGatewayとVirtualServiceを作成します。
この時、VirtualServiceの destination -> host の値を上記で作成したサービスの名前にするのがポイントです。

```shell
{%- raw %}
rm -f templates/*.yaml templates/NOTES.txt

cat << 'EOF' > values.yaml
nameOverride: ""
fullnameOverride: ""

istio:
  hosts:
    - 127.0.0.1.xip.io
EOF

cat << 'EOF' > templates/gateway.yaml
apiVersion: networking.istio.io/v1alpha3
kind: Gateway
metadata:
  name: {{ include "sample-gateway.fullname" . }}
  labels:
    app: {{ include "sample-gateway.name" . }}
    chart: {{ include "sample-gateway.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  selector:
    istio: ingressgateway # use Istio default gateway implementation
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    {{- range .Values.istio.hosts }}
    - {{ . | quote }}
    {{- end }}
EOF

cat << 'EOF' > templates/virtualservice.yaml
apiVersion: networking.istio.io/v1alpha3
kind: VirtualService
metadata:
  name: {{ include "sample-gateway.fullname" . }}
  labels:
    app: {{ include "sample-gateway.name" . }}
    chart: {{ include "sample-gateway.chart" . }}
    release: {{ .Release.Name }}
    heritage: {{ .Release.Service }}
spec:
  hosts:
  {{- range .Values.istio.hosts }}
  - {{ . | quote }}
  {{- end }}
  gateways:
  - {{ include "sample-gateway.fullname" . }}
  http:
  - route:
    - destination:
        host: foolish-penguin-sample
EOF
{% endraw %}
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss21.png){: .terminal-ss}

そしてChart.yamlを編集し、変更内容のコミットとタグ打ちをします。

```shell
vi Chart.yaml
git diff Chart.yaml
git add .
git commit -a -m "bump version to 0.1.0"
git tag 0.1.0
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss22.png){: .terminal-ss}

helmコマンドを使ってデプロイします。今回も説明の都合上、リリース名を`exciting-lemur`に固定します。

```shell
helm install --name=exciting-lemur .
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss23.png){: .terminal-ss}

Istioの自動注入機能を有効にするため、istio-injectionをdefaultネームスーペースに対して有効にする

```shell
kubectl get namespace -L istio-injection
kubectl label namespace default istio-injection=enabled
kubectl get namespace -L istio-injection
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss24.png){: .terminal-ss}

既存のhelmパッケージをIstioで利用するためには上記の注入機能を有効にした上でデプロイする必要があります。アップデートついでにレスポンスボディも変えておきます。

```shell
cd ../sample
vi Chart.yaml
vi templates/deployment.yaml
git diff
git commit -a -m "bump version to 1.0.1"
git tag 1.0.1
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss25.png){: .terminal-ss}

更新したパッケージをデプロイします。

```shell
helm upgrade foolish-penguin .
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss26.png){: .terminal-ss}


ブラウザで http://127.0.0.1.xip.io:80 を開き、更新されたサービスにistio経由で表示されていることを確認します。この時先ほどのアップデートが適用されていることをレスポンスボディを使って判断します。

![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss27.png)


アプリケーションのバージョンアップとカナリアリリース
--------

本来はイメージのタグを変えるが今回はスキップしてレスポンスボディの変更で代用します。

```shell
vi Chart.yaml
git diff
git commit -a -m "bump version to 1.1.0"
git tag 1.1.0
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss28.png){: .terminal-ss}

新しいバージョンのアプリケーションをデプロイします。
この時新しいリリース名をつけてインストールします。
今回も説明の都合上、リリース名を`peeking-rabbit`に固定します。

```shell
helm install --name=peeking-rabbit .
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss29.png){: .terminal-ss}


もちろんこの時点で http://127.0.0.1.xip.io:80 に変化はありません。

![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss27.png)


新しいバージョンのServiceに対してポートフォワーディングを設定し、デプロイできているか確認して見ます。

```shell
kubectl port-forward svc/peeking-rabbit-sample 8083:80
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss30.png){: .terminal-ss}

ブラウザで http://localhost:8083 にアクセスし、`hello world peeking-rabbit`と表示されていれば成功です。

![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss31.png)

一通り確認し終わったらCtrl+Cで終了し、sample-gatewayのディレクトリまで戻ります。

```shell
cd ../sample-gateway
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss32.png){: .terminal-ss}


新しいバージョンにリクエストの半分を割り当てるよう設定を追加し、バージョンを上げます。

```shell
vi Chart.yaml
vi templates/virtualservice.yaml
git diff
git commit -a -m "bump version to 0.1.1"
git tag 0.1.1
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss33.png){: .terminal-ss}

Istioの設定を更新するためにhelmコマンドでデプロイします。

```shell
helm upgrade exciting-lemur .
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss34.png){: .terminal-ss}


その後ブラウザで http://127.0.0.1.xip.io:80 に何度かアクセスし、表示されているページが変化することを確認できたら完了です。

![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss27.png)
![ブラウザの表示結果]({{ site.url }}/images/posts/istio-with-helm/ss35.png)

試しにcurlでHTTPリクエストを10回実行すると、ほぼ指定したウェイト通りに分配されていることが確認できます。

```shell
for i in `seq 0 9` ; do curl 127.0.0.1.xip.io ; done
```

![コマンドの実行結果]({{ site.url }}/images/posts/istio-with-helm/ss36.png){: .terminal-ss}

以上でhelmパッケージのIstio対応とカナリアリリースの設定が完了しました。
