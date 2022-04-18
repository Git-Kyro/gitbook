## K8S Command

命令行强制删除namespace下所有pod

```sh
kubectl get namespace "rook-ceph" -o json | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/" | kubectl replace --raw /api/v1/namespaces/rook-ceph/finalize -f -
kubectl delete pod [pod_name] -n [namespace] --force --grace-period=0
```

调用强制删除namespace下所有pod

```sh
NAMESPACE=rook-ceph
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```

强制删除NAMESPACE

```sh
kubectl delete namespace NAMESPACENAME --force --grace-period=0
```

强制删除POD

```sh
kubectl delete pod PODNAME --force --grace-period=0
```

删除default namespace下的pod名为pod-to-be-deleted-0

```sh
ETCDCTL_API=3 etcdctl del /registry/pods/default/pod-to-be-deleted-0
```

删除需要删除的NAMESPACE

```sh
etcdctl del /registry/namespaces/NAMESPACENAME
```

删除pv

```sh
kubectl patch pv pvc-9cb42231-1cd0-452a-86c0-ef4e45ca4eb2 -p '{"metadata":{"finalizers":null}}'
```

导入本地镜像

```sh
ctr -n=k8s.io image import app.tar
```
