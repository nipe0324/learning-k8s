# Hello Helm

* Helmはk8s Chartsを管理するツール。Chartsは設定済みのk8sリソースのパッケージしたもの。
* 開発環境や本番環境など複数k8sクラスタへのデプロイを簡潔に管理できるようにするツール。

## HelmやChartsの関係

* Helmでデプロイやアップデートを実行する。kubectlではなく
* Chartでパッケージングを行う。


```
helm -> chart [manifest template) -> manifest file -> k8s
```

## Installation & Settings

```
# Install
$ brew install kubernetes-helm

# Setting Helm
$ kubectl config use-context docker-for-desktop
Switched to context "docker-for-desktop".

$ helm init
kube-systemネームスペースにTillerというサーバーアプリケーションがデプロイされる

$ kubectl -n kube-system get service,deployment,pod --selector app=help
kubectl -n kube-system get service,deployment,pod --selector app=helm
NAME                    TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)     AGE
service/tiller-deploy   ClusterIP   10.101.228.205   <none>        44134/TCP   102s

NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/tiller-deploy   1/1     1            1           103s

NAME                                 READY   STATUS    RESTARTS   AGE
pod/tiller-deploy-675cbc8478-ngnpb   1/1     Running   0          102s

$ helm version
Client: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.14.3", GitCommit:"0e7f3b6637f7af8fcfddb3d2941fcc7cbebb0085", GitTreeState:"clean"}
```

## About Helm

* Helm Repository

```
local - ローカルで作成したChartが配置
stable - 安定した品質を持ったChartが配置されるリポジトリ
incubator - stableの要件を満たしていないが、stableへの移行をみすえた開発がされている
```

* Search

```
helm search
```

* Chart Directory

```
{chart_name}
 L templates/
 |  |- xxxx.yaml    各種k8sリソースのマニュフェストテンプレート
 |  |- _helper.tpl  マニフェスト構築に利用されるテンプレートヘルパー
 |  |- NOTE.txt     Chartの利用方法とのドキュメントテンプレート
 |
 |- charts/      依存するChartのディレクトリ
 |- Chart.yaml   Chart情報の定義ファイル
 |- values.yaml  Chartのデフォルトvalueファイル
```

## Use Helm

* Install Redmine

```
$ helm search redmine
NAME            CHART VERSION   APP VERSION     DESCRIPTION                                   
stable/redmine  12.2.5          4.0.4           A flexible project management web application.

$ helm install -f redmine.yaml --name redmine stable/redmine

$ helm ls
NAME    REVISION        UPDATED                         STATUS          CHART           APP VERSION       NAMESPACE
redmine 1               Wed Sep 11 22:52:24 2019        DEPLOYED        redmine-12.2.5  4.0.4             default  

$ kubectl get service,deployment --selector release=redmine
NAME                      TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/redmine           LoadBalancer   10.99.202.9     localhost     80:30972/TCP   4m26s
service/redmine-mariadb   ClusterIP      10.111.82.252   <none>        3306/TCP       4m26s

NAME                            READY   UP-TO-DATE   AVAILABLE   AGE

# Update
$ helm upgrade -f redmine.yaml redmine stable/redmine

# Uninstall
$ helm delete redmine

# History
$ helm ls --all

# Purge
$ helm del --purge redmine
```

## Original Chart

```
$ helm repo list
NAME    URL                                             
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts

# Run local repo server
$ helm serve &

$ helm create echo
Creating echo

$ helm package echo
Successfully packaged chart and saved it to: /.../helm_hello/echo-0.1.0.tgz

$ helm search echo
NAME            CHART VERSION   APP VERSION     DESCRIPTION                
local/echo      0.1.0           1.0             A Helm chart for Kubernetes

$ helm install -f local-echo.yaml --name echo local/echo
```
