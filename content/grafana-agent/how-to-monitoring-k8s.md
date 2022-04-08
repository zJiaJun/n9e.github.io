---
title: "监控K8s集群和应用"
---

>Acknowledgement: grafana-agent is powered by [Grafana Agent](https://github.com/grafana/agent). Grafana Agent is a lightweight telemetry collector based on Prometheus that only performs its scraping and remote_write functions. Agent can also collect metrics, logs, and traces for storage in Grafana Cloud and Grafana Enterprise, as well as OSS deployments of Loki (logs), and Tempo (traces), Prometheus (metrics), and Cortex (metrics). Grafana Agent also contains several integrations (embedded metrics exporters) like node-exporter, a MySQL exporter, and many more.
>
>The Grafana Agent uses the same code as Prometheus, but tackles these issues by only using the most relevant parts of Prometheus for interaction with hosted metrics:
>    - Service Discovery
>    - Scraping
>    - Write Ahead Log (WAL)
>    - Remote Write


**对于Kubernetes集群及其上应用，我们推荐从以下几个方面，建立起完整的kubernetes指标监控体系：**

# 前置依赖
1. 如何在K8s中运行和启动grafana-agent，请参考[在kubernetes中运行grafana-agent收集](/grafana-agent/k8s_metrics)。
2. 推荐您以daemonset，在每个节点上启动一个grafana-agent实例。

# 通过kubelet来了解和监控k8s节点的基本运行状态数据
## 方案一：直接访问kubelet来获取节点状态指标数据

Kubelet组件运行在Kubernetes集群的各个节点中，其负责维护和管理节点上Pod的运行状态。kubelet组件的正常运行直接关系到该节点是否能够正常的被Kubernetes集群正常使用。

基于[Prometheus在K8s环境下的服务发现能力](https://prometheus.io/docs/prometheus/latest/configuration/configuration/#kubernetes_sd_config)，在Node模式，grafana-agent会自动发现Kubernetes中所有Node节点的信息并作为监控的目标Target。 而这些Target的访问地址实际上就是Kubelet的访问地址。

> **创建ConfigMap，其中包含grafana-agent的配置文件如下**
```yaml
export NAMESPACE=default
export CLUSTER_NAME=kubernetes
export REMOTE_WRITE_URL=http://n9e-server:19000/prometheus/v1/write
export REMOTE_WRITE_USERNAME=fc_laiwei
export REMOTE_WRITE_PASSWORD=fc_laiweisecret

cat <<EOF |
kind: ConfigMap
metadata:
  name: grafana-agent
apiVersion: v1
data:
  agent.yaml: |
    server:
      http_listen_port: 12345
    metrics:
      wal_directory: /tmp/grafana-agent-wal
      global:
        scrape_interval: 15s
        scrape_timeout: 10s
        external_labels:
          cluster: ${CLUSTER_NAME}
      configs:
      - name: fc_k8s_scrape
        remote_write:
        - url: ${REMOTE_WRITE_URL}
          basic_auth:
            username: ${REMOTE_WRITE_USERNAME}
            password: ${REMOTE_WRITE_PASSWORD}
        scrape_configs:
        - job_name: integrations/kubernetes/kubelet
          scheme: https
          tls_config:
              ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
              insecure_skip_verify: true
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
          - role: node
          relabel_configs:
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
EOF

envsubst | kubectl apply -n $NAMESPACE -f -
```

> **重建grafana-agent实例**
```bash
kubectl rollout restart daemonset/grafana-agent
```
这里使用Node模式自动发现集群中所有Kubelet作为监控的数据采集目标，同时通过`labelmap`步骤，将Node节点上的标签，作为样本的标签保存到时间序列当中。
重新加载grafana-agent的配置文件，并重建grafana-agent的Pod实例后，在nightingale dashboard中搜索`{job="integrations/kubernetes/kubelet"}`，即可看到相应的时序数据了。

## 方案二：通过kube-apiserver提供的API间接获取kubelet的指标数据

不同于上面第一种方法，其直接通过kubelet的metrics服务采集监控数据，方法二通过Kubernetes的api-server提供的代理API访问各个节点中kubelet的metrics服务。

> **创建ConfigMap，其中包含grafana-agent的配置文件如下**
```yaml
export NAMESPACE=default
export CLUSTER_NAME=kubernetes
export REMOTE_WRITE_URL=http://10.206.0.16:8480/insert/0/prometheus/api/v1/write
export REMOTE_WRITE_URL=http://n9e-server:19000/prometheus/v1/write
export REMOTE_WRITE_USERNAME=fc_laiwei
export REMOTE_WRITE_PASSWORD=fc_laiweisecret

cat <<EOF |
kind: ConfigMap
metadata:
  name: grafana-agent
apiVersion: v1
data:
  agent.yaml: |
    server:
      http_listen_port: 12345
    metrics:
      wal_directory: /tmp/grafana-agent-wal
      global:
        scrape_interval: 15s
        scrape_timeout: 10s
        external_labels:
          cluster: ${CLUSTER_NAME}
      configs:
      - name: fc_k8s_scrape
        remote_write:
        - url: ${REMOTE_WRITE_URL}
          basic_auth:
            username: ${REMOTE_WRITE_USERNAME}
            password: ${REMOTE_WRITE_PASSWORD}
        scrape_configs:
        - job_name: 'integrations/kubernetes/kubelet'
          scheme: https
          tls_config:
            ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
          - role: node
          relabel_configs:
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
          - target_label: __address__
            replacement: kubernetes.default.svc:443
          - source_labels: [__meta_kubernetes_node_name]
            regex: (.+)
            target_label: __metrics_path__
            replacement: /api/v1/nodes/\${1}/proxy/metrics
EOF

envsubst | kubectl apply -n $NAMESPACE -f -
```

通过relabeling，将从Kubernetes获取到的默认地址`__address__`替换为`kubernetes.default.svc:443`。同时将`__metrics_path__`替换为api-server的代理地址`/api/v1/nodes/${1}/proxy/metrics`。

通过获取各个节点中kubelet的监控指标，您可以评估集群中各节点的性能表现。例如:

**1. 通过指标`kubelet_pod_start_duration_seconds`可以获得当前节点中Pod启动时间相关的统计数据。**
```yaml
kubelet_pod_start_duration_seconds{quantile="0.99"}
```

**2. Pod平均启动时间（包含镜像下载时间）：**
```yaml
kubelet_pod_start_duration_seconds_sum / kubelet_pod_start_duration_seconds_count
```

除此以外，监控指标`kubelet_docker_*`还可以体现出kubelet与当前节点的docker服务的调用情况，从而可以反映出docker本身是否会影响kubelet的性能表现等问题。

# 通过cAdvisor来了解和监控节点中的容器运行状态

各节点的kubelet组件中除了包含自身的监控指标信息以外，kubelet组件还内置了对cAdvisor的支持。cAdvisor能够获取当前节点上运行的所有容器的资源使用情况，通过访问kubelet的/metrics/cadvisor地址可以获取到cadvisor的监控指标，因此和获取kubelet监控指标类似，这里同样通过node模式自动发现所有的kubelet信息，并通过适当的relabel过程，修改监控采集任务的配置。 与采集kubelet自身监控指标相似，这里也有两种方式采集cadvisor中的监控指标：

## 方案一：直接访问kubelet的/metrics/cadvisor地址，需要跳过ca证书认证
```yaml
export NAMESPACE=default
export CLUSTER_NAME=kubernetes
export REMOTE_WRITE_URL=http://10.206.0.16:8480/insert/0/prometheus/api/v1/write
export REMOTE_WRITE_URL=http://n9e-server:19000/prometheus/v1/write
export REMOTE_WRITE_USERNAME=fc_laiwei
export REMOTE_WRITE_PASSWORD=fc_laiweisecret

cat <<EOF |
kind: ConfigMap
metadata:
  name: grafana-agent
apiVersion: v1
data:
  agent.yaml: |
    server:
      http_listen_port: 12345
    metrics:
      wal_directory: /tmp/grafana-agent-wal
      global:
        scrape_interval: 15s
        scrape_timeout: 10s
        external_labels:
          cluster: ${CLUSTER_NAME}
      configs:
      - name: fc_k8s_scrape
        remote_write:
        - url: ${REMOTE_WRITE_URL}
          basic_auth:
            username: ${REMOTE_WRITE_USERNAME}
            password: ${REMOTE_WRITE_PASSWORD}
        scrape_configs:
        - job_name: 'integrations/kubernetes/cadvisor'
          scheme: https
          tls_config:
            ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
            insecure_skip_verify: true
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
          - role: node
          relabel_configs:
          - source_labels: [__meta_kubernetes_node_name]
            regex: (.+)
            target_label: __metrics_path__
            replacement: metrics/cadvisor
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
EOF

envsubst | kubectl apply -n $NAMESPACE -f -
```

## 方案二：通过api-server提供的代理地址访问kubelet的/metrics/cadvisor地址
```yaml
export NAMESPACE=default
export CLUSTER_NAME=kubernetes
export REMOTE_WRITE_URL=http://10.206.0.16:8480/insert/0/prometheus/api/v1/write
export REMOTE_WRITE_URL=http://n9e-server:19000/prometheus/v1/write
export REMOTE_WRITE_USERNAME=fc_laiwei
export REMOTE_WRITE_PASSWORD=fc_laiweisecret

cat <<EOF |
kind: ConfigMap
metadata:
  name: grafana-agent
apiVersion: v1
data:
  agent.yaml: |
    server:
      http_listen_port: 12345
    metrics:
      wal_directory: /tmp/grafana-agent-wal
      global:
        scrape_interval: 15s
        scrape_timeout: 10s
        external_labels:
          cluster: ${CLUSTER_NAME}
      configs:
      - name: fc_k8s_scrape
        remote_write:
        - url: ${REMOTE_WRITE_URL}
          basic_auth:
            username: ${REMOTE_WRITE_USERNAME}
            password: ${REMOTE_WRITE_PASSWORD}
        scrape_configs:
        - job_name: 'integrations/kubernetes/cadvisor'
          scheme: https
          tls_config:
            ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
          - role: node
          relabel_configs:
          - target_label: __address__
            replacement: kubernetes.default.svc:443
          - source_labels: [__meta_kubernetes_node_name]
            regex: (.+)
            target_label: __metrics_path__
            replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor
          - action: labelmap
            regex: __meta_kubernetes_node_label_(.+)
```

# 使用NodeExporter监控节点资源使用情况

为了能够采集集群中各个节点的资源使用情况，我们可以借助grafana-agent内置的NodeExporter。具体的步骤可以参考：[grafana-agent node_exporter](/grafana-agent/integrations/node-exporter-config)。


# 通过kube-apiserver来了解整个K8s集群的详细运行状态

`kube-apiserver`扮演了整个Kubernetes集群管理的入口的角色，负责对外暴露Kubernetes API。kube-apiserver组件一般是独立部署在集群外的，为了能够让部署在集群内的应用（kubernetes插件或者用户应用）能够与kube-apiserver交互，Kubernetes会默认在命名空间下创建一个名为kubernetes的服务，如下所示：

```bash
$ kubectl get svc kubernetes -o wide
NAME                  TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE       SELECTOR
kubernetes            ClusterIP   10.96.0.1       <none>        443/TCP          166d      <none>
```

而该kubernetes服务代理的后端实际地址通过endpoints进行维护，如下所示：

```bash
$ kubectl get endpoints kubernetes
NAME         ENDPOINTS        AGE
kubernetes   10.0.2.15:8443   166d
```

通过这种方式集群内的应用或者系统主机就可以通过集群内部的DNS域名kubernetes.default.svc访问到部署外部的kube-apiserver实例。

因此，如果我们想要监控kube-apiserver相关的指标，只需要通过endpoints资源找到kubernetes对应的所有后端地址即可。

如下所示，创建监控任务kubernetes-apiservers，这里指定了服务发现模式为endpoints。grafana-agent会查找当前集群中所有的endpoints配置，并通过relabel进行判断是否为apiserver对应的访问地址：

```yaml
    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https
      - target_label: __address__
        replacement: kubernetes.default.svc:443
```

在relabel_configs配置中第一步用于判断当前endpoints是否为kube-apiserver对用的地址。第二步，替换监控采集地址到kubernetes.default.svc:443即可。重新加载配置文件，重建grafana-agent实例，用以下`promql` `{service="kubernetes", job="apiserver"}`即可在nightingale dashboard中得到kube-apiserver相关的metrics数据。

# 通过BlackboxExporter了解和监控K8s集群中的网络连通状况
为了能够对Ingress和Service进行探测，我们需要在K8s集群部署`Blackbox Exporter`实例。 如下所示，创建blackbox-exporter.yaml用于描述部署相关的内容:

```yaml
cat << EOF |
apiVersion: v1
kind: Service
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
spec:
  ports:
  - name: blackbox
    port: 9115
    protocol: TCP
  selector:
    app: blackbox-exporter
  type: ClusterIP
---
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  labels:
    app: blackbox-exporter
  name: blackbox-exporter
spec:
  replicas: 1
  selector:
    matchLabels:
      app: blackbox-exporter
  template:
    metadata:
      labels:
        app: blackbox-exporter
    spec:
      containers:
      - image: prom/blackbox-exporter
        imagePullPolicy: IfNotPresent
        name: blackbox-exporter
EOF
kubectl apply -f -
```

通过以上命令，将在K8s集群中部署了一个`Blackbox Exporter`的Pod实例，同时通过服务blackbox-exporter在集群内暴露访问地址blackbox-exporter.default.svc.cluster.local，对于集群内的任意服务都可以通过该内部DNS域名访问Blackbox Exporter实例：

```bash
$ kubectl get pods
NAME                                        READY     STATUS        RESTARTS   AGE
blackbox-exporter-f77fc78b6-72bl5           1/1       Running       0          4s

$ kubectl get svc
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
blackbox-exporter           ClusterIP   10.109.144.192   <none>        9115/TCP         3m
```

为了能够让grafana-agent能够自动的对Service进行探测，我们需要通过服务发现自动找到所有的Service信息。 如下所示，在grafana-agent的配置文件中添加名为kubernetes-services的监控采集任务：

```yaml
    - job_name: 'kubernetes-services'
      metrics_path: /probe
      params:
        module: [http_2xx]
      kubernetes_sd_configs:
      - role: service
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.default.svc.cluster.local:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name
```

在该任务配置中，通过指定kubernetes_sd_config的role为service指定服务发现模式：

```yaml
  kubernetes_sd_configs:
    - role: service
```

为了区分集群中需要进行探测的Service实例，我们通过标签‘prometheus.io/probe: true’进行判断，从而过滤出需要探测的所有Service实例：

```yaml
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
```

并且将通过服务发现获取到的Service实例地址```__address__```转换为获取监控数据的请求参数。同时将```__address```执行Blackbox Exporter实例的访问地址，并且重写了标签instance的内容：

```yaml
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.default.svc.cluster.local:9115
      - source_labels: [__param_target]
        target_label: instance
```

最后，为监控样本添加了额外的标签信息：

```yaml
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name
```

对于Ingress而言，也是一个相对类似的过程，这里给出对Ingress探测的grafana-agent任务配置作为参考：
```yaml
    - job_name: 'kubernetes-ingresses'
      metrics_path: /probe
      params:
        module: [http_2xx]
      kubernetes_sd_configs:
      - role: ingress
      relabel_configs:
      - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.default.svc.cluster.local:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: kubernetes_name
```

# 通过kube-state-metrics了解和监控K8s集群自身和应用的运行状态

kube-state-metrics重点回答以下方面的问题：
* 我调度了多少个replicas？现在可用的有几个？
* 多少个Pod是running/stopped/terminated状态？
* Pod重启了多少次？
* 我有多少job在运行中？

kube-state-metrics基于client-go开发，轮询Kubernetes API，并将Kubernetes的结构化信息转换为metrics。他所支持的指标包括：
* CronJob Metrics
* DaemonSet Metrics
* Deployment Metrics
* Job Metrics
* LimitRange Metrics
* Node Metrics
* PersistentVolume Metrics
* PersistentVolumeClaim Metrics
* Pod Metrics
* Pod Disruption Budget Metrics
* ReplicaSet Metrics
* ReplicationController Metrics
* ResourceQuota Metrics
* Service Metrics
* StatefulSet Metrics
* Namespace Metrics
* Horizontal Pod Autoscaler Metrics
* Endpoint Metrics
* Secret Metrics
* ConfigMap Metrics

以Pod为例：
* kube_pod_info
* kube_pod_owner
* kube_pod_status_phase
* kube_pod_status_ready
* kube_pod_status_scheduled
* kube_pod_container_status_waiting
* kube_pod_container_status_terminated_reason
* ...

[部署清单](https://github.com/kubernetes/kube-state-metrics/tree/master/examples/standard)：
```yaml
    ├── cluster-role-binding.yaml
    ├── cluster-role.yaml
    ├── deployment.yaml
    ├── service-account.yaml
    ├── service.yaml
```

主要镜像有：
- image: quay.io/coreos/kube-state-metrics:v2.4.2

对于pod的资源限制，一般情况下：
```
200MiB memory
0.1 cores
```

超过100节点的集群：
```
2MiB memory per node
0.001 cores per node
```

因为kube-state-metrics-service.yaml中有`prometheus.io/scrape: 'true'`标识，因此会将metric暴露给grafana-agent，而grafana-agent会在kubernetes-service-endpoints这个job下自动发现kube-state-metrics，并开始拉取metrics，无需其他配置。

**使用kube-state-metrics后的常用场景有**：
* 存在执行失败的Job: kube_job_status_failed{job="kubernetes-service-endpoints",k8s_app="kube-state-metrics"}==1
* 集群节点状态错误: kube_node_status_condition{condition="Ready",status!="true"}==1
* 集群中存在启动失败的Pod：kube_pod_status_phase{phase=~"Failed|Unknown"}==1
* 最近30分钟内有Pod容器重启: changes(kube_pod_container_status_restarts[30m])>0

> 参考资料

- [Prometheus与服务发现](https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/sd/why-need-service-discovery)
- [基于文件的服务发现](https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/sd/service-discovery-with-file)
- [基于Consul的服务发现](https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/sd/service-discovery-with-consul)
- [服务发现与Relabel](https://yunlzheng.gitbook.io/prometheus-book/part-ii-prometheus-jin-jie/sd/service-discovery-with-relabel)
- [Kubernetes下的服务发现](https://yunlzheng.gitbook.io/prometheus-book/part-iii-prometheus-shi-zhan/readmd/service-discovery-with-kubernetes)
- [监控Kubernetes集群](https://yunlzheng.gitbook.io/prometheus-book/part-iii-prometheus-shi-zhan/readmd/use-prometheus-monitor-kubernetes)
- [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
- [kube-state-metrics deoplyment](https://yasongxu.gitbook.io/container-monitor/yi-.-kai-yuan-fang-an/di-1-zhang-cai-ji/kube-state-metrics)


> *Acknowledgement:本文档在[yunlzheng 监控Kubernetes集群](https://yunlzheng.gitbook.io/prometheus-book/part-iii-prometheus-shi-zhan/readmd/use-prometheus-monitor-kubernetes)的基础上修改和补充而成，相关文字的版权归属[原作者yunlzheng](https://github.com/yunlzheng)所有，并致以谢意。*
