# Hello Istio

Original: https://gitlab.com/cloudnative_impress/tutorial

## Arch

```
<Data Plane>
ServiceA             ServiceB
 ↓                      ↑
Envoy --------------> Envoy

<Control Plane>
Mixer ポリシーのチェック、テレメトリ情報の送信
Pilot Config情報の配布(Envoy用)
Galley Config情報の配布
Citadel TLS証明書の配布、更新
```

## Istionのインストール

* Helmのリポジトリとして`istio.io`を追加

```
$ helm repo add istio.io https://storage.googleapis.com/istio-release/releases/1.3.1/charts/
```

* Helm と Tillerでインストール

```
$ kubectl apply -f install/kubernetes/helm/helm-service-account.yaml
serviceaccount/tiller created
clusterrolebinding.rbac.authorization.k8s.io/tiller created

$ helm install install/kubernetes/helm/istio-init --name istio-init --namespace istio-system
NAME:   istio-init
LAST DEPLOYED: Wed Oct  2 21:25:54 2019
NAMESPACE: istio-system
STATUS: DEPLOYED

RESOURCES:
==> v1/ClusterRole
NAME                     AGE
istio-init-istio-system  0s

==> v1/ClusterRoleBinding
NAME                                        AGE
istio-init-admin-role-binding-istio-system  0s

==> v1/ConfigMap
NAME          DATA  AGE
istio-crd-10  1     0s
istio-crd-11  1     0s
istio-crd-14  1     0s

==> v1/Job
NAME                                   COMPLETIONS  DURATION  AGE
istio-init-crd-10-master-latest-daily  0/1          0s        0s
istio-init-crd-11-master-latest-daily  0/1          0s        0s
istio-init-crd-14-master-latest-daily  0/1          0s        0s

==> v1/Pod(related)
NAME                                         READY  STATUS             RESTARTS  AGE
istio-init-crd-10-master-latest-daily-dskzk  0/1    ContainerCreating  0         0s
istio-init-crd-11-master-latest-daily-m5lkk  0/1    ContainerCreating  0         0s
istio-init-crd-14-master-latest-daily-rtz4f  0/1    ContainerCreating  0         0s

==> v1/ServiceAccount
NAME                        SECRETS  AGE
istio-init-service-account  1        0s

# 確認する
$ kubectl get crds | grep 'istio.io' | wc -l
23

# istioインストール
$ helm install install/kubernetes/helm/istio \
  --name istio \
  --namespace istio-system \
  --set tracing.enabled=true \
  --set pilot.traceSampling=100.0
```

## Istioによるトラフィック制御

* VirtualService トラフィックの搬送時に通信経路にけるメッシュの振る舞いを定義。特定経路でリトライを有効にしたり、タイムアウト時間を設定する
* DestinationRule トラフィック搬送が完了したあと実際にPodに届くまでの振る舞いを定義。Podのレプリカ数が複数ある場合のロードバランシングのアルゴリズムやコネクションプーリングのサイズの設定など
* ServiceEntry マニュアルでトラフィックの配送先となる宛先を作成するリソース。サービスメッシュの外部にあるAPIに対してリクエストを送信したい場合など
* Gateway サービスメッシュの外からのHTTP/TCPトラフィックを受け入れる入り口を定義するリソース。
* Sidecar Envoyの振る舞いを設定するリソース。あるnamespaceにあるEnvoyに対して、特定の外部サービスへのリクエストを許可する設定を追加するなど

## Sock Shop with Service Mesh

* namespaceの作成

```
$ kubectl create namespace sock-shop-with-mesh
namespace/sock-shop-with-mesh created

$ kubectl label namespace sock-shop-with-mesh istio-injection=enabled
namespace/sock-shop-with-mesh labeled
```

* sock shopのデプロイ

```
$ kubectl -n sock-shop-with-mesh apply -f 10-sock-shop-complete-demo.yaml
deployment.extensions/carts-db created
service/carts-db created
deployment.extensions/carts created
...

# サービスのPodにEnvoyが含まれているか確認
$ POD_NAME_FRONT_END=$(kubectl -n sock-shop-with-mesh get pod -l app=front-end,version=v1 -o jsonpath="{.items[0].metadata.name}")
$ kubectl -n sock-shop-with-mesh get pod $POD_NAME_FRONT_END -o jsonpath="{.spec.containers[*].name}"
front-end istio-proxy
```

* トラフィック制御用のリソースの適用

```
$ kubectl -n sock-shop-with-mesh apply -f 20-sock-shop-gateway.yaml
gateway.networking.istio.io/sock-shop-gateway created

$ kubectl -n sock-shop-with-mesh apply -f 30-sock-shop-destination-rule.yaml
destinationrule.networking.istio.io/front-end created
destinationrule.networking.istio.io/carts created
destinationrule.networking.istio.io/catalogue created
destinationrule.networking.istio.io/user created
destinationrule.networking.istio.io/payment created
destinationrule.networking.istio.io/orders created

$ kubectl -n sock-shop-with-mesh apply -f 40-sock-shop-virtual-service.yaml
virtualservice.networking.istio.io/sock-shop created
virtualservice.networking.istio.io/carts created
virtualservice.networking.istio.io/catalogue created
virtualservice.networking.istio.io/user created
virtualservice.networking.istio.io/payment created
virtualservice.networking.istio.io/orders created

$ kubectl -n istio-system get svc istio-ingressgateway
NAME                   TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
istio-ingressgateway   LoadBalancer   10.104.215.39   localhost     15020:32196/TCP,80:31380/TCP,443:31390/TCP,31400:31400/TCP,15029:31287/TCP,15030:32522/TCP,15031:30604/TCP,15032:32582/TCP,15443:31746/TCP   32m
```

## リトライ

* IstioによるFault injection

```
# 80%の確率でカートへのリクエストを失敗させる
$ kubectl -n sock-shop-with-mesh apply -f 50-sock-shop-virtual-service-failt.yaml
virtualservice.networking.istio.io/carts configured
```

* Isitoのよるリトライ

```
# リトライを実施
$ kubectl -n sock-shop-with-mesh apply -f 51-sock-shop-virtual-service-retry.yaml
virtualservice.networking.istio.io/sock-shop configured
```

* 設定を戻しておく

```
$ kubectl -n sock-shop-with-mesh apply -f 40-sock-shop-virtual-service.yaml 
```

## タイムアウト

* IstioによるFault injection

```
# 50%の確率で5秒間の通信遅延が起きる
$ kubectl -n sock-shop-with-mesh apply -f 60-sock-shop-virtual-service-delay.yaml 
```

* Isitoによるタイムアウト設定

```
# 1秒でタイムアウトをさせる
$ kubectl -n sock-shop-with-mesh apply -f 61-sock-shop-virtual-service-timeout.yaml
```
* 設定を戻しておく

```
$ kubectl -n sock-shop-with-mesh apply -f 40-sock-shop-virtual-service.yaml 
```

## サーキットブレーカー（通信を遮断する）

想定以上のアクセス検知や、一定以上の割合のエラーが発生したときなどにサービスに対する通信を遮断する
障害の起きたサービスを負荷から解放して復旧しやすくしたり、過負荷によってサービスがダウンすることを防止する

```
# TCPコネクションが1つ以上になり、並列にリクエストが送信されると通信遮断
$ kubectl -n sock-shop-with-mesh apply -f 70-sock-shop-destination-rule-circuit-breaker.yaml 
```

* Istioの負荷テストルールのfortio

```
# デプロイ
$ kubectl -n sock-shop-with-mesh apply -f fortio-deploy.yaml

# fortioの動作確認
$ FORTIO_POD=$(kubectl -n sock-shop-with-mesh get pod | grep fortio | awk '{ print $1 }')
# 2並列で20回のリクエストを送信
$ kubectl -n sock-shop-with-mesh exec -it $FORTIO_POD \
  -c fortio /usr/bin/fortio \
  -- load -c 2 -qps 0 -n 20 -loglevel Warning \
  http://carts/items

# 2並列で40回のリクエストを送信
$ kubectl -n sock-shop-with-mesh exec -it $FORTIO_POD \
  -c fortio /usr/bin/fortio \
  -- load -c 4 -qps 0 -n 40 -loglevel Warning \
  http://carts/items

Sockets used: 5 (for perfect keepalive, would be 4)
Code 200 : 39 (97.5 %)
Code 503 : 1 (2.5 %)
```

* 負荷ツールの削除

```
$ kubectl -n sock-shop-with-mesh delete -f fortio-deploy.yaml
```

## 分散トレーシング (Jaeger)

* トレース 連鎖的に実行される一連のサービスの呼び出しを追跡して記録した情報をさす。
* スパン ここのサービスがリクエストをうけ、それに応答するまでの単位をさす。
* スパンコンテキスト 複数のスパンの連鎖的なサービス呼び出しを関連づけるための識別子となる情報。

```
# JaegerのGUIをNodePortで公開
$ kubectl -n istio-system expose deployment istio-tracing \
  --name=jaeger-query-nodeport \
  --type="NodePort" \
  --port 16686

$ kubectl -n istio-system get svc jaeger-query-nodeport -o jsonpath='{.spec.ports[0].nodePort}'

$ open http://localhost:[PORT]
```


## 片付け

```
$ kubectl delete namespace sock-shop-with-mesh
$ helm delete --purge istio 
$ helm delete --purge istio-init
$ kubectl delete namespace istio-system
```