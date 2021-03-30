---
title: Kubernetes快速搭建与启动
date: 2021-03-11 17:15:33
tags:
 - k8s
 - kubernetes
---

Kubernetes(以下简称k8s)已经成为当下最流行的一款容器编排工具，而且目前很多公司都在往容器化方向发展，调研最多的也是k8s，我们公司目前也在逐步向k8s靠拢，使用k8s开放的api做一些服务开发；因此学习掌握k8s技术也成为刻不容缓的事。

<!-- more -->

本文主要介绍在MaxOS环境下进行k8s的搭建和启动运行，有条件的可以购买云服务器在linux环境下进行搭建。

#### 一，安装

- 前提条件
  - 一台MacBook本，内存至少在8G以上，有足够的硬盘空
  - 安装有Docker，可以到[官网]([Download for
    Mac ](https://desktop.docker.com/mac/stable/Docker.dmg))下载安装

- Step1，查看Docker对应的k8s版本号，我目前Docker版本为3.1.0，对应的最新版本k8s为v1.19.3，注意安安装过Docker的记得要把Docker更新到最新版本

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/docker-about-3.1.0.png)

- Step2，启动```Docker```，打开```Docker```的```Preferences```界面，切换到```Kubernetes```选项，未安装之前```Enable Kubernetes```是未选中状态，这里先不要选中，由于国内网络原因，会始终停留在```kunernets is starting...```状态这里已经有开发者开发了对应的脚本[k8s-docker-desktop-for-mac](https://github.com/maguowei/k8s-docker-desktop-for-mac)直接使用。

- Step3，克隆下来之后，运行脚本```./load_images.sh```，等待相关镜像拉取完成后，将```Enable Kubernetes```选中后应用重启。等待```kubernetes```启动，然后检查 ```Docker Preferences``` 界面左下角的```Kubernetes``` 状态是否正常，状态为```kubernetes is running```就表示启动成功了。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/docker-k8s-running.png)

#### 二，安装Dashboard

安装完```k8s```后，我们需要安装一个可视化的管理界面```Dashboard```。执行命令:

```bash
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v0.44.0/deploy/static/provider/cloud/deploy.yaml
```

也可以到```Dashboard```[官网仓库](https://github.com/kubernetes/dashboard)找到最新稳定版本，找到对应的版本号替换上面的命令。

执行完上述命令后，就可以启动服务：

```bash
// 默认会启用8001端口号
$ kubectl proxy
```

或者，指定端口号，如果我们希望其他机器也可以访问，可以加上```--accept-hosts```

```bash
$ kubectl proxy --address=0.0.0.0 --port=8001 --accept-hosts='.*'
```

启动成功后，我们通过浏览器直接访问访问```http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/#/login```，会跳到如下图：

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/k8s-dashboard.png)

此时需要我们需要我们登陆，我们需要创建```ServiceAccount```生成对应的```Token```。

#### 三，创建ServiceAccount

在本地目录新建```k8s-dashboard-admin.yaml```文件，建立一个```ServiceAccount```的角色绑定关系，文件内容如下：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: k8s-dashboard-admin
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: k8s-dashboard-admin
subjects:
  - kind: ServiceAccount
    name: k8s-dashboard-admin
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```

然后通过如下命令获取管理员角色```Secret```:

```bash
$ kubectl get secrets -n kube-system | grep k8s-dashboard-admin | awk '{print $1}'
k8s-dashboard-admin-token-s7l4l
```

获取对应的```Token```:

```bash
$ kubectl describe secret k8s-dashboard-admin-token-s7l4l -n kube-system
Name:         k8s-dashboard-admin-token-s7l4l
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: k8s-dashboard-admin
              kubernetes.io/service-account.uid: bc85c3d1-450e-4357-9567-8b3c2019e88d

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  11 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkpoT1BqejZKMDlXMXczZTJRZ1k4OUFVVVJNZEx1RGc1YlV3YXJDeTlyeTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJkYXNoYm9hcmQtYWRtaW4tdG9rZW4tczdsNGwiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC5uYW1lIjoiZGFzaGJvYXJkLWFkbWluIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYmM4NWMzZDEtNDUwZS00MzU3LTk1NjctOGIzYzIwMTllODhkIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50Omt1YmUtc3lzdGVtOmRhc2hib2FyZC1hZG1pbiJ9.kx7ROQOB47_sM98v_WibH_k27lLm0dE1Cf35j0djAyVrNAPXvTuTGx65ss1Qnt7NxFDPDuUdyX9SbvOBlD2JqbThpqhU7XHvab1ZKQ4JBqHweuWb3YcAZHDWzu_p5cbc2bpN4fPjggRNsNZCZFZL2ylB_kXKr3mnCny4CILZEvgLRHdskjUYWpzOPz2SeligJHtCIjMwUdtWA7XYp-13-qnFyJmXtM29vjVmVUIV2ROXbA6rBRXszKC07lZxuBwkDLox83-xBepovJxJfdKkY-XqVb9kdSPhANwDJumkiyAsUGUI78BhG7_-YtA7rQjVXvkGaRPNoGnPK8tO8771nQ
```

然后我们```Copy```对应的```token```值到我们的界面表单，点击登录，就可以进入到我们的``k8s dashboard```界面。

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/k8s-dashboard-workloads.png)

至此，我们的k8s本地安装和启动就完成了，有兴趣和条件的可以使用云服务器安装启动。