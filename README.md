### 部署prometheus 监控k8s

```shell
#git clone

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

