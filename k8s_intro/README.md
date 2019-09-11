
## Pod

* Podはコンテナの集合体の単位で少なくとも1つのコンテナを持つ。

* Create Pod

```
$ kubectl apply -f simple-pod.yaml 
pod/simple-echo created
```

* Get Pod

```
$ kubectl get pods
NAME          READY   STATUS    RESTARTS   AGE
simple-echo   2/2     Running   0          10s
```

* login container

```
$ kubectl exec -it simple-echo sh -c nginx
```

* log

```
$ kubectl logs -f simple-echo -c echo
```

* Delete Pod

```
$ kubectl delete pod simple-echo
or
$ kubectl delete -f simple-pod.yaml
pod "simple-echo" deleted
```

## ReplicaSet

* ReplicaSetは、同じ仕様のPodを複数生成・管理するためのリソース

* Create ReplicaSet

```
$ kubectl apply -f simple-replicaset.yaml
replicaset.apps/echo created
```

* Get Pods

```
$ kubectl get pod
NAME         READY   STATUS    RESTARTS   AGE
echo-675lh   2/2     Running   0          27s
echo-kwzlj   2/2     Running   0          27s
echo-sr56n   2/2     Running   0          27s
```

* Remove ReplicaSet

```
$ kubectl delete -f simple-replicaset.yaml
replicaset.apps "echo" deleted
```

## Deployment

* Deploymentはアプリケーションデプロイの基本単位となるリソース。
* 新しいバージョンへのPodへの入れ替え、以前のバージョンへのPodのロールバック、指定されたPod数の確保などを実行する
* ReplicaSetより上位のリソース（Deployment - create -> ReplicaSet - create -> Pods)

* Apply deployment

```
$ kubectl apply -f sample-deployment.yaml --record
```

* Get Resources

```
$ kubectl get pod,replicaset,deployment --selector app=echo 
NAME                        READY   STATUS    RESTARTS   AGE
pod/echo-679c46ddf9-fq2hj   2/2     Running   0          30s
pod/echo-679c46ddf9-ssj8r   2/2     Running   0          30s
pod/echo-679c46ddf9-vwcmh   2/2     Running   0          30s

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.extensions/echo-679c46ddf9   3         3         3       30s

NAME                         READY   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/echo   3/3     3            3           30s
```

* Get Deployment Revision History

```
$ kubectl rollout history deployment echo
deployment.extensions/echo 
REVISION  CHANGE-CAUSE
1         kubectl apply --filename=sample-deployment.yaml --record=true
```

* Show deployment revision detail

```
$ kubectl rollout history deployment echo --revision=1
```

* Rollback ddeployment revision

```
$kubectl rollout undo deployment echo
deployment.extensions/echo rolled back
```

## Service

* Serviceは、Podの集合(主にReplicaSet)に対する経路やサービスディスカバリを提供する


```
$ kubectl apply -f simple-service.yaml
service/echo created

$ kubectl get svc echo
NAME   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
echo   ClusterIP   10.101.121.245   <none>        80/TCP    11s
```

#### クラスタ内の命名規則

`サービス名.ネームスペース.svc.local`
`svc.local`は省略が可能
同じネームスペースなら`ネームスペース`も省略が可能

#### サービスの種類

* ClusterIP Service: デフォルトのサービス。k8sクラスタ上の内部IPアドレスにサービスを公開できる。あるPodから別のPodへアクセスができるようになる
* NodePort Service: クラスタ外からアクセスできるサービス。内部的にはClusterIPを作っているが、グローバルなポートをあけていることで外からもアクセスできるようになる
* LoadBalancer Service: クラウドプラットフォームで提供されているLBと連携するためのもの。ローカルのk8s環境では利用できない
* ExternalName Service: k8sクラスタ内から外部のホストを解決するためのエイリアスを提供する。


## Ingress

* Ingressは、ServiceのL7層レベルのk8sクラスタの外への公開、VirtualHostやパスベースでの高度なHTTPルーティングをおこなう
* 素の状態のローカルk8s環境ではIngressを使ったServicの公開はdけいないので、nginx_ingress_controllerを挟む必要がある

```
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.16.2/deploy/mandatory.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/nginx-0.16.2/deploy/provider/cloud-generic.yaml

$ kubectl -n ingress-nginx get service,pod
NAME                           TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
service/default-http-backend   ClusterIP      10.110.135.60   <none>        80/TCP                       83s
service/ingress-nginx          LoadBalancer   10.97.254.170   localhost     80:31339/TCP,443:31337/TCP   24s

NAME                                           READY   STATUS    RESTARTS   AGE
pod/default-http-backend-55b84578bf-lmkj8      1/1     Running   0          83s
pod/nginx-ingress-controller-b5d545f8f-d7jj4   1/1     Running   0          83s
```

* Ingress

```
$ kubectl apply -f simple-ingress.yaml
ingress.extensions/echo created

$ kubectl get ingress
NAME   HOSTS              ADDRESS   PORTS   AGE
echo   ch05.gihyo.local             80      28s

$ curl http://localhost -H 'Host: ch05.gihyo.local'
Hello Docker!!
```

## Advanced

* Job

```
$ kubectl apply -f sample-job.yaml
job.batch/pingpong created

$ kubectl logs -l app=pingpong
[Wed Sep 11 12:41:36 UTC 2019] ping!
[Wed Sep 11 12:41:46 UTC 2019] pong!
[Wed Sep 11 12:41:31 UTC 2019] ping!
[Wed Sep 11 12:41:41 UTC 2019] pong!
[Wed Sep 11 12:41:33 UTC 2019] ping!
[Wed Sep 11 12:41:43 UTC 2019] pong!

$ kubectl get pod -l app=pingpong
NAME             READY   STATUS      RESTARTS   AGE
pingpong-5whvn   0/1     Completed   0          5m21s
pingpong-94nxw   0/1     Completed   0          5m21s
pingpong-g4jxt   0/1     Completed   0          5m21s
```

* CronJob

```
$ kubectl apply -f simple-cronjob.yaml
cronjob.batch/pingpong-batch created

$ kubectl logs -l app=pingpong-batch
[Wed Sep 11 12:52:02 UTC 2019] ping!
[Wed Sep 11 12:52:12 UTC 2019] pong!
[Wed Sep 11 12:53:02 UTC 2019] ping!
[Wed Sep 11 12:53:12 UTC 2019] pong!

$ kubectl get job -l app=pingpong-batch
NAME                        COMPLETIONS   DURATION   AGE
pingpong-batch-1568206320   1/1           13s        111s
pingpong-batch-1568206380   1/1           13s        51s
```

* Secret

```
$ echo "myusername:$(openssl passwd -quiet -crypt mypassword)" | base64
bXl1c2VybmFtZTpBM0ZZLmVvREQxRE5BCg==

$ kubectl apply -f simple-nginx-secret.yaml
secret/nginx-server created

$ kubectl get secret nginx-secret
NAME           TYPE     DATA   AGE
nginx-secret   Opaque   1      13m

$ kubectl apply -f simple-nginx-basicauth.yaml
service/basicauth created
deployment.apps/basicauth created

$ kubectl get service basicauth
NAME        TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
basicauth   NodePort   10.104.139.198   <none>        80:30060/TCP   37s

$ curl http://127.0.0.1:30060

$ curl -i --user myusername:mypassword http://127.0.0.1:30060
```
