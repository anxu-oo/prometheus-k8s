### 部署prometheus 监控k8s

>### 资源类型注解：
>
>prometheusrules  监控报警规则
>
>prometheuses     prometheuses实例
>
>servicemonitors  监控资源配置

##### 版本要求

`当我们在各自的分支中针对这些版本进行测试时，支持并运行以下版本。但请注意，其他版本可能有效！`

| kube-prometheus 堆栈                                         | Kubernetes 1.18 | Kubernetes 1.19 | Kubernetes 1.20 | Kubernetes 1.21 | Kubernetes 1.22 |
| ------------------------------------------------------------ | --------------- | --------------- | --------------- | --------------- | --------------- |
| [`release-0.6`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.6) | ✗               | ✔               | ✗               | ✗               | ✗               |
| [`release-0.7`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.7) | ✗               | ✔               | ✔               | ✗               | ✗               |
| [`release-0.8`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.8) | ✗               | ✗               | ✔               | ✔               | ✗               |
| [`release-0.9`](https://github.com/prometheus-operator/kube-prometheus/tree/release-0.9) | ✗               | ✗               | ✗               | ✔               | ✔               |
| [`main`](https://github.com/prometheus-operator/kube-prometheus/tree/main) | ✗               | ✗               | ✗               | ✔               | ✔               |

###### 部署前准备

```shell
#1.时间同步
yum -y install ntp ntpdate
ntpdate ntp.aliyun.com
hwclock --systohc
yum install -y chrony
systemctl restart chronyd ; systemctl enable chronyd
```

###### 安装helm

```shell
curl -L -o helm-v3.2.4-linux-amd64.tar.gz https://file.choerodon.com.cn/kubernetes-helm/v3.2.4/helm-v3.2.4-linux-amd64.tar.gz
tar -zxvf helm-v3.2.4-linux-amd64.tar.gz
sudo mv linux-amd64/helm /usr/bin/helm
helm version
```

###### 部署动态存储卷

```shell
#每个节点安装
sudo yum install -y nfs-utils
#master机器执行
mkdir -p /data/nfs_data
vim /etc/exports
/data/nfs_data xxx.xxx.0.0/16(rw,sync,insecure,no_subtree_check,no_root_squash)

sudo systemctl enable nfs-server
sudo systemctl start nfs-server
showmount -e '192.168.x.x'
helm repo add c7n https://openchart.choerodon.com.cn/choerodon/c7n/
helm upgrade --install nfs-client-provisioner c7n/nfs-client-provisioner \
  --set rbac.create=true \
  --set persistence.enabled=true \
  --set storageClass.name=nfs-provisioner \
  --set persistence.nfsServer=xxx.xxx.xxx.xxx \
  --set persistence.nfsPath=/data/nfs_data \
  --version 0.1.1 \
  --namespace kube-system
```

##### 拉取git

```shell
git clone https://github.com/anxu-oo/prometheus-k8s.git

```

```shell
#部署crd资源
cd prometheus/manifests/setup
kubectl apply --server-side -f .
#创建etcd密钥
kubectl -n monitoring create secret generic etcd-certs --from-file=/etc/kubernetes/pki/etcd/ca.crt --from-file=/etc/kubernetes/pki/etcd/peer.crt    --from-file=/etc/kubernetes/pki/etcd/peer.key
#部署组件
cd /data/prometheus/manifests
kubectl apply -f .
cd prometheus
#修改etcd地址
vim etcd-svc.yaml  
#修改kube-proxy地址
kube-proxy-svc.yaml
#修改钉钉地址 dingtalk.yaml  
vim dingtalk.yaml
kubectl apply -f .
```

##### 配置`nginx-ingress svc ` 使endpoints可以自动发现

```shell
#添加metrics端口 
kubectl -n ingress-controller  edit  svc ingress-nginx-controller
  - name: metrics
    port: 10254
    protocol: TCP
    targetPort: 10254
```

##### 配置报警规则

```shell
cd prometheus/rule
kubectl apply -f .
```

