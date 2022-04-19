## K8S clientset

```go
package main

import (
    "flag"
    "fmt"
    "os"
    "path/filepath"

    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/client-go/kubernetes"
    "k8s.io/client-go/rest"
    "k8s.io/client-go/tools/clientcmd"
)

func main() {
    var err error
    var config *rest.Config
    var kubeconfig *string

    if home := homeDir(); home != "" {
        kubeconfig = flag.String("kubeconfig", filepath.Join(home, ".kube", "config"), "(optional) absolute path to the kubeconfig file")
    } else {
        kubeconfig = flag.String("kubeconfig", "", "absolute path to the kubeconfig file")
    }
    flag.Parse()

    // 使用 ServiceAccount 创建集群配置（InCluster模式）
    if config, err = rest.InClusterConfig(); err != nil {
        // 使用 KubeConfig 文件创建集群配置
        if config, err = clientcmd.BuildConfigFromFlags("", *kubeconfig); err != nil {
            panic(err.Error())
        }
    }

    // 创建 clientset
    clientset, err := kubernetes.NewForConfig(config)
    if err != nil {
        panic(err.Error())
    }

    // 使用 clientsent 获取 Deployments
    deployments, err := clientset.AppsV1().Deployments("default").List(metav1.ListOptions{})
    if err != nil {
        panic(err)
    }
    for idx, deploy := range deployments.Items {
        fmt.Printf("%d -> %s\n", idx+1, deploy.Name)
    }

}

func homeDir() string {
    if h := os.Getenv("HOME"); h != "" {
        return h
    }
    return os.Getenv("USERPROFILE") // windows
}
```

通过 client-go 提供的 Clientset 对象获取资源数据

1. 使用 kubeconfig 文件或者 ServiceAccount（InCluster 模式）来创建访问 Kubernetes API 的 Restful 配置参数，也就是代码中的 rest.Config 对象

2. 使用 rest.Config 参数创建 Clientset 对象，这一步非常简单，直接调用kubernetes.NewForConfig(config)即可初始化

3. 然后是 Clientset 对象的方法去获取各个 Group 下面的对应资源对象进行 CRUD 操作


```go
// staging/src/k8s.io/client-go/kubernetes/clientset.go

import (appsv1 "k8s.io/client-go/kubernetes/typed/apps/v1")

// NewForConfig 使用给定的 config 创建一个新的 Clientset
func NewForConfig(c *rest.Config) (*Clientset, error) {
    configShallowCopy := *c
    var cs Clientset
    var err error
    // 将其他 Group 和版本的资源的 RESTClient 封装到全局的 Clientset 对象中
    cs.appsV1, err = appsv1.NewForConfig(&configShallowCopy)
    if err != nil {
        return nil, err
    }
    ......
    return &cs, nil
}
```

资源实现下面的接口就可以使用 Clientset 进行 CRUD 操作, 例如 Deployment

```go
clientset.AppsV1().Deployments("default").List(metav1.ListOptions{})
```

Deployment 在 appsv1.AppsV1Interface 这个接口上

```go
// staging/src/k8s.io/client-go/kubernetes/clientset.go

// AppsV1 retrieves the AppsV1Client
func (c *Clientset) AppsV1() appsv1.AppsV1Interface {
    return c.appsV1
}
```

appsv1.AppsV1Interface 包含的 rest.Interface 和 DeploymentsGetter 接口, DeploymentsGetter 实际上是对 DeploymentInterface 做了一层封装而已，

所以只要实现 rest.Interface 和 DeploymentInterface 就可以了

```go
// staging/src/k8s.io/client-go/kubernetes/typed/apps/v1/apps_client.go

type AppsV1Interface interface {
    RESTClient() rest.Interface
    DeploymentsGetter
    ...
}
```

rest.Interface 的具体实现

```go
type Interface interface {
    GetRateLimiter() flowcontrol.RateLimiter
    Verb(verb string) *Request
    Post() *Request
    Put() *Request
    Patch(pt types.PatchType) *Request
    Get() *Request
    Delete() *Request
    APIVersion() schema.GroupVersion
}
```

DeploymentsGetter 实际是 DeploymentInterface

```go
// staging/src/k8s.io/client-go/kubernetes/typed/apps/v1/deployment.go

type DeploymentsGetter interface {
    Deployments(namespace string) DeploymentInterface
}


type DeploymentInterface interface {
    Create(ctx context.Context, deployment *v1.Deployment, opts metav1.CreateOptions) (*v1.Deployment, error)
    Update(ctx context.Context, deployment *v1.Deployment, opts metav1.UpdateOptions) (*v1.Deployment, error)
    UpdateStatus(ctx context.Context, deployment *v1.Deployment, opts metav1.UpdateOptions) (*v1.Deployment, error)
    Delete(ctx context.Context, name string, opts metav1.DeleteOptions) error
    DeleteCollection(ctx context.Context, opts metav1.DeleteOptions, listOpts metav1.ListOptions) error
    Get(ctx context.Context, name string, opts metav1.GetOptions) (*v1.Deployment, error)
    List(ctx context.Context, opts metav1.ListOptions) (*v1.DeploymentList, error)
    Watch(ctx context.Context, opts metav1.ListOptions) (watch.Interface, error)
    Patch(ctx context.Context, name string, pt types.PatchType, data []byte, opts metav1.PatchOptions, subresources ...string) (result *v1.Deployment, err error)
    GetScale(ctx context.Context, deploymentName string, options metav1.GetOptions) (*autoscalingv1.Scale, error)
    UpdateScale(ctx context.Context, deploymentName string, scale *autoscalingv1.Scale, opts metav1.UpdateOptions) (*autoscalingv1.Scale, error)

    DeploymentExpansion
}
```

AppsV1Client 实现上面接口全过程，rest.Interface 和 DeploymentInterface

```go
// staging/src/k8s.io/client-go/kubernetes/clientset.go

import (appsv1 "k8s.io/client-go/kubernetes/typed/apps/v1")

// NewForConfig 使用给定的 config 创建一个新的 Clientset
func NewForConfig(c *rest.Config) (*Clientset, error) {
    configShallowCopy := *c
    var cs Clientset
    var err error
    // 将其他 Group 和版本的资源的 RESTClient 封装到全局的 Clientset 对象中
    cs.appsV1, err = appsv1.NewForConfig(&configShallowCopy)
    if err != nil {
        return nil, err
    }
    ......
    return &cs, nil
}
```

将资源的 RESTClient 封装到了全局的 Clientset 中，也是 rest.Interface 的具体实现


```go
// staging/src/k8s.io/client-go/kubernetes/typed/apps/v1/apps_client.go

// NewForConfig 根据 rest.Config 创建一个 AppsV1Client
func NewForConfig(c *rest.Config) (*AppsV1Client, error) {
    config := *c
    // 为 rest.Config 设置资源对象默认的参数
    if err := setConfigDefaults(&config); err != nil {
        return nil, err
    }
  // 实例化 AppsV1Client 的 RestClient
    client, err := rest.RESTClientFor(&config)
    if err != nil {
        return nil, err
    }
    return &AppsV1Client{client}, nil
}

// 追踪 rest.RESTClientFor 实现 rest.Interface 过程

1. rest.RESTClientFor

2. func RESTClientFor(config *Config) (*RESTClient, error) {
    ...
    return NewRESTClient(baseURL, versionedAPIPath, clientContent, rateLimiter, httpClient)

3. func NewRESTClient(baseURL *url.URL, versionedAPIPath string, config ClientContentConfig, rateLimiter flowcontrol.RateLimiter, client *http.Client) (*RESTClient, error) {
   ...
   return &RESTClient{...}

4. func (c *RESTClient) Get(verb string) *Request 
   func (c *RESTClient) Post() *Request
```

DeploymentInterface 的具体实现

```go
// staging/src/k8s.io/client-go/kubernetes/typed/apps/v1/apps_client.go

type AppsV1Client struct {
    restClient rest.Interface
}

func (c *AppsV1Client) Deployments(namespace string) DeploymentInterface {
    return newDeployments(c, namespace)
}
```

```go
// staging/src/k8s.io/client-go/kubernetes/typed/apps/v1/deployment.go
// deployments 实现了 DeploymentInterface 接口
type deployments struct {
    client rest.Interface
    ns     string
}

// newDeployments 实例化 deployments 对象
func newDeployments(c *AppsV1Client, namespace string) *deployments {
    return &deployments{
        client: c.RESTClient(),
        ns:     namespace,
    }
}

// deployments 实现了 DeploymentInterface 接口
func (c *deployments) Get(ctx context.Context, name string, options metav1.GetOptions) (result *v1.Deployment, err error) {
func (c *deployments) List(ctx context.Context, opts metav1.ListOptions) (result *v1.DeploymentList, err error) {
func (c *deployments) Watch(ctx context.Context, opts metav1.ListOptions) (watch.Interface, error) {
```

