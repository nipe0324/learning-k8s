
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
* ReplicaSetより上位のリソース（Deployment - create -> ReplicaSet - create -> Pods)