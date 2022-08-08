## Service与Endpoint关联关系
正常情况下，创建一个`service`之后，默认会同步生成一个`endpoint`资源，但是在有一次排查故障的时候，
发现了某一个`service`对象没有对应`endpoint`资源的情况。

问题：`service`在什么情况下会创建对应的`endpoint`资源。

## 场景复现
场景一：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

```shell
## service
$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
my-service    ClusterIP   10.96.105.215   <none>        80/TCP     14s

## endpoint
$ kubectl get endpoint
NAME          ENDPOINTS                                           AGE
my-service    <none>                                              14s
```
可以发现，这种场景下，会创建出对应`endpoint`资源。

通过查看官方文档，发现如果`service`没有定义选择器的话，就不会创建对应的`endpoint`。
> Because this Service has no selector, the corresponding Endpoints object is not created automatically. You can manually map the Service to the network address and port where it's running, by adding an Endpoints object manually:

场景二：
```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

```shell
## service
$ kubectl get svc
NAME          TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
my-service    ClusterIP   10.96.105.215   <none>        80/TCP     14s

## endpoint
$ kubectl get endpoint
NAME          ENDPOINTS                                           AGE
```
可以看到确实没有自动生成`endpoint`资源。

## 源码分析
自动创建`endpoint`资源的业务逻辑肯定是放在`kube-controller-manager`组件中，
当发现有`service`资源的变动时，需要确保自动创建的`endpoint`一直存在才行。

源码位置：`pkg/controller/endpoint/endpoints_controller.go`
```go
func (e *Controller) syncService(ctx context.Context, key string) error {
	startTime := time.Now()
	defer func() {
		klog.V(4).Infof("Finished syncing service %q endpoints. (%v)", key, time.Since(startTime))
	}() 
	
	// 如果 service 没有 selector 属性的话，直接跳过去，不做任何操作。
	if service.Spec.Selector == nil {
		// services without a selector receive no endpoints from this controller;
		// these services will receive the endpoints that are created out-of-band via the REST API.
		return nil
	}
	// 省略code
	
	// 开始更新或者创建对应的 endpoint 资源.
	klog.V(4).Infof("Update endpoints for %v/%v, ready: %d not ready: %d", service.Namespace, service.Name, totalReadyEps, totalNotReadyEps)
	if createEndpoints {
		// No previous endpoints, create them
		_, err = e.client.CoreV1().Endpoints(service.Namespace).Create(ctx, newEndpoints, metav1.CreateOptions{})
	} else {
		// Pre-existing
		_, err = e.client.CoreV1().Endpoints(service.Namespace).Update(ctx, newEndpoints, metav1.UpdateOptions{})
	}
	// 省略code
	return nil
}
```

## 参考文献
1. https://kubernetes.io/docs/concepts/services-networking/service/