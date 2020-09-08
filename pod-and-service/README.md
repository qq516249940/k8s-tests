Service是kubernetes最核心的概念，通过创建Service，可以为一组具有相同功能的容器应用提供一个统一的入口地址，并且将请求进行负载分发到后端的各个容器应用上。

Service服务是一个虚拟概念，逻辑上代理后端pod。众所周知，pod生命周期短，状态不稳定，pod异常后新生成的pod ip会发生变化，之前pod的访问方式均不可达。通过service对pod做代理，service有固定的ip和port，ip:port组合自动关联后端pod，即使pod发生改变，kubernetes内部更新这组关联关系，使得service能够匹配到新的pod。这样，通过service提供的固定ip，用户再也不用关心需要访问哪个pod，以及pod是否发生改变，大大提高了服务质量。如果pod使用rc创建了多个副本，那么service就能代理多个相同的pod，通过kube-proxy，实现负载均衡。

集群中每个Node节点都有一个组件kube-proxy，实际上是为service服务的，通过kube-proxy，实现流量从service到pod的转发，kube-proxy也可以实现简单的负载均衡功能。

kube-proxy代理模式：userspace方式。kube-proxy在节点上为每一个服务创建一个临时端口，service的IP:port过来的流量转发到这个临时端口上，kube-proxy会用内部的负载均衡机制（轮询），选择一个后端pod，然后建立iptables，把流量导入这个pod里面。


yaml格式的Service定义文件的完整内容：

```
apiVersion: v1
kind: Service
matadata:
  name: string
  namespace: string
  labels:
    - name: string
  annotations:
    - name: string
spec:
  selector: []
  type: string
  clusterIP: string
  sessionAffinity: string
  ports:
  - name: string
    protocol: string
    port: int
    targetPort: int
    nodePort: int
  status:
    loadBalancer:
      ingress:
        ip: string
        hostname: string
```
# 负载分发策略
目前kubernetes提供了两种负载分发策略：RoundRobin和SessionAffinity

RoundRobin：轮询模式，即轮询将请求转发到后端的各个Pod上

SessionAffinity：基于客户端IP地址进行会话保持的模式，第一次客户端访问后端某个Pod，之后的请求都转发到这个Pod上

默认是RoundRobin模式

在某些场景中，开发人员希望自己控制负载均衡的策略，不使用Service提供的默认负载，kubernetes通过Headless Service的概念来实现。不给Service设置ClusterIP（无入口IP地址）：
