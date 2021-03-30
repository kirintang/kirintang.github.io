---
title: Kubernetes中进行简单应用部署
date: 2021-03-12 15:13:48
tags:
 - k8s
 - kubernetes
---

掌握了```k8s```的部署启动，还需要知道如何在```k8s```中部署相应的应用服务，所谓实践检验真理，在```k8s```中部署一个简单的```nginx```服务来上手。

<!-- more -->

#### 一，创建```Nginx Deployment```

- ```nginx-deployment.yaml```文件编写

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.18.0
        ports:
        - containerPort: 80
```

- 应用```nginx-deployment.yaml```

```bash
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created
```

- 查看```Pod```运行状况

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/nginx-deployment-status.png)

- 查看```Dashboard```

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/nginx-deployment-dashboard.png)

#### 二，创建部署```Nginx Service```

- ```nginx-service.yaml```文件编写

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - port: 80
    targetPort: 80
    nodePort: 32500
```

- 应用配置文件

```bash
$ kubectl apply -f nginx-service.yaml
```

- 查看服务运行状况

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/nginx-service-status.png)

- 我们通过``http://localhost:32500``就可访问应用

![](https://raw.githubusercontent.com/chunlintang/imgLib/master/nginx-page.png)

至此，我们的nginx服务就部署成功了！

