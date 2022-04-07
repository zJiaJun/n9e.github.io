---
title: "在K8s中运行grafana-agent收集metrics"
weight: 3
---
在本文档中，介绍如何以Deployment或者Daemonset的方式部署grafana-agent到您的k8s集群中，抓取宿主机上`kubelet`和`cAdvisor`的metrics指标，并把抓取到的数据，以remote_write的方式推送到Nightingale.

**通过本文档，我们预期达成以下目标：**
1. 部署grafana-agent到您的K8s集群中；
2. 配置grafana-agent抓取kubelet和cAdvisor的metrics；


K8s是开源的容器编排系统，自动化管理容器的部署、扩缩容等工作。K8s默认会暴露Node和控制面的若干metrics接口，这些接口兼容Prometheus的metrics规范。我们可以部署grafana-agent来收集Node的cAdvisor和kubelet metrics，并以remote_write的方式发送到Nightingale.


## 前置依赖
1. 一个开启RBAC（role-based access control）的Kubernetes集群；
1. 安装并配置好了kubectl命令行工具；

## 步骤一：创建 `ServiceAcount`、`ClusterRole`、`ClusterRoleBinding`
```bash
export NAMESPACE=default
MANIFEST_URL=https://raw.githubusercontent.com/flashcatcloud/fc-agent/fc-release/etc/k8s/agent-bare.yaml

curl -fsSL $MANIFEST_URL | envsubst |  kubectl apply -f -
```
## 步骤二：创建`ConfigMap`，配置grafana-agent
```bash
export NAMESPACE=default
export CLUSTER_NAME=kubernetes
export FC_REMOTE_WRITE_URL=http://10.206.0.16:8480/insert/0/prometheus/api/v1/write
#export FC_REMOTE_WRITE_URL=https://n9e-server:19000/prometheus/v1/write
#export FC_REMOTE_WRITE_USERNAME=fc_laiwei
#export FC_REMOTE_WRITE_PASSWORD=fc_laiweisecret

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
      - name: integrations
        remote_write:
        - url: ${FC_REMOTE_WRITE_URL}
          basic_auth:
            username: ${FC_REMOTE_WRITE_USERNAME}
            password: ${FC_REMOTE_WRITE_PASSWORD}
        scrape_configs:
        - job_name: integrations/kubernetes/cadvisor
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
            - role: node
          metric_relabel_configs:
            - action: drop
              regex: container_([a-z_]+);
              source_labels:
                - __name__
                - image
            - action: drop
              regex: container_(network_tcp_usage_total|network_udp_usage_total|tasks_state|cpu_load_average_10s)
              source_labels:
                - __name__
          relabel_configs:
            - replacement: kubernetes.default.svc:443
              target_label: __address__
            - regex: (.+)
              replacement: /api/v1/nodes/\$1/proxy/metrics/cadvisor
              source_labels:
                - __meta_kubernetes_node_name
              target_label: __metrics_path__
          scheme: https
          tls_config:
              ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
              insecure_skip_verify: false
              server_name: kubernetes
        - job_name: integrations/kubernetes/kubelet
          bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
          kubernetes_sd_configs:
            - role: node
          relabel_configs:
            - replacement: kubernetes.default.svc:443
              target_label: __address__
            - regex: (.+)
              replacement: /api/v1/nodes/\$1/proxy/metrics
              source_labels:
                - __meta_kubernetes_node_name
              target_label: __metrics_path__
          scheme: https
          tls_config:
              ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
              insecure_skip_verify: false
              server_name: kubernetes
EOF

envsubst | kubectl apply -n $NAMESPACE -f -
```

## 步骤三：在K8s中创建grafana-agent实例

> Daemonset

### 对于采集 node_exporter/ kubelet/ cAdvisor等指标，每个节点上只运行一个grafana-agent实例的情况，推荐以daemonset运行
```bash
export NAMESPACE=default
MANIFEST_URL=https://raw.githubusercontent.com/flashcatcloud/fc-agent/fc-release/etc/k8s/agent-daemonset.yaml

curl -fsSL $MANIFEST_URL | envsubst |  kubectl apply -f -
```

> Deployment

### 对于采集MySQLd_Exporter等需要运行多个grafana-agent实例的情况，推荐以deployment运行。
```bash
export NAMESPACE=default
MANIFEST_URL=https://raw.githubusercontent.com/flashcatcloud/fc-agent/fc-release/etc/k8s/agent-deployment.yaml

curl -fsSL $MANIFEST_URL | envsubst |  kubectl apply -f -
```


## 如何重建grafana-agent

> Daemonset

```bash
kubectl rollout restart daemonset/grafana-agent
```

> Deployment
```bash
kubectl rollout restart deployment/grafana-agent
```


至此，我们已经完成了在K8s中部署grafana-agent并收集metrics，进一步，我们还可以配置grafana-agent来[建立起完整的kubernetes指标监控体系](/fc-monitor/kube-o11y)。
