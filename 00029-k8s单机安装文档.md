# Debian 单机版 Kubernetes 安装文档（containerd + kubeadm + flannel）

> 适用环境：Debian 12 / 单机测试环境 / 普通用户部署 / 国内网络环境  
> 本文档按本次实际安装与排障过程整理，包含：用户创建、containerd 配置、kubeadm 初始化、flannel 网络、国内镜像处理、常见故障处理。

---

## 1. 环境说明

### 1.1 系统信息

- 系统：Debian
- 容器运行时：containerd 1.6.x
- Kubernetes：v1.35.4
- 网络插件：flannel
- Pod 网段：`10.244.0.0/16`
- API Server 地址示例：`192.168.1.188`

### 1.2 机器要求

最低建议：

- CPU：2 核
- 内存：4GB
- 磁盘：20GB+
- 关闭 swap

实际本次环境中：

- 内存 31Gi
- swap 已关闭
- 节点最终正常 Ready

---

## 2. 新建部署用户

root 执行：

```bash
useradd -m -s /bin/bash k8s
echo 'k8s:K8s@123456' | chpasswd
usermod -aG sudo k8s

cat >/etc/sudoers.d/99-k8s <<'EOF2'
k8s ALL=(ALL) NOPASSWD:ALL
EOF2
chmod 440 /etc/sudoers.d/99-k8s
```

切换用户：

```bash
su - k8s
```

---

## 3. 系统初始化

### 3.1 关闭 swap

```bash
sudo swapoff -a
sudo sed -ri 's@^([^#].*\sswap\s+.*)$@#\1@' /etc/fstab
free -h
grep -n swap /etc/fstab
```

确认：

- `free -h` 中 `交换` 为 `0B`
- `/etc/fstab` 中 swap 行已注释

---

### 3.2 加载内核模块

```bash
sudo modprobe overlay
sudo modprobe br_netfilter

cat <<'EOF' | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```

---

### 3.3 配置内核参数

```bash
cat <<'EOF' | sudo tee /etc/sysctl.d/99-k8s.conf
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward = 1
EOF

sudo sysctl --system
```

校验：

```bash
sudo sysctl net.bridge.bridge-nf-call-iptables
sudo sysctl net.ipv4.ip_forward
```

应为：

```bash
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

> 如果普通用户提示 `sysctl: 未找到命令`，直接用 `sudo sysctl ...` 即可。Debian 普通用户 PATH 里可能不带 `/usr/sbin`。

---

## 4. 安装 containerd

```bash
sudo apt update
sudo apt install -y containerd
```

生成默认配置：

```bash
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml >/dev/null
```

修改关键配置：

```bash
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo sed -i 's#sandbox_image = ".*"#sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.10.1"#' /etc/containerd/config.toml
sudo sed -i 's/disabled_plugins = \["cri"\]/disabled_plugins = \[\]/' /etc/containerd/config.toml
```

校验：

```bash
grep -n 'sandbox_image\|SystemdCgroup\|disabled_plugins' /etc/containerd/config.toml
```

应至少看到：

```bash
1:disabled_plugins = []
61:    sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.10.1"
125:            SystemdCgroup = true
```

启动服务：

```bash
sudo systemctl enable --now containerd
sudo systemctl restart containerd
sudo systemctl status containerd --no-pager -l
```

验证：

```bash
sudo ctr version
sudo ctr -n k8s.io images pull registry.aliyuncs.com/google_containers/pause:3.10.1
```

---

## 5. 安装 kubeadm / kubelet / kubectl

```bash
sudo apt update
sudo apt install -y apt-transport-https ca-certificates curl gpg

sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.35/deb/Release.key \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

cat <<'EOF' | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.35/deb/ /
EOF

sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

验证：

```bash
kubeadm version
kubectl version --client
```

---

## 6. 国内环境预拉镜像

```bash
sudo kubeadm config images list
sudo kubeadm config images pull --image-repository registry.aliyuncs.com/google_containers
```

---

## 7. 初始化集群

### 7.1 干净环境初始化

如果之前反复失败过，建议先彻底清理：

```bash
sudo kubeadm reset -f
sudo systemctl stop kubelet
sudo systemctl stop containerd

sudo pkill -9 -f containerd-shim-runc-v2 || true
sudo pkill -9 -f containerd-shim || true
sudo pkill -9 -f runc || true

sudo rm -rf /var/lib/etcd
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/kubelet
sudo rm -rf /etc/cni/net.d
sudo rm -rf /var/lib/containerd/*
sudo rm -rf /run/containerd/*

sudo systemctl daemon-reload
sudo systemctl start containerd
sudo systemctl start kubelet
```

检查残留：

```bash
sudo ctr -n k8s.io containers ls
sudo ctr -n k8s.io tasks ls
```

正常应为空。

---

### 7.2 kubeadm init

```bash
sudo kubeadm init \
  --apiserver-advertise-address=$(hostname -I | awk '{print $1}') \
  --pod-network-cidr=10.244.0.0/16 \
  --image-repository registry.aliyuncs.com/google_containers \
  --v=5
```

> 注意：containerd 1.6.x 会提示 CRI RuntimeConfig 警告，这不是当前阻塞项，可先忽略：
>
> ```bash
> [WARNING ContainerRuntimeVersion]: You must update your container runtime...
> ```

---

## 8. 配置 kubectl

```bash
mkdir -p $HOME/.kube
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

查看当前上下文：

```bash
kubectl config view
```

---

## 9. 安装 flannel 网络插件

### 9.1 国内代理下载 YAML

> 由于国内访问 GitHub 经常超时，建议先下载到本地。

```bash
curl -L "https://gh-proxy.com/https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml" -o kube-flannel.yml
```

首次安装：

```bash
kubectl apply -f kube-flannel.yml
```

如果出现：

```bash
The DaemonSet "kube-flannel-ds" is invalid: spec.selector ... field is immutable
```

说明之前已经创建过旧版 flannel，直接删掉 DaemonSet 再重建：

```bash
kubectl -n kube-flannel delete daemonset kube-flannel-ds
kubectl apply -f kube-flannel.yml
```

---

### 9.2 验证 flannel

```bash
kubectl get pods -n kube-flannel -o wide
kubectl describe -n kube-flannel $(kubectl get pod -n kube-flannel -o name | head -n1)
```

正常状态：

- `install-cni-plugin` -> `Completed`
- `install-cni` -> `Completed`
- `kube-flannel` -> `Running`
- `Ready -> True`

---

## 10. 去掉单机节点 taint

单机控制节点默认不可调度普通 Pod，需要去掉 taint：

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane- || true
kubectl taint nodes --all node-role.kubernetes.io/master- || true
```

---

## 11. 验证节点与系统组件

```bash
kubectl get nodes
kubectl get pods -A -o wide
```

成功状态示例：

- `debian188   Ready   control-plane`
- `kube-flannel` Running
- `kube-proxy` Running
- `etcd` Running
- `kube-apiserver` Running
- `kube-controller-manager` Running
- `kube-scheduler` Running

如果 `coredns` 还是 `0/1 Running`，可重建：

```bash
kubectl delete pod -n kube-system -l k8s-app=kube-dns
kubectl get pods -n kube-system -o wide
```

---

## 12. 验证业务 Pod

### 12.1 创建 nginx

> 国内环境不要直接拉 `docker.io/library/nginx:latest`，容易超时。

错误现象：

```bash
ImagePullBackOff
failed to pull image "docker.io/library/nginx:latest"
```

本次最终验证使用可拉通镜像：

```bash
kubectl create deployment nginx --image=docker.m.daocloud.io/library/nginx:1.27-alpine
kubectl expose deployment nginx --port=80 --type=NodePort
```

如果 deployment 已存在，可改镜像：

```bash
kubectl get deploy nginx -o yaml | grep -A5 'containers:'
```

> 注意先看容器名，不要默认写死为 nginx。

如果容器名就是 `nginx`，则：

```bash
kubectl set image deployment/nginx nginx=docker.m.daocloud.io/library/nginx:1.27-alpine
kubectl rollout restart deployment/nginx
```

查看：

```bash
kubectl get pods -o wide
kubectl get svc
```

成功状态：

```bash
nginx-xxxxx   1/1 Running
```

访问：

```bash
curl http://192.168.1.188:30406
```

> 端口 `30406` 仅为示例，实际以 `kubectl get svc` 输出为准。

---

## 13. 常见问题与处理

### 13.1 `pause` 镜像不一致

报错示例：

```bash
detected that the sandbox image ... is inconsistent
```

处理：

```bash
sudo sed -i 's#sandbox_image = ".*"#sandbox_image = "registry.aliyuncs.com/google_containers/pause:3.10.1"#' /etc/containerd/config.toml
sudo systemctl restart containerd
```

---

### 13.2 `server is not initialized yet`

报错示例：

```bash
validate CRI v1 runtime API ... server is not initialized yet
```

一般原因：

- containerd 没启动完成
- CRI 配置错误
- 残留 shim / sandbox 没清理

处理：

```bash
sudo systemctl stop kubelet
sudo systemctl stop containerd
sudo pkill -9 -f containerd-shim-runc-v2 || true
sudo pkill -9 -f containerd-shim || true
sudo pkill -9 -f runc || true
sudo rm -rf /var/lib/containerd/*
sudo rm -rf /run/containerd/*
sudo systemctl start containerd
```

---

### 13.3 kube-apiserver / etcd 反复不健康

排查：

```bash
sudo crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock ps -a
sudo crictl --runtime-endpoint unix:///var/run/containerd/containerd.sock logs <容器ID>
```

也可查看：

```bash
sudo journalctl -u containerd -n 200 --no-pager
sudo journalctl -u kubelet -n 200 --no-pager
```

---

### 13.4 flannel 卡住

检查：

```bash
kubectl describe -n kube-flannel $(kubectl get pod -n kube-flannel -o name | head -n1)
kubectl logs -n kube-flannel $(kubectl get pod -n kube-flannel -o name | head -n1) -c install-cni-plugin
kubectl logs -n kube-flannel $(kubectl get pod -n kube-flannel -o name | head -n1) -c install-cni
```

确保目录存在：

```bash
sudo mkdir -p /opt/cni/bin
sudo mkdir -p /etc/cni/net.d
sudo chmod 755 /opt/cni/bin /etc/cni/net.d
sudo systemctl restart containerd
sudo systemctl restart kubelet
```

---

### 13.5 `kubectl describe pod $(kubectl get pod -o name ...)` 报错

错误写法：

```bash
kubectl describe pod $(kubectl get pod -o name | grep nginx)
```

因为 `-o name` 已经返回：

```bash
pod/nginx-xxxx
```

正确写法：

```bash
kubectl describe $(kubectl get pod -o name | grep nginx)
```

---

### 13.6 crictl warning 太多

配置默认 endpoint：

```bash
cat <<'EOF' | sudo tee /etc/crictl.yaml
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 10
debug: false
EOF
```

---

## 14. 最终验证命令汇总

```bash
kubectl get nodes
kubectl get pods -A -o wide
kubectl get svc
```

业务验证：

```bash
kubectl create deployment nginx --image=docker.m.daocloud.io/library/nginx:1.27-alpine
kubectl expose deployment nginx --port=80 --type=NodePort
kubectl get pods -o wide
kubectl get svc
curl http://<节点IP>:<NodePort>
```

---

## 15. 本次最终结果

本次实际完成状态：

- containerd 正常
- kubeadm 初始化成功
- flannel 正常 Running
- 节点 `Ready`
- nginx 业务 Pod 最终 `1/1 Running`

即：**Debian 单机版 Kubernetes 集群部署成功**。
