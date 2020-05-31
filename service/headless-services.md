有时不需要或不想要负载均衡，以及单独的 Service IP。 遇到这种情况，可以通过指定 Cluster IP（spec.clusterIP）的值为 "None" 来创建 Headless Service。

您可以使用 headless Service 与其他服务发现机制进行接口，而不必与 Kubernetes 的实现捆绑在一起。

对这 headless Service 并不会分配 Cluster IP，kube-proxy 不会处理它们，而且平台也不会为它们进行负载均衡和路由。 DNS 如何实现自动配置，依赖于 Service 是否定义了 selector。

**配置 Selector**
对定义了 selector 的 Headless Service，Endpoint 控制器在 API 中创建了 Endpoints 记录，并且修改 DNS 配置返回 A 记录（地址），通过这个地址直接到达 Service 的后端 Pod 上。

**不配置 Selector**
对没有定义 selector 的 Headless Service，Endpoint 控制器不会创建 Endpoints 记录。 然而 DNS 系统会查找和配置，无论是：

ExternalName 类型 Service 的 CNAME 记录
记录：与 Service 共享一个名称的任何 Endpoints，以及所有其它类型
