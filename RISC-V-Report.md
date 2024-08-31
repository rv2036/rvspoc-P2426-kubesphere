# RISC-V 移植调研

## 所需 Image List (排除目前已有)

### kube-system
- [kube-rbac-proxy](https://github.com/brancz/kube-rbac-proxy)

### kubesphere-system
- [ks-installer](https://github.com/kubesphere/ks-installer)
- [ks-console](https://github.com/kubesphere/console)
- [ks-controller-manager, ks-apiserver](https://github.com/kubesphere/kubesphere)

### kubesphere-controls-system
- [kubectl](https://github.com/kubesphere/kubectl)
- defaultbackend(构筑内容可以是纯静态文件，与架构无关)
- [snapshot-controller](https://github.com/openebs/dynamic-localpv-provisioner)
- [provisioner-localpv](https://github.com/openebs/dynamic-localpv-provisioner)
- [k8s-dns-node-cache](https://github.com/kubernetes/dns)

### kubesphere-monitoring-system
- [alertmanager](https://github.com/prometheus/alertmanager)
- [node-exporter](https://github.com/prometheus/node_exporter) 
- [kube-state-metrics](https://github.com/kubernetes/kube-state-metrics)
- [promethues](https://github.com/prometheus/prometheus)
- [prometheus-operator, prometheus-config-reloader](https://github.com/prometheus-operator/prometheus-operator)
- [notification-manager, notification-manager-operator, notification-tenant-sidecar](https://github.com/kubesphere/notification-manager)

### 版本参考 (kubernetes v1.29.1, kubesphere v3.4.1)
```shell
root@kube-master:~# crictl images
IMAGE                                                                         TAG                 IMAGE ID            SIZE
registry.cn-beijing.aliyuncs.com/kubesphereio/alertmanager                    v0.23.0             ba2b418f427c0       26.5MB
registry.cn-beijing.aliyuncs.com/kubesphereio/coredns                         1.9.3               5185b96f0becf       14.8MB
registry.cn-beijing.aliyuncs.com/kubesphereio/defaultbackend-amd64            1.4                 846921f0fe0e5       1.82MB
registry.cn-beijing.aliyuncs.com/kubesphereio/flannel-cni-plugin              v1.1.2              7a2dcab94698c       3.84MB
registry.cn-beijing.aliyuncs.com/kubesphereio/flannel                         v0.21.3             0d004b381af6c       24.2MB
registry.cn-beijing.aliyuncs.com/kubesphereio/k8s-dns-node-cache              1.22.20             ff71cd4ea5ae5       30.5MB
registry.cn-beijing.aliyuncs.com/kubesphereio/ks-apiserver                    v3.4.1              c486abe6f1cc8       65.8MB
registry.cn-beijing.aliyuncs.com/kubesphereio/ks-console                      v3.4.1              aa81987f764d3       51.7MB
registry.cn-beijing.aliyuncs.com/kubesphereio/ks-controller-manager           v3.4.1              2a2294b6c6af0       50.3MB
registry.cn-beijing.aliyuncs.com/kubesphereio/ks-installer                    v3.4.1              d6ce52546e1c3       156MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kube-apiserver                  v1.29.1             53b148a9d1963       35.1MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kube-controller-manager         v1.29.1             79d451ca186a6       33.4MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kube-proxy                      v1.29.1             43c6c10396b89       28.4MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kube-rbac-proxy                 v0.11.0             29589495df8d9       19.2MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kube-scheduler                  v1.29.1             406945b511542       18.5MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kube-state-metrics              v2.6.0              ec6e2d871c544       12MB
registry.cn-beijing.aliyuncs.com/kubesphereio/kubectl                         v1.22.0             30c7baa8e18c0       26.6MB
registry.cn-beijing.aliyuncs.com/kubesphereio/linux-utils                     3.3.0               e88cfb3a763b9       26.9MB
registry.cn-beijing.aliyuncs.com/kubesphereio/node-exporter                   v1.3.1              1dbe0e9319764       10.3MB
registry.cn-beijing.aliyuncs.com/kubesphereio/notification-manager-operator   v2.3.0              7ffe334bf3772       19.3MB
registry.cn-beijing.aliyuncs.com/kubesphereio/notification-manager            v2.3.0              2c35ec9a2c185       21.6MB
registry.cn-beijing.aliyuncs.com/kubesphereio/notification-tenant-sidecar     v3.2.0              4b47c43ec6ab6       14.7MB
registry.cn-beijing.aliyuncs.com/kubesphereio/pause                           3.9                 e6f1816883972       321kB
registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus-config-reloader      v0.55.1             7c63de88523a9       4.84MB
registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus-operator             v0.55.1             b30c215b787f5       14.3MB
registry.cn-beijing.aliyuncs.com/kubesphereio/prometheus                      v2.39.1             6b9895947e9e4       88.5MB
registry.cn-beijing.aliyuncs.com/kubesphereio/provisioner-localpv             3.3.0               739e82fed8b2c       28.8MB
registry.cn-beijing.aliyuncs.com/kubesphereio/snapshot-controller             v4.0.0              f1d8a00ae690f       19MB
```

## 所需配置
网络层的默认配置需要从 Calico 切换为 Flannel

```yaml
apiVersion: kubekey.kubesphere.io/v1alpha2
kind: Cluster
metadata:
  name: all-in-one
spec:
  hosts:
  ##You should complete the ssh information of the hosts
  - {name: kube-master, address: 192.168.30.21, internalAddress: 192.168.30.21, user: root, password: "2002"}
  roleGroups:
    etcd:
    - kube-master
    master:
    - kube-master
    worker:
    - kube-master
  controlPlaneEndpoint:
    ##Internal loadbalancer for apiservers
    #internalLoadbalancer: haproxy
    ##If the external loadbalancer was used, 'address' should be set to loadbalancer's ip.
    domain: lb.kubesphere.local
    address: "192.168.30.21"
    port: 6443
  kubernetes:
    version: v1.29.1
    clusterName: cluster.local
    proxyMode: ipvs
    masqueradeAll: false
    maxPods: 110
    nodeCidrMaskSize: 24
  network:
    plugin: flannel # chang to flannel
    kubePodsCIDR: 10.233.64.0/18
    kubeServiceCIDR: 10.233.0.0/18
  registry:
    privateRegistry: ""

```
