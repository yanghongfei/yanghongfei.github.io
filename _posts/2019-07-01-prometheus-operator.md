---
layout: post
title: Prometheus-Operator参考
categories: [K8S, Prometheus]
description: K8S Prometheus-Operator部署及自定义监控
keywords: K8S, Prometheus
---


### Prometheus-Operator

- [Prometheus Operator介绍](https://github.com/yanghongfei/Kubernetes#153-%E5%AE%89%E8%A3%85promethues-operator)
- 官网：https://github.com/coreos/prometheus-operator

- 参考文档：http://www.servicemesher.com/blog/prometheus-operator-manual/

####  快速开始

> 官方把所有文件都放在一起，这里我分类下

```
cd contrib/kube-prometheus/manifests/
mkdir -p operator node-exporter alertmanager grafana kube-state-metrics prometheus serviceMonitor adapter
mv *-serviceMonitor* serviceMonitor/
mv 0prometheus-operator* operator/
mv grafana-* grafana/
mv kube-state-metrics-* kube-state-metrics/
mv alertmanager-* alertmanager/
mv node-exporter-* node-exporter/
mv prometheus-adapter* adapter/
mv prometheus-* prometheus/
$ ll
total 40
drwxr-xr-x 9 root root 4096 Jan  6 14:19 ./
drwxr-xr-x 9 root root 4096 Jan  6 14:15 ../
-rw-r--r-- 1 root root   60 Jan  6 14:15 00namespace-namespace.yaml
drwxr-xr-x 3 root root 4096 Jan  6 14:19 adapter/
drwxr-xr-x 3 root root 4096 Jan  6 14:19 alertmanager/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 grafana/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 kube-state-metrics/
drwxr-xr-x 2 root root 4096 Jan  6 14:18 node-exporter/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 operator/
drwxr-xr-x 2 root root 4096 Jan  6 14:19 prometheus/
drwxr-xr-x 2 root root 4096 Jan  6 14:17 serviceMonitor/
```

- 部署opertor

> 先创建ns和operator，quay.io仓库拉取慢，可以使用脚本拉取，其他镜像也可以这样去拉，不过在apply之前才能拉，一旦被docker接手拉取就只能漫长等

```shell
kubectl apply -f .
curl -s https://zhangguanzhang.github.io/bash/pull.sh | bash -s -- quay.io/coreos/prometheus-operator:v0.26.0
kubectl apply -f operator/
# 确认状态运行正常再往后执行，这里镜像是quay.io仓库的可能会很慢耐心等待或者自行修改成能拉取到的。
$ kubectl -n monitoring get pod
NAME                                   READY     STATUS    RESTARTS   AGE
prometheus-operator-56954c76b5-qm9ww   1/1       Running   0          24s
```

- 若脚本访问不到，以下是脚本内容

```shell
#!/bin/bash
[ -z "$set_e" ] && set -e

[ -z "$1" ] && { echo '$1 is not set';exit 2; }

if [ "$1" == search ];then
    shift
    which jq &> /dev/null || { echo 'search needs jq, please install the jq';exit 2; }
    img=${1%/}
    [[ $img == *:* ]] && img_name=${img/://} || img_name=$img
    if [[ "$img" =~ ^gcr.io|^k8s.gcr.io ]];then
        if [[ "$img" =~ ^k8s.gcr.io ]];then
            img_name=${img_name/k8s.gcr.io\//gcr.io/google_containers/}
        elif [[ "$img" == *google-containers* ]]; then
            img_name=${img_name/google-containers/google_containers}
        fi
        repository=gcr.io
    elif [[ "$img" =~ ^quay.io ]];then
            repository=quay.io 
    else 
        echo 'not sync the namespaces!';exit 0;
    fi
    #echo '查询用的github,GFW原因可能会比较久,请确保能访问到github'
    curl -sX GET https://api.github.com/repos/zhangguanzhang/${repository}/contents/$img_name?ref=develop |
        jq -r 'length as $len | if $len ==2 then .message elif $len ==12 then .name else .[].name  end'
else
    img=$1
    name=${img%:*}
    [[ $img == *:* ]] && tag=${img##*:} || tag=latest

    if [[ "$img" =~ ^gcr.io|^quay.io ]];then
        repo=zhangguanzhang/${name//\//.}
        [[ "$img" == *google-containers* ]] && repo=${repo/google-containers/google_containers}
        docker pull $repo:$tag
        docker tag $repo:$tag $img
        docker rmi $repo:$tag

    elif [[ "$img" =~ ^k8s.gcr.io ]];then
        repo=zhangguanzhang/${name/k8s.gcr.io\//gcr.io.google_containers.}
        docker pull $repo:$tag
        docker tag $repo:$tag $img
        docker rmi $repo:$tag
    else
        echo 'not sync the namespaces!';exit 0;
    fi
fi

```



- 接着部署整套CRD

```shell
# 由于网络原因，可能需要很长时间
kubectl apply -f adapter/
kubectl apply -f alertmanager/
kubectl apply -f node-exporter/
kubectl apply -f kube-state-metrics/
kubectl apply -f grafana/
kubectl apply -f prometheus/
kubectl apply -f serviceMonitor/

#查看进度
kubectl -n monitoring get all
```

#### 使用Traefik 代理出来Prometheus Grafana 

> 将以下文件进行apply即可
>
> kubectl apply -f  traefik-grafana.yaml 
>
> kubectl apply -f  traefik-prometheus.yaml 

- `traefik-grafana.yaml `

```yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: grafana
  namespace: monitoring
spec:
  rules:
  - host: k8s-grafana.domain.com
    http:
      paths:
      - path: /
        backend:
          serviceName: grafana
          servicePort: 3000
```

- ` traefik-prometheus.yaml `

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: prometheus-k8s
  namespace: monitoring
spec:
  rules:
  - host: k8s-prometheus.domain.com
    http:
      paths:
      - path: /
        backend:
          serviceName: prometheus-k8s
          servicePort: 9090     
```

- `alertmanager-ingress.yaml`

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: alertmanager-k8s
  namespace: monitoring
spec:
  rules:
  - host: k8s-alertmanager.domain.com
    http:
      paths:
      - path: /
        backend:
          serviceName: alertmanager-main
          servicePort: 9093
```



- 效果图

  ![](https://github.com/yanghongfei/Kubernetes/raw/master/images/dashboard.png)

![](https://github.com/yanghongfei/Kubernetes/raw/master/images/grafana-cluster.png)

![](https://github.com/yanghongfei/Kubernetes/blob/master/images/grafana-nodes.png)



![](https://github.com/yanghongfei/Kubernetes/blob/master/images/altermanager.png)



#### 配置Promethe-Operator自定义报警

> OK ，到了这一步监控组件已经安装完成，接下来就是要理解怎么自定义规则，规则怎么写？报警怎么触发。

现在登陆到Prometheus的Dashboard发现已经有一些自带的规则了，还有已经触发的Alter，如下图

![](https://github.com/yanghongfei/Kubernetes/raw/master/images/prometheus.png)



>但是这些报警信息是哪里来的呢？他们应该用怎样的方式通知我们呢？现在我们通过 Operator 部署的呢？我们可以在 Prometheus Dashboard 的 Config 页面下面**查看关于 AlertManager 的配置**：

```yaml
global:
  scrape_interval: 30s
  scrape_timeout: 10s
  evaluation_interval: 30s
  external_labels:
    prometheus: monitoring/k8s
    prometheus_replica: prometheus-k8s-0
alerting:
  alert_relabel_configs:
  - separator: ;
    regex: prometheus_replica
    replacement: $1
    action: labeldrop
  alertmanagers:
  - kubernetes_sd_configs:
    - role: endpoints
      namespaces:
        names:
        - monitoring
    scheme: http
    path_prefix: /
    timeout: 10s
    relabel_configs:
    - source_labels: [__meta_kubernetes_service_name]
      separator: ;
      regex: alertmanager-main
      replacement: $1
      action: keep
    - source_labels: [__meta_kubernetes_endpoint_port_name]
      separator: ;
      regex: web
      replacement: $1
      action: keep
rule_files:
- /etc/prometheus/rules/prometheus-k8s-rulefiles-0/*.yaml
```

上面 alertmanagers 实例的配置我们可以看到是通过角色为 endpoints 的 kubernetes 的服务发现机制获取的，匹配的是服务名为 alertmanager-main，端口名未 web 的 Service 服务，我们查看下 alertmanager-main 这个 Service：

```shell
$ kubectl describe svc alertmanager-main -n monitoring
Name:              alertmanager-main
Namespace:         monitoring
Labels:            alertmanager=main
Annotations:       kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"alertmanager":"main"},"name":"alertmanager-main","namespace":"...
Selector:          alertmanager=main,app=alertmanager
Type:              ClusterIP
IP:                10.111.82.41
Port:              web  9093/TCP
TargetPort:        web/TCP
Endpoints:         100.64.2.6:9093,100.64.2.8:9093,100.64.3.6:9093
Session Affinity:  None
Events:            <none>
```

可以看到服务名正是 alertmanager-main，Port 定义的名称也是 web，符合上面的规则，所以 Prometheus 和 AlertManager 组件就正确关联上了。而对应的报警规则文件位于：/etc/prometheus/rules/prometheus-k8s-rulefiles-0/目录下面所有的 YAML 文件。我们可以进入 Prometheus 的 Pod 中验证下该目录下面是否有 YAML 文件：

```shell
$ kubectl exec -it prometheus-k8s-0 /bin/sh -n monitoring
Defaulting container name to prometheus.
Use 'kubectl describe pod/prometheus-k8s-0 -n monitoring' to see all of the containers in this pod.
/prometheus $ ls /etc/prometheus/rules/prometheus-k8s-rulefiles-0/
monitoring-prometheus-k8s-rules.yaml
/prometheus $ cat /etc/prometheus/rules/prometheus-k8s-rulefiles-0/monitoring-pr
ometheus-k8s-rules.yaml
groups:
- name: k8s.rules
  rules:
  - expr: |
      sum(rate(container_cpu_usage_seconds_total{job="kubelet", image!="", container_name!=""}[5m])) by (namespace)
    record: namespace:container_cpu_usage_seconds_total:sum_rate
......
```

这个 YAML 文件实际上就是我们之前创建的一个 PrometheusRule 文件包含的：

```shell
$ cat prometheus-rules.yaml
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: prometheus-k8s-rules
  namespace: monitoring
spec:
  groups:
  - name: k8s.rules
    rules:
    - expr: |
        sum(rate(container_cpu_usage_seconds_total{job="kubelet", image!="", container_name!=""}[5m])) by (namespace)
      record: namespace:container_cpu_usage_seconds_total:sum_rate
```

我们这里的 PrometheusRule 的 name 为 prometheus-k8s-rules，namespace 为 monitoring，我们可以猜想到我们创建一个 PrometheusRule 资源对象后，会自动在上面的 prometheus-k8s-rulefiles-0 目录下面生成一个对应的<namespace>-<name>.yaml文件，所以如果以后我们需要自定义一个报警选项的话，只需要定义一个 PrometheusRule 资源对象即可。至于为什么 Prometheus 能够识别这个 PrometheusRule 资源对象呢？这就需要查看我们创建的 prometheus 这个资源对象了，里面有非常重要的一个属性 ruleSelector，用来匹配 rule 规则的过滤器，要求匹配具有 prometheus=k8s 和 role=alert-rules 标签的 PrometheusRule 资源对象，现在明白了吧？

```yaml
ruleSelector:
  matchLabels:
    prometheus: k8s
    role: alert-rules
```

所以我们要想自定义一个报警规则，只需要创建一个具有 prometheus=k8s 和 role=alert-rules 标签的 PrometheusRule 对象就行了，这里简单测试一个rule规则文件，[更多rules文件参考](https://github.com/yanghongfei/Kubernetes/tree/master/kube-prometheus/manifests/prometheus/prometheus_rules)

```yaml
$ cat prometheus-cpu-rules.yaml 
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: k8s
    role: alert-rules
  name: prometheus-cpu-rules
  namespace: monitoring
spec:
  groups:
  - name: 主机CPU监控
    rules:
    - alert: CPU利用率过高
      annotations:
        detail:  "{{$labels.instance}}: CPU利用率过高于75% (当前值: {{ $value }})"
        summary: "{{$labels.instance}}: CPU利用率过高"
      expr: |
        100 - (avg by (instance) (irate(node_cpu_seconds_total{job="node-exporter",mode="idle"}[5m])) * 100) > 75
      for: 1m
      labels:
        severity: 严重
    - name: 主机CPU Load15 监控
    rules:
    - alert: CPU Load 15分钟过高
      annotations:
        detail:  "{{$labels.instance}}: 15分钟内CPU Load 过高，(当前值: {{ $value }})"
        summary: "{{$labels.instance}}: 15分钟内CPU Load 过高"
      expr: |
        (node_load15) > 1     #根据你的主机核心数来定
      for: 1m
      labels:
        severity: 严重
```

注意 label 标签一定至少要有 prometheus=k8s 和 role=alert-rules，创建完成后，隔一会儿再去容器中查看下 rules 文件夹

![](https://github.com/yanghongfei/Kubernetes/blob/master/images/yanghongfei_rule.png)

可以看到我们创建的 rule 文件已经被注入到了对应的 rulefiles 文件夹下面了，证明我们上面的设想是正确的。然后再去 Prometheus Dashboard 的 Alert 页面下面就可以查看到上面我们新建的报警规则了



**配置报警**

  我们知道了如何去添加一个报警规则配置项，但是这些报警信息用怎样的方式去发送呢？前面的课程中我们知道我们可以通过 AlertManager 的配置文件去配置各种报警接收器，现在我们是通过 Operator 提供的 alertmanager 资源对象创建的组件，应该怎样去修改配置呢？ 

  首先我们将 alertmanager-main 这个 Service 改为 NodePort 类型的 Service，修改完成后我们可以在页面上的 status 路径下面查看 AlertManager 的默认配置信息。

这些配置信息实际上是来自于我们之前在prometheus-operator/contrib/kube-prometheus/manifests目录下面创建的 alertmanager-secret.yaml 文件：

```shell
apiVersion: v1
data:
  alertmanager.yaml: Imdsb2JhbCI6IAogICJyZXNvbHZlX3RpbWVvdXQiOiAiNW0iCiJyZWNlaXZlcnMiOiAKLSAibmFtZSI6ICJudWxsIgoicm91dGUiOiAKICAiZ3JvdXBfYnkiOiAKICAtICJqb2IiCiAgImdyb3VwX2ludGVydmFsIjogIjVtIgogICJncm91cF93YWl0IjogIjMwcyIKICAicmVjZWl2ZXIiOiAibnVsbCIKICAicmVwZWF0X2ludGVydmFsIjogIjEyaCIKICAicm91dGVzIjogCiAgLSAibWF0Y2giOiAKICAgICAgImFsZXJ0bmFtZSI6ICJEZWFkTWFuc1N3aXRjaCIKICAgICJyZWNlaXZlciI6ICJudWxsIg==
kind: Secret
metadata:
  name: alertmanager-main
  namespace: monitoring
type: Opaque
```

可以将 alertmanager.yaml 对应的 value 值做一个 base64 解码：

```yaml
$ echo "Imdsb2JhbCI6IAogICJyZXNvbHZlX3RpbWVvdXQiOiAiNW0iCiJyZWNlaXZlcnMiOiAKLSAibmFtZSI6ICJudWxsIgoicm91dGUiOiAKICAiZ3JvdXBfYnkiOiAKICAtICJqb2IiCiAgImdyb3VwX2ludGVydmFsIjogIjVtIgogICJncm91cF93YWl0IjogIjMwcyIKICAicmVjZWl2ZXIiOiAibnVsbCIKICAicmVwZWF0X2ludGVydmFsIjogIjEyaCIKICAicm91dGVzIjogCiAgLSAibWF0Y2giOiAKICAgICAgImFsZXJ0bmFtZSI6ICJEZWFkTWFuc1N3aXRjaCIKICAgICJyZWNlaXZlciI6ICJudWxsIg==" | base64 -d
"global":
  "resolve_timeout": "5m"
"receivers":
- "name": "null"
"route":
  "group_by":
  - "job"
  "group_interval": "5m"
  "group_wait": "30s"
  "receiver": "null"
  "repeat_interval": "12h"
  "routes":
  - "match":
      "alertname": "DeadMansSwitch"
    "receiver": "null"
```

我们可以看到内容和上面查看的配置信息是一致的，所以如果我们想要添加自己的接收器，或者模板消息，我们就可以更改这个文件：

```yaml
cat alertmanager.yaml 
global:
  resolve_timeout: 5m
  smtp_smarthost: 'smtp.163.com:25'
  smtp_from: 'yanghongfei97@163.com'
  smtp_auth_username: 'yanghongfei97@163.com'
  smtp_auth_password: '<email_password>'
  smtp_hello: '163.com'
  smtp_require_tls: false
route:
  group_by: ['job', 'severity']
  group_wait: 30s
  group_interval: 5m
  repeat_interval: 12h
  receiver: default
  routes:
  - receiver: webhook
    match:
      alertname: CoreDNSDown
receivers:
- name: 'default'
  email_configs:
  - to: '1923671815@qq.com'
    send_resolved: true
- name: 'webhook'
  webhook_configs:
  - url: 'https://oapi.dingtalk.com/robot/send?access_token=xxxxx'
    send_resolved: true
```

将上面文件保存为 alertmanager.yaml，然后使用这个文件创建一个 Secret 对象：

```shell
# 先将之前的 secret 对象删除
$ kubectl delete secret alertmanager-main -n monitoring
secret "alertmanager-main" deleted
$ kubectl create secret generic alertmanager-main --from-file=alertmanager.yaml -n monitoring
secret "alertmanager-main" created
```

**这时候再次确认配置已经被更改，如下图**

![](https://github.com/yanghongfei/Kubernetes/blob/master/images/altermanager_config.png)

**以上是自带规则，当然读到这里你已经知道了他的rules逻辑，你可以自己进行自定义规则**

这里是我的自定义规则面板，和测试结果，rules规则文件我也放到了`promethues-rules`目录中,[更多rules文件参考](https://github.com/yanghongfei/Kubernetes/tree/master/kube-prometheus/manifests/prometheus/prometheus_rules)

![](https://github.com/yanghongfei/Kubernetes/blob/master/images/prometheus_alerts.png)

![](https://github.com/yanghongfei/Kubernetes/raw/master/images/prometheus_rules.png)

**CPU 15分钟Load>2报警**

![](https://github.com/yanghongfei/Kubernetes/raw/master/images/email.png)


