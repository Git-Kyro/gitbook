## K8S Command

命令行强制删除namespace下所有pod

```shell script
kubectl get namespace "rook-ceph" -o json | tr -d "\n" | sed "s/\"finalizers\": \[[^]]\+\]/\"finalizers\": []/" | kubectl replace --raw /api/v1/namespaces/rook-ceph/finalize -f -
kubectl delete pod [pod_name] -n [namespace] --force --grace-period=0
```

调用强制删除namespace下所有pod

```shell script
NAMESPACE=rook-ceph
kubectl proxy &
kubectl get namespace $NAMESPACE -o json |jq '.spec = {"finalizers":[]}' >temp.json
curl -k -H "Content-Type: application/json" -X PUT --data-binary @temp.json 127.0.0.1:8001/api/v1/namespaces/$NAMESPACE/finalize
```

强制删除NAMESPACE

```shell script
kubectl delete namespace NAMESPACENAME --force --grace-period=0
```

强制删除POD

```shell script
kubectl delete pod PODNAME --force --grace-period=0
```

删除default namespace下的pod名为pod-to-be-deleted-0

```shell script
ETCDCTL_API=3 etcdctl del /registry/pods/default/pod-to-be-deleted-0
```

删除需要删除的NAMESPACE

```shell script
etcdctl del /registry/namespaces/NAMESPACENAME
```

删除pv

```shell script
kubectl patch pv pvc-9cb42231-1cd0-452a-86c0-ef4e45ca4eb2 -p '{"metadata":{"finalizers":null}}'
```

导入本地镜像

```shell script
ctr -n=k8s.io image import app.tar
```
