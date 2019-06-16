## CNI Plugin部署
cni plugin 0.8.0以上支持bandwidth插件，最新为0.8.1
1. 修改cni配置文件，添加bandwidth插件
```json
{
  "name": "k8s-pod-network",
  "cniVersion": "0.3.0",
  "plugins": [
    {
      "type": "calico",
      "log_level": "info",
      "datastore_type": "kubernetes",
      "nodename": "127.0.0.1",
      "ipam": {
        "type": "host-local",
        "subnet": "usePodCidr"
      },
      "policy": {
        "type": "k8s"
      },
      "kubernetes": {
        "kubeconfig": "/etc/cni/net.d/calico-kubeconfig"
      }
    },
    {
      "type": "bandwidth",
      "capabilities": {"bandwidth": true}
    }
  ]
}
```

2. 下载bandwidtch插件，拷贝到/opt/cni/bin（默认）目录下。  
[calico cni-plugin v0.8.1 下载地址](https://github.com/containernetworking/plugins/releases/download/v0.8.1/cni-plugins-linux-amd64-v0.8.1.tgz)

## k8s 版本支持情况
目前1.13以上版本可以支持流量控制，但在kubelet中带宽单位的转换存在[bug](https://github.com/kubernetes/kubernetes/commit/f4937619a2228a71ac62270d643dc869d3765d99)。
该bug但会导致用户设置的带宽与实际限制的带宽不同。 预计k8s 1.15版本后才会修复改bug。

1.11版本的k8s不支持带宽限制特性。  
1.12版本的k8s kubelet中将burst值强制设为0导致带宽限制特性不可用([相关源码](https://github.com/kubernetes/kubernetes/blob/6f482974b76db3f1e0f5d24605a9d1d38fad9a2b/pkg/kubelet/dockershim/network/cni/cni.go#L382))。 

kubelet报错信息：  
```
E0612 03:47:27.233298    6946 cni.go:310] Error adding network: if rate is set, burst must also be set
E0612 03:47:27.233322    6946 cni.go:278] Error while adding to cni network: if rate is set, burst must also be set
```
