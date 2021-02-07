# k8s-tutorial

Kubernetes关联内容（从Docker Desktop到集群构筑，EFK日志收集，Promethues,Grafana监控，Jenkins CI/CD)

主要参考了<https://www.qikqiak.com/k8s-book/>的内容。替换了一些docker image的版本，做了些小的修改。

## Docker Desktop安装

[Docker官网](https://www.docker.com/)直接下载就好。Windows版需要虚拟机的支持，Win10 20H4版本自带Hyper-V，直接可以用。安装完Docker就可以用了。
可以在CMD或者PowerShell下面执行下命令看看确认安装完成。

```bash
docker version
```

## 在Docker Desktop上部署K8S

在Docker的设定画面把Kubernetes的项目勾上，就完成了。
。。。
然而国内不翻墙那些image都pull不下来。建议参考下面的地址
<https://github.com/AliyunContainerService/k8s-for-docker-desktop>
把项目Clone下来，然后执行

```bash
.\load_images.ps1
```

还有几个注意事项，最好还是仔细的按着步骤来

+ 其实，打开sh或者ps1文件可以看到，sh或者ps1只不过从aliyun把镜像下载下来，然后又重命名了。有耐心自己去docker pull | docker tag 也是一样的。

比如对应images.properties里的第一项

```bash
k8s.gcr.io/pause:3.2=registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
```

其实就是

```bash
docker pull registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2
docker tag registry.cn-hangzhou.aliyuncs.com/google_containers/pause:3.2 k8s.gcr.io/pause:3.2
docker rmi registry.cn-hangzhou.aliyuncs.com/google
```

可以继续跟着<https://github.com/AliyunContainerService/k8s-for-docker-desktop>的说明启动并验证Kubernetes的集群状态

## 安装kubernetes-dashboard

在PowerShell中执行（首先要进入到保存着kubernetes-dashboard.yaml文件的文件夹下）

```bash
kubectl apply -f .\kubernetes-dashboard.yaml
```

这个文件先创建了一个Namespace,然后在这个namespace下创建了1个ServiceAccount,2个service，2个Deployment都是replicas:1，所以会各自生成一个pod。
还有一堆config和权限相关的设定文件。
文件的写法和docker-compose有点像，但不仅仅限于ENV,VOLUME,COPY,PORT等等，还有很多和编排，权限相关的内容。
通过下面的命令，可以返回sa/kubernetes-dashboard账号的token，用于登陆dashboard。

```bash
kubectl -n kubernetes-dashboard get secret $(kubectl -n kubernetes-dashboard get sa/kubernetes-dashboard -o jsonpath="{.secrets[0].name}") -o go-template="{{.data.token | base64decode}}"
```

其他步骤还是参考上面的k8s-for-docker-desktop的说明。
在控制台执行
`kubectl proxy`
后访问
`http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/`
