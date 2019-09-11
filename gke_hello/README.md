# Hello GKE

## Arch

```
todoweb
[ingress -> service -> deployment(nginx, web, assets volume)]
↓
todoapi
[service -> deployment(nginx api)]
↓                ↓
mysql-master & slave
[service->statefulset(mysql, data volume) -> Persisitent VolumeClaim]
```

* PersistentVolume: ストレージの実体。
* PersistentVolumeClaim: ストレージを論理的に抽象化したリソース
* StoregeClass: PersistentVolumeが確保するストレージの種類を定義できるリソース。
* StatefulSet: データストアのような永続化するステートフルなアプリの管理に向いたリソース。

## Create Arch on GKE

* Create k8s cluster

```
# k8sクラスタ作成
$ gcloud container clusters create hello-gke --machine-type=n1-standard-1 --num-nodes=3

# Describe
$ gcloud container clusters describe hello-gke

# kubectlに認証情報を設定する
$ gcloud container clusters get-credentials hello-gke
Fetching cluster endpoint and auth data.
kubeconfig entry generated for hello-gke.

# Node取得
$ kubectl get nodes
NAME                                       STATUS   ROLES    AGE   VERSION
gke-hello-gke-default-pool-c261337a-01x7   Ready    <none>   41s   v1.13.7-gke.8
gke-hello-gke-default-pool-c261337a-26cf   Ready    <none>   40s   v1.13.7-gke.8
gke-hello-gke-default-pool-c261337a-2c7v   Ready    <none>   40s   v1.13.7-gke.8
```

* Create MySQL Master / Slaveewq

```
# StorageClass
$ kubectl apply -f storage-class-ssd.yaml
storageclass.storage.k8s.io/ssd created

# Mysql Master
$ kubectl apply -f mysql-master.yaml
service/mysql-master unchanged
statefulset.apps/mysql-master created

# Mysql Slave
$ kubectl apply -f mysql-slave.yaml
service/mysql-slave created
statefulset.apps/mysql-slave created

# Confirm
$ kubectl get pod
NAME             READY   STATUS    RESTARTS   AGE
mysql-master-0   1/1     Running   0          4m59s
mysql-slave-0    1/1     Running   0          20s
mysql-slave-1    0/1     Pending   0          2s

# Init data
$ kubectl exec -it mysql-master-0 init-data.sh 

# Check data on slave
$ kubectl exec -it mysql-slave-0 bash
root@mysql-slave-0:/# mysql -u root -pgihyo tododb -e "SHOW TABLES;"
mysql: [Warning] Using a password on the command line interface can be insecure.
+------------------+
| Tables_in_tododb |
+------------------+
| todo             |
+------------------+
```

* Create todoapi pods

```
$ kubectl apply -f todo-api.yaml
service/todoapi unchanged
deployment.apps/todoapi created

$ kubectl get pod -l app=todoapi
NAME                      READY   STATUS    RESTARTS   AGE
todoapi-cdffbff68-t8z2c   2/2     Running   0          91s
todoapi-cdffbff68-vx2v9   2/2     Running   0          91s
```

* Create todoweb pods

```
$ kubectl apply -f todo-web.yaml
service/todoweb created
deployment.apps/todoweb created

$ kubectl get pod -l app=todoapi
kubectl get pod -l app=todoweb
NAME                      READY   STATUS    RESTARTS   AGE
todoweb-7b9bbcdbb-6hmnk   2/2     Running   0          109s
todoweb-7b9bbcdbb-jzvvf   2/2     Running   0          109s

$ kubectl get svc todoweb
NAME      TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
todoweb   NodePort   10.7.255.129   <none>        80:30063/TCP   2m50s
```

* Create Ingress

```
$ kubectl apply -f ingress.yaml
ingress.extensions/ingress created

$ kubectl get ingress
NAME      HOSTS   ADDRESS          PORTS   AGE
ingress   *       34.102.211.122   80      2m53s
```
