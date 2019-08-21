
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
