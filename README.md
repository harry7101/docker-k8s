Docker Desktop for Mac/Windows 开启 Kubernetes
中文 | English

说明:

需安装 Docker Desktop 的 Mac 或者 Windows 版本，如果没有请下载下载 Docker CE最新版本
当前 master 分支已经在 Docker for Mac/Windows 4.1.0 (包含 Docker CE 20.10.8 和 Kubernetes 1.21.5) 版本测试通过
如果需要测试其他版本，请查看 Docker Desktop版本，Docker -> About Docker Desktop about
如Kubernetes版本为 v1.21.5, 请使用下面命令切换 v1.21.5 分支 git checkout v1.21.5

注：如果发现K8s版本与您的环境不一致，可以修改images.properties文件指明所需镜像版本，欢迎Pull Request。

开启 Kubernetes
为 Docker daemon 配置镜像加速，参考阿里云镜像服务 或中科大镜像加速地址https://docker.mirrors.ustc.edu.cn

mirror

可选操作: 为 Kubernetes 配置 CPU 和 内存资源，建议分配 4GB 或更多内存。

resource

从阿里云镜像服务下载 Kubernetes 所需要的镜像

在 Mac 上执行如下脚本

./load_images.sh
在Windows上，使用 PowerShell

 .\load_images.ps1
说明:

如果因为安全策略无法执行 PowerShell 脚本，请在 “以管理员身份运行” 的 PowerShell 中执行 Set-ExecutionPolicy RemoteSigned 命令。
如果需要，可以通过修改 images.properties 文件自行加载你自己需要的镜像
开启 Kubernetes，并等待 Kubernetes 开始运行 k8s

TIPS：

在Mac上:

如果在Kubernetes部署的过程中出现问题，可以通过docker desktop应用日志获得实时日志信息：

pred='process matches ".*(ocker|vpnkit).*"
  || (process in {"taskgated-helper", "launchservicesd", "kernel"} && eventMessage contains[c] "docker")'
/usr/bin/log stream --style syslog --level=debug --color=always --predicate "$pred"
在Windows上:

如果在Kubernetes部署的过程中出现问题，可以在 C:\ProgramData\DockerDesktop下的service.txt 查看Docker日志, 在 C:\Users\yourUserName\AppData\Local\Docker下的log.txt 查看Kubernetes日志

问题诊断：

如果看到 Kubernetes一直在启动状态，请参考

Issue 3769(comment) 或 Issue 3649(comment)
在macOS上面，执行 rm -fr '~/Library/Group\ Containers/group.com.docker/pki'
在Windows上面删除 'C:\ProgramData\DockerDesktop\pki' 目录 和 'C:\Users\yourUserName\AppData\Local\Docker\pki' 目录
Issue 1962(comment)
配置 Kubernetes
可选操作: 切换Kubernetes运行上下文至 docker-desktop (之前版本的 context 为 docker-for-desktop)

kubectl config use-context docker-desktop
验证 Kubernetes 集群状态

kubectl cluster-info
kubectl get nodes
配置 Kubernetes 控制台
部署 Kubernetes dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.4/aio/deploy/recommended.yaml
或

kubectl create -f kubernetes-dashboard.yaml
检查 kubernetes-dashboard 应用状态

kubectl get pod -n kubernetes-dashboard
开启 API Server 访问代理

kubectl proxy
通过如下 URL 访问 Kubernetes dashboard

http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/

配置控制台访问令牌
对于Mac环境

TOKEN=$(kubectl -n kube-system describe secret default| awk '$1=="token:"{print $2}')
kubectl config set-credentials docker-for-desktop --token="${TOKEN}"
echo $TOKEN
对于Windows环境

$TOKEN=((kubectl -n kube-system describe secret default | Select-String "token:") -split " +")[1]
kubectl config set-credentials docker-for-desktop --token="${TOKEN}"
echo $TOKEN
登录dashboard的时候
resource

选择 令牌

输入上文控制台输出的内容

或者选择 Kubeconfig 文件,路径如下：

Mac: $HOME/.kube/config
Win: %UserProfile%\.kube\config
点击登陆，进入Kubernetes Dashboard

配置 Ingress
说明：如果测试 Istio，不需要安装 Ingress

安装 Ingress
源地址安装说明

- 若安装脚本无法安装，可以跳转到该地址查看最新操作
安装

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/cloud/deploy.yaml
验证

kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx
测试示例应用
部署测试应用，详情参见社区文章

kubectl create -f sample/apple.yaml
kubectl create -f sample/banana.yaml
kubectl create -f sample/ingress.yaml
测试示例应用

$ curl -kL http://localhost/apple
apple
$ curl -kL http://localhost/banana
banana
删除示例应用

kubectl delete -f sample/apple.yaml
kubectl delete -f sample/banana.yaml
kubectl delete -f sample/ingress.yaml
删除 Ingress
kubectl delete -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-0.32.0/deploy/static/provider/cloud/deploy.yaml
安装 Helm
可以根据文档安装 helm v3 https://helm.sh/docs/intro/install/ 在国内由于helm的cdn节点使用的是谷歌云所以可能访问不到，可以参考已存在的官方issue： https://github.com/helm/helm/issues/7028

在 Mac OS 上安装
通过 brew 安装
# Use homebrew on Mac
brew install helm

# Add helm repo
helm repo add stable http://mirror.azure.cn/kubernetes/charts/

# Update charts repo
helm repo update
在Windows上安装
如果在后续使用 helm 安装组件的过程中出现版本兼容问题，可以参考 通过二进制包安装 思路安装匹配的版本

# Use Chocolatey on Windows
# 注：安装的时候需要保证网络能够访问googleapis这个域名
choco install kubernetes-helm

# Change helm repo
helm repo add stable http://mirror.azure.cn/kubernetes/charts/

# Update charts repo
helm repo update
测试 Helm (可选)
安装 Wordpress

helm install wordpress stable/wordpress
查看 wordpress 发布状态

helm status wordpress
卸载 wordpress 发布

helm uninstall wordpress
配置 Istio
说明：Istio Ingress Gateway和Ingress缺省的端口冲突，请移除Ingress并进行下面测试

可以根据文档安装 Istio https://istio.io/docs/setup/getting-started/

下载 Istio 1.5.0
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.5.0 sh -
cd istio-1.5.0
export PATH=$PWD/bin:$PATH
在Windows上，您可以手工下载Istio安装包，或者把getLatestIstio.ps1拷贝到你希望下载 Istio 的目录，并执行 - 说明：根据社区提供的安装脚本修改而来

.\getLatestIstio.ps1
安装 Istio
istioctl manifest apply --set profile=demo
检查 Istio 状态
kubectl get pods -n istio-system
为 default 名空间开启自动 sidecar 注入
kubectl label namespace default istio-injection=enabled
kubectl get namespace -L istio-injection
安装 Book Info 示例
请参考 https://istio.io/docs/examples/bookinfo/

kubectl apply -f samples/bookinfo/platform/kube/bookinfo.yaml
查看示例应用资源

kubectl get svc,pod
确认示例应用在运行中

kubectl exec -it $(kubectl get pod -l app=ratings -o jsonpath='{.items[0].metadata.name}') -c ratings -- curl productpage:9080/productpage | grep -o "<title>.*</title>"
创建 Ingress Gateway

kubectl apply -f samples/bookinfo/networking/bookinfo-gateway.yaml
查看 Gateway 配置

kubectl get gateway
确认示例应用可以访问

export GATEWAY_URL=localhost:80
curl -s http://${GATEWAY_URL}/productpage | grep -o "<title>.*</title>"
可以通过浏览器访问

http://localhost/productpage

删除实例应用
samples/bookinfo/platform/kube/cleanup.sh
卸载 Istio
istioctl manifest generate --set profile=demo | kubectl delete -f -
