# EKS Hello

Create Cluster on EKS and Deploy guestbook app.

## Commands

* Create Cluster on EKS

```
eksctl create cluster \
  --name eks-hello \
  --version 1.13 \
  --nodegroup-name standard-workers \
  --node-type t3.micro \
  --nodes 3 \
  --nodes-min 1 \
  --nodes-max 4 \
  --node-ami auto
```

* Create or Update kubeconfig

```
aws eks --region ap-northeast-1 update-kubeconfig --name eks-hello 
```

* Setting test

```
kubectl get svc
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.100.0.1   <none>        443/TCP   2m18s
```

* Create guestbook

```
# Redis Master
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-master-controller.json
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-master-service.json

# Redis Slave
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-slave-controller.json
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/redis-slave-service.json

# guestbook
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/guestbook-controller.json
kubectl apply -f https://raw.githubusercontent.com/kubernetes/examples/master/guestbook-go/guestbook-service.json

# get services
kubectl get services -o wide
```

* Access guestbook on eks

```
open http://a6bf73bc1d31611e9bc6c0ab73b0f0ff-1576525680.ap-northeast-1.elb.amazonaws.com:3000/
```

* Cleanup guestbook

```
kubectl delete rc/redis-master rc/redis-slave rc/guestbook svc/redis-master svc/redis-slave svc/guestbook
```
