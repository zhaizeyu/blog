title: quickstart
date: 2025-05-02
categories:
  - 学习笔记
  - 编程
  - k8s
  - operator
tags:
  - k8s
  - operator
---


在一个**全新的 CentOS 环境**中开发并运行一个 **Kubernetes Operator**，需要完成一系列安装、配置和开发步骤。下面是一个完整的流程指南，从系统准备到部署 Operator 到 Kind 集群。

---

# 🧩 总体流程

### 一、环境准备  
### 二、Kind 集群搭建  
### 三、安装开发工具（Kubebuilder 等）  
### 四、Operator 项目开发  
### 五、构建 & 加载 Operator 镜像  
### 六、部署 Operator 到 Kind  
### 七、测试自定义资源

---

## ✅ 一、环境准备（系统级）

### 1. 安装 Docker
```bash
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo yum install -y docker-ce docker-ce-cli containerd.io
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
newgrp docker
```

### 2. 安装 Go（建议 Go ≥ 1.19）
```bash
curl -LO https://go.dev/dl/go1.24.2.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.24.2.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:$HOME/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
source ~/.bashrc
go version
```

### 3. 安装 kubectl
```bash
curl -LO "https://dl.k8s.io/release/$(curl -Ls https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
kubectl version --client
```

### 4. 安装 kind
```bash
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.22.0/kind-linux-amd64
chmod +x kind
sudo mv kind /usr/local/bin/
kind version
```

---

## 🚀 二、创建 Kind 集群（测试 K8s 环境）

```bash
cat <<EOF > kind-config.yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
- role: control-plane
EOF

kind create cluster --name my-operator --config kind-config.yaml
kubectl cluster-info --context kind-my-operator
```

---

## 🔧 三、安装开发工具：Kubebuilder + controller-gen

### 1. 安装 Kubebuilder

```bash
# download kubebuilder and install locally.
curl -L -o kubebuilder "https://go.kubebuilder.io/dl/latest/$(go env GOOS)/$(go env GOARCH)"
chmod +x kubebuilder && sudo mv kubebuilder /usr/local/bin/
```

### 2. 验证
```bash
kubebuilder version
```

---

## 📁 四、开发一个 Operator 项目

```bash
mkdir -p ~/projects/guestbook
cd ~/projects/guestbook
kubebuilder init --domain my.domain --repo my.domain/guestbook
# 回答 y 以生成 API 和 Controller
kubebuilder create api --group webapp --version v1 --kind Guestbook
```

- 修改 API 定义文件：`api/v1/guestbook_types.go`
- 编写逻辑：`controllers/guestbook_controller.go`

---

## 🛠️ 五、构建并加载镜像进 Kind

### 1. 构建镜像

```bash
make docker-build IMG=my-operator:latest
```

### 2. 加载镜像到 Kind 集群中

```bash
kind load docker-image my-operator:latest --name my-operator
```

---

## 📦 六、部署 Operator 到 K8s 集群

```bash
make install            # 安装 CRD
make deploy IMG=my-operator:latest
kubectl get pods -n my-operator-system
```

---

## 🔍 七、测试你的 Operator

### 1. 创建 CR 实例

编辑 `guestbook.yaml` 文件，如下：

```yaml
apiVersion: webapp.my.domain/v1
kind: Guestbook
metadata:
  name: example-guestbook
spec:
  foo: "hello-world"
```

### 2. 应用测试资源：

```bash

kubectl apply -f guestbook.yaml
kubectl logs -nguestbook-system deploy/guestbook-controller-manager
```

---

## 🧪 本地运行调试（可选）

你也可以使用本地 Go 进程调试 Operator：

```bash
make install
make run
```

---

## ✅ 清理（可选）

```bash
kind delete cluster --name my-operator
```

---

## 📎 总结关键工具版本建议

| 工具         | 推荐版本     |
|--------------|--------------|
| Go           | ≥ 1.23       |
| Kubebuilder  | 4.5.2       |
| Kind         | ≥ 0.22.0     |
| Kubernetes   | 1.28+（Kind 默认）|

---

## PS：
完善controller基本功能后需要主机配置rbac，基础镜像等
rbac导致对pod的读写无权限，需要配置rbac内家role
默认很简单的基础镜像内部无ls，cat，bash等命令