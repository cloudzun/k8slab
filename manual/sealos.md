
下载sealos 4.0 
```bash
wget https://chengzhstor.blob.core.windows.net/k8slab/sealos_4.0.0_linux_amd64.tar.gz 
tar zxvf sealos_4.0.0_linux_amd64.tar.gz 
chmod +x sealos 
mv sealos /usr/bin
```

生成Clusterfile文件，一个单master两node的kubernetes集群
```bash
sealos gen labring/kubernetes:v1.24.0 labring/calico:v3.22.1 --masters 192.168.1.231 --nodes 192.168.1.232,192.168.1.233 --passwd 2wsx#EDC > Clusterfile
```

编辑Clusterfile
nano Clusterfile

```yaml
apiVersion: apps.sealos.io/v1beta1
kind: Cluster
metadata:
  creationTimestamp: null
  name: default
spec:
  hosts:
  - ips:
    - 192.168.1.231:22
    roles:
    - master
    - amd64
  - ips:
    - 192.168.1.232:22
    - 192.168.1.233:22
    roles:
    - node
    - amd64
  image:
  - labring/kubernetes:v1.24.0
  - labring/calico:v3.22.1
  ssh:
    passwd: 2wsx#EDC
    pk: /root/.ssh/id_rsa
    port: 22
    user: root
status: {} # 将以下内容复制到Clusterfile中
---
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
networking:
  podSubnet: 10.244.0.0/16
---
apiVersion: apps.sealos.io/v1beta1
kind: Config
metadata:
  name: calico
spec:
  path: manifests/calico.yaml
  data: |
    apiVersion: operator.tigera.io/v1
    kind: Installation
    metadata:
      name: default
    spec:
      # Configures Calico networking.
      calicoNetwork:
        # Note: The ipPools section cannot be modified post-install.
        ipPools:
        - blockSize: 26
          # Note: Must be the same as podCIDR
          cidr: 10.244.0.0/16
          encapsulation: IPIP
          natOutgoing: Enabled
          nodeSelector: all()
        nodeAddressAutodetectionV4:
          interface: "eth.*|en.*"
```

#下载Clusterfile
wget https://raw.githubusercontent.com/cloudzun/k8slab/v1.23/manual/Clusterfile

创建群集
sealos apply -f Clusterfile

删除所有master污点，使其能承载工作负载
kubectl taint nodes --all node-role.kubernetes.io/master-
kubectl taint nodes --all node-role.kubernetes.io/control-plane-


















以下内容属于旧版本已失效，忽略

# 下载并安装sealos, sealos是个golang的二进制工具，直接下载拷贝到bin目录即可, release页面也可下载
wget -c https://sealyun-home.oss-cn-beijing.aliyuncs.com/sealos/latest/sealos && \
    chmod +x sealos && mv sealos /usr/bin 

# 下载离线资源包
wget -c https://sealyun.oss-cn-beijing.aliyuncs.com/05a3db657821277f5f3b92d834bbaf98-v1.22.0/kube1.22.0.tar.gz


# 安装一个单master两node的kubernetes集群
sealos init --passwd '2wsx#EDC' \
	--master 192.168.1.231   \
	--node 192.168.1.232  --node 192.168.1.233 \
	--pkg-url /root/kube1.22.0.tar.gz \
        --podcidr 10.244.0.0/16  \
	--version v1.22.0

#清理
sealos clean --all 


#适用场景
1.需要快速安装k8s测试群集(可能版本是次新的)
2.离线安装



https://github.com/fanux/sealos
