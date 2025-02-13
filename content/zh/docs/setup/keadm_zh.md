---
draft: false
linktitle: 使用Keadm进行部署
menu:
  docs:
    parent: setup
    weight: 1
title: 使用Keadm进行部署
toc: true
type: docs
---
Keadm用于安装KubeEdge的云端和边缘端组件。它不负责K8s的安装和运行。

请参考 [kubernetes-compatibility](https://github.com/kubeedge/kubeedge#kubernetes-compatibility)
了解 **Kubernetes** 兼容性来确定安装哪个版本的Kubernetes。

## 使用限制

- `keadm` 目前支持 Ubuntu 和 CentOS OS。RaspberryPi的支持正在进行中。
- 需要超级用户权限（或root权限）才能运行。

## 设置云端（KubeEdge主节点）

### keadm init

默认情况下边缘节点需要访问cloudcore中 `10000` ，`10002` 端口。

`keadm init` 将安装 cloudcore，生成证书并安装CRD。它还提供了一个命令行参数，通过它可以设置特定的版本。

**重要提示：**

1. 必须正确配置 kubeconfig 或 master 中的至少一个，以便可以将其用于验证k8s集群的版本和其他信息。
2. 请确保边缘节点可以使用云节点的本地IP连接云节点，或者需要使用 `--advertise-address` 标记指定云节点的公共IP 。
3. `--advertise-address`（仅从1.3版本开始可用）是云端公开的地址（将添加到CloudCore证书的SAN中），默认值为本地IP。
4. `keadm init` 将会使用二进制方式部署 cloudcore 为一个系统服务，如果您想实现容器化部署，可以参考 `keadm beta init` 。

举个例子：

```shell
# keadm init --advertise-address="THE-EXPOSED-IP"(only work since 1.3 release)
```

输出：

```
Kubernetes version verification passed, KubeEdge installation will start...
...
KubeEdge cloudcore is running, For logs visit:  /var/log/kubeedge/cloudcore.log
```

### keadm beta init

现在可以使用 `keadm beta init` 进行云端组件安装。

举个例子:

```shell
# keadm beta init --advertise-address="THE-EXPOSED-IP" --set cloudcore-tag=v1.9.0 --kube-config=/root/.kube/config
```

**IMPORTANT NOTE:**

1. 自定义 `--set key=value`
   值可以参考 [KubeEdge Cloudcore Helm Charts README.md](https://github.com/kubeedge/kubeedge/blob/master/build/helm/charts/cloudcore/README.md)
2. 您可以从 Keadm 的一个内置配置概要文件开始，然后根据您的特定需求进一步定制配置。目前，内置的配置概要文件关键字是 `version`
   。请参考 [`version.yaml`](https://github.com/kubeedge/kubeedge/blob/master/build/helm/charts/profiles/version.yaml)
   ，您可以在这里创建您的自定义配置文件, 使用 `--profile version=v1.9.0 --set key=value` 来使用它。

此外，还可使用 `--external-helm-root` 安装外部的 helm chart 组件，如 edgemesh 。

举个例子:

```shell
# keadm beta init --set server.advertiseAddress="THE-EXPOSED-IP" --set server.nodeName=allinone  --kube-config=/root/.kube/config --force --external-helm-root=/root/go/src/github.com/edgemesh/build/helm --profile=edgemesh
```

如果您对 Helm Chart 比较熟悉，可以直接参考 [KubeEdge Helm Charts](https://github.com/kubeedge/kubeedge/tree/master/build/helm/charts)
进行安装。

### keadm beta manifest generate

`keadm beta manifest generate` 可以帮助我们快速渲染生成期望的 manifests 文件，并输出在终端显示。

Example:

```shell
# keadm beta manifest generate --advertise-address="THE-EXPOSED-IP" --kube-config=/root/.kube/config > kubeedge-cloudcore.yaml
```

> 使用 --skip-crds 跳过打印 CRDs

## 设置边缘端（KubeEdge工作节点）

### 从云端获取令牌

在**云端**运行 `keadm gettoken` 将返回token令牌，该令牌将在加入边缘节点时使用。

```shell
# keadm gettoken
27a37ef16159f7d3be8fae95d588b79b3adaaf92727b72659eb89758c66ffda2.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTAyMTYwNzd9.JBj8LLYWXwbbvHKffJBpPd5CyxqapRQYDIXtFZErgYE
```

### 加入边缘节点

#### keadm join

`keadm join` 将安装 edgecore 和 mqtt。它还提供了一个命令行参数，通过它可以设置特定的版本。

举个例子：

```shell
# keadm join --cloudcore-ipport=192.168.20.50:10000 --token=27a37ef16159f7d3be8fae95d588b79b3adaaf92727b72659eb89758c66ffda2.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTAyMTYwNzd9.JBj8LLYWXwbbvHKffJBpPd5CyxqapRQYDIXtFZErgYE
```

#### keadm beta join

现在可以使用 `keadm beta join` 通过镜像下载所需资源，进行节点接入。

##### Docker

```shell
# keadm beta join --cloudcore-ipport=192.168.20.50:10000 --token=27a37ef16159f7d3be8fae95d588b79b3adaaf92727b72659eb89758c66ffda2.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTAyMTYwNzd9.JBj8LLYWXwbbvHKffJBpPd5CyxqapRQYDIXtFZErgYE
```

##### CRI

```shell
# keadm beta join --cloudcore-ipport=192.168.20.50:10000 --runtimetype remote --remote-runtime-endpoint unix:///run/containerd/containerd.sock --token=27a37ef16159f7d3be8fae95d588b79b3adaaf92727b72659eb89758c66ffda2.eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE1OTAyMTYwNzd9.JBj8LLYWXwbbvHKffJBpPd5CyxqapRQYDIXtFZErgYE
```

**重要提示：**

1. `--cloudcore-ipport` 是必填参数。
2. 加上 `--token` 会自动为边缘节点生成证书，如果您需要的话。
3. 需要保证云和边缘端使用的KubeEdge版本相同。
4. 加上 `--with-mqtt` 会自动为边缘节点以容器运行的方式部署 `mosquitto` 服务 

输出：

```shell
Host has mosquit+ already installed and running. Hence skipping the installation steps !!!
...
KubeEdge edgecore is running, For logs visit:  /var/log/kubeedge/edgecore.log
```

=======
> 也可以使用 `keadm beta join` 来添加边缘节点。

### 在云端支持 Metrics-server

1. 实现该功能点的是重复使用了 cloudstream 和 edgestream 模块。因此，您还需要执行 *启用 `kubectl logs` 功能* 的所有步骤。

2. 由于边缘节点和云节点的 kubelet 端口不同，故当前版本的 metrics-server（0.3.x）不支持自动端口识别（这是0.4.0功能），因此您现在需要手动编译从master分支拉取的镜像。

   Git clone 最新的 metrics server 代码仓:

    ```bash
    git clone https://github.com/kubernetes-sigs/metrics-server.git
    ```

   转到 metrics server 目录:

    ```bash
    cd metrics-server
    ```

   制作 docker 容器:

    ```bash
    make container
    ```

   检查您是否有此 docker 镜像：

    ```bash
    docker images
    ```

   |                  仓库                           |                    标签                   |   镜像ID   |     创建时间     |  大小  |
   |-------------------------------------------------------|------------------------------------------|--------------|----------------|--------|
   | gcr.io/k8s-staging-metrics-serer/ metrics-serer-amd64 | 6d92704c5a68cd29a7a81bce68e6c2230c7a6912 | a24f71249d69 | 19秒前 | 57.2MB |
   | metrics-server-kubeedge                               |                 latest                   | aef0fa7a834c | 28秒前 | 57.2MB |

    确保您使用镜像ID来对镜像标签进行变更，以使其与yaml文件中的镜像名称一致。

    ```bash
    docker tag a24f71249d69 metrics-server-kubeedge:latest
    ```

3. 部署yaml应用。可以参考相关部署文档：https://github.com/kubernetes-sigs/metrics-server/tree/master/manifests。

   注意：下面的那些iptables必须应用在机器上（精确地是网络名称空间，因此metrics-server也需要在主机网络模式下运行）metric-server在其上运行。

   **注意：** 下面的那些iptables必须应用在已运行metric-server 机器上（精确地命名是网络名称空间，因此metrics-server也需要在主机网络模式下运行）
    ```
    iptables -t nat -A OUTPUT -p tcp --dport 10350 -j DNAT --to $CLOUDCOREIPS:10003
    ```

   （引导对metric-data的请求edgecore:10250至在cloudcore和edgecore之间的隧道中，iptables至关重要。）

   在部署 metrics-server 之前，必须确保将其部署在已部署apiserver的节点上。在这种情况下，这就是master节点。作为结果，需要通过以下命令使主节点可调度：

    ``` shell
    kubectl taint nodes --all node-role.kubernetes.io/master-
    ```

   然后，在 deployment.yaml 文件中，必须指定 metrics-server 部署在主节点上。（选择主机名作为标记的标签。）

   在**metrics-server-deployment.yaml**中
    ``` yaml
        spec:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                - matchExpressions:
                  #Specify which label in [kubectl get nodes --show-labels] you want to match
                  - key: kubernetes.io/hostname
                    operator: In
                    values:
                    #Specify the value in key
                    - charlie-latest
    ```

**重要提示：**

1. Metrics-server需要使用主机网络网络模式。

2. 使用您自己编译的镜像，并将 imagePullPolicy 设置为Never。

3. 为 Metrics 服务器启用 --kubelet-use-node-status-port 功能

   需要将这些设置写入部署yaml（metrics-server-deployment.yaml）文件中，如下所示：

    ``` yaml
          volumes:
          # mount in tmp so we can safely use from-scratch images and/or read-only containers
          - name: tmp-dir
            emptyDir: {}
          hostNetwork: true                          #Add this line to enable hostnetwork mode
          containers:
          - name: metrics-server
            image: metrics-server-kubeedge:latest    #Make sure that the REPOSITORY and TAG are correct
            # Modified args to include --kubelet-insecure-tls for Docker Desktop (don't use this flag with a real k8s cluster!!)
            imagePullPolicy: Never                   #Make sure that the deployment uses the image you built up
            args:
              - --cert-dir=/tmp
              - --secure-port=4443
              - --v=2
              - --kubelet-insecure-tls
              - --kubelet-preferred-address-types=InternalDNS,InternalIP,ExternalIP,Hostname
              - --kubelet-use-node-status-port       #Enable the feature of --kubelet-use-node-status-port for Metrics-server
            ports:
            - name: main-port
              containerPort: 4443
              protocol: TCP
    ```

## 重置KubeEdge master节点和工作节点

### Master

`keadm reset`将停止 `cloudcore` 并从 Kubernetes master 中删除与KubeEdge相关的资源，如 `kubeedge` 命名空间。它不会卸载/删除任何先决条件。

它为用户提供了一个标志，以指定kubeconfig路径，默认路径为 `/root/.kube/config` 。

例子：

```shell
 # keadm reset --kube-config=$HOME/.kube/config
```

### 节点

`keadm reset` 将停止 `edgecore` ，并且不会卸载/删除任何先决条件。
