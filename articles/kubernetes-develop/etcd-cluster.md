## etcd cluster

```
service 三种模式 cluster/ExternalName/Headless Service[clusterIP: None]

3个节点的etcd集群
资源对象：service/pdb/statefulset

statefulset.yaml
创建集群脚步 --listen-peer-urls --listen-client-urls 不支持域名绑定形式，使用download api获取环境变量

提前定义CRD资源对象
apiVersion: etcd....
kind: EtcdCluster
spec:
  size: 3
  image: etcd-cluster:v1

业务逻辑 
使用kubebuilder构建脚手架，然后定义资源API
根据EtcdCluster构造Statefulset/Service/pdb，其实在vm操作的创建步骤使用k8s的资源对象整合起来
在控制器的Reconcile函数中进行逻辑处理，
	获取Etcdluster实例
	调谐获取当前状态和期望的状态进行对比
	需要使用到的Statefulset/Service/pdb 都需要单独的去实现一次调谐，然后去创建或在更新对应的资源对象
	除了调谐还需要对这些资源对象进行Watch，添加Statefulet和service的RBAC声明，k8s才会一直管理Etcdcluster，尽管不小心删除了也会自动创建回来
```