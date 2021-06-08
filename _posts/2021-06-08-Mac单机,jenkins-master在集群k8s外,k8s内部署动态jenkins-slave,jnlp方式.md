---
layout:     post
title:      "Mac单机,jenkins-master在集群k8s外,k8s内部署动态jenkins-slave,jnlp方式"
subtitle:   " \"踩坑+吐血详细总结\""
date:       2021-06-08 17:30:00
author:     "WS"
header-img: "img/jenkins-0.jpg"
catalog:    true
tags:
    - Mac
    - Jenkins
    - Devops
    - K8s
---



## 1.准备工作

> 1. 安装kubernetes的kubectl和minikute, baidu很多, easy(不再赘述, 自行查找)
>
> 2. 启动minikube, 直接启动由于国内网络问题一直失败, 加上镜像仓库地址
>
>    ```shell
>    minikube start  image-mirror-country='cn' --registry-mirror=https://registry.docker-cn.com --memory=4096  --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
>    ```
>
> 3. 新建k8s namespace, 用于逻辑隔离整套elk服务(可不建, 走default namespace)
>
>    ```shell
>    kubectl create namespace elkspace
>    ```



## 2. k8s集群内操作:

#### 2.1 设置rbac, pv, pvc信息, 通过yaml文件启动, 先本地路径新建文件分别如下:

> - rbac.yaml : 定义供jenkins-master链接使用的ServiceAccount信息, 以及分配相应的角色权限, 其中Service Account Name是jenkins, Namespace是devops, 权限是cluser-admin, 这些信息也都可以使用kubectl命令一步步新建, 使用yaml文件可一次执行更加方便可视

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: jenkins
  namespace: devops
---
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: jenkins
  namespace: devops
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/exec"]
  verbs: ["create","delete","get","list","patch","update","watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get","list","watch"]
- apiGroups: [""]
  resources: ["secrets"]
  verbs: ["get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: jenkins
  namespace: devops
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: jenkins
    namespace: devops
```

---



> - jenkins-pv.yaml : 新建Persistent Volume信息, pv, pvc, 以及nfs, 其实用来集群内pods同步传输文件, 也就是常说的数据持久化, 主要是因为k8s内部工作的很多pods, 有一定生命周期, 也有一定损坏的风险, 故为了解决各个pods中的数据如何在运行时同步传输到指定的地方(文件服务器, 此处为nfs), 需要这些配套使用, 当然, 如果只是简单玩玩动态jenkins-slave, 此处可不必深究, 可配可不配, 我本地当时配了

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: jenkins-pv
  namespace: devops
spec:
  storageClassName: pv1
  persistentVolumeReclaimPolicy: Recycle
  capacity:
    storage: 15Gi
  accessModes:
    - ReadWriteOnce
  nfs:
    server: 10.68.128.26
    path: /Users/grahamliu/App/nfs/jenkins
```

---



> -  jenkins-pvc.yaml : 配合jenkins-pv.yaml使用

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: jenkins-pvc
  namespace: devops
spec:
  storageClassName: pv1
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
```

#### 2.2 使用kubectl命令分别执行以上yaml文件, 必须不能报错, 报错请排查

```shell
kubectl create -f rbac.yaml -n devops
kubectl create -f jenkins-pv.yaml -n devops
kubectl create -f jenkins-pvc.yaml -n devops
```

#### 2.3 使用kubectl命令查看pv-pvc绑定状态, STATUS都为Bound说明成功

```shell
pro-2:~ grahamliu$ kubectl get pv -n devops
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                STORAGECLASS   REASON   AGE
jenkins-pv   15Gi       RWO            Retain           Bound    devops/jenkins-pvc   pv1                     23h
```

```shell
pro-2:~ grahamliu$ kubectl get pvc -n devops
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
jenkins-pvc   Bound    jenkins-pv   15Gi       RWO            pv1            23h
```



## 3. Jenkins-master配置, 主要步骤是配置链接k8s集群信息:

> 1. 我是mac本地用docker部署jenkins服务, docker pull 需要的镜像部署, baidu很多, easy
>
> 2. Jenkins安装Kubernetes插件, 可能安装完成后需要重启jenkins才可用
>
> 3. 插件安装成功后 :
>
>    > Jenkins进入Dashboard->系统管理->系统配置->页面最下面显示‘Cloud’,The cloud configuration has moved to a separate configuration page.点击进入->配置集群->kubernetes
>
> 4. 开始详细配置, 我的配置如下:

![javascript](/img/jenkins-1.png)

> 4.1 kubernetes地址获取:

```shell
pro-2:~ grahamliu$ kubectl cluster-info
Kubernetes master is running at https://192.168.64.9:8443
KubeDNS is running at https://192.168.64.9:8443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy
```

---

> 4.2 kubernetes命名空间 : 以上在k8s配置部分的rbac中的配置, 我配置为devops

> 4.3 凭据 : rbac中在devops下的service account所拥有的token, 通过两步命令获取, 获取后添加jenkins凭据

```shell
pro-2:~ grahamliu$ kubectl describe sa jenkins -n devops
Name:                jenkins
Namespace:           devops
Labels:              <none>
Annotations:         <none>
Image pull secrets:  <none>
Mountable secrets:   jenkins-token-6fqd9
Tokens:              jenkins-token-6fqd9
Events:              <none>
pro-2:~ grahamliu$ kubectl describe secrets jenkins-token-6fqd9 -n devops
Name:         jenkins-token-6fqd9
Namespace:    devops
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: jenkins
              kubernetes.io/service-account.uid: b4c3b24a-54ea-4d30-9f24-f4df6df2002b

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1066 bytes
namespace:  6 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6Ikt0eW9BWmZtc3prX0NEcER2b1NmVmcyc1hqR2tleVlWeXBabXZoUkdoV2sifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZXZvcHMiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlY3JldC5uYW1lIjoiamVua2lucy10b2tlbi02ZnFkOSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJqZW5raW5zIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiYjRjM2IyNGEtNTRlYS00ZDMwLTlmMjQtZjRkZjZkZjIwMDJiIiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRldm9wczpqZW5raW5zIn0.A64F7RxvnLc1Oj9SvMftLdaEKrZLqpbICeAmB49uzGqFHyvnDZYvLpaxguFvpsX6x-jo0pm4frUGE8bDKlskBYwl04nHpyKIKoMc6e4t23BuJshFR1kgd2CVP98K6GASJ-5wnMM3KSQ4DjDDcLGZC0VIvtie2B-jrOsBpHuR4_KgnF3Wpyex9JOHMqxYAYA2pjwV2731GsmSk3EtzqrwE6t2qobm3Wq3cwfIy1CC0myj_ObrTlG7TUP_s4Ui7LSWGO3ae2goAkBcSGUqEVpcqSYM4KGA7dJ9M68J4ItB6s1uaHuxlVmKG5iv_CRAHruuVxLvsPpsj7CqWeWq1TlNmQ
```

![javascript](/img/jenkins-2.png)

> 5.以上配置完成后, 点击“连接测试”, 应该显示Connected to Kubernetes vX.XX.X信息, 如果报错, 请按报错信息和以上配置步骤排查

> 6.Jenkins 地址 : 实际配置多少写多少, docker中起的jenkins服务, 查看本机IP地址和开放端口配置, 比较容易配错, 请多尝试
>
> ⚠️注 : Jenkins-master如果在k8s集群内, 需要配置k8s内给jenkins分配的ip地址, 此ip和电脑本机ip不同

> 7.Jenkins通道 : ip同Jenkins 地址的ip, 不带http头, 端口配置对, 确定到底是5000还是50000? (我当时配错, 排查了很久, 其他都没问题, 这一个配置不对同样带不起k8s配置)
>
> > 注⚠️ : 1. 首先确定本地jenkins服务是否开启5000端口映射? 如果开启, 下一步
> >
> > 2.Jenkins进入Dashboard->系统管理->全局安全配置->代理, 确定此处配置和jenkins服务开启端口一致!!!!
> >
> > 3.确定ip配置一致, 端口一致, 此配置表面算成功了

![javascript](/img/jenkins-3.png)

> 8.添加Pod Templete配置, 如下:

![javascript](/img/jenkins-4.png)

![javascript](/img/jenkins-5.png)

以上Jenkins配置K8s结束



## 4. 新建pipline Job测试后build:

```groovy
pipeline {
  agent {
      label 'jenkins-slave-k8s'
  }
  stages {
      stage('test') {
          steps {
              script {
                  println "test"
              }
          }
      }
  }
}
```



## 5. 正常的话应该build success😄🎉

![javascript](/img/jenkins-6.png)



注意🔞:

> 查询网上信息, 在jenkins-slave构建中, 有可能在pull image时无法成功拉取镜像, 或者想拉取私人image, 可以尝试配置个人docker仓库的密钥, 然后通过个人仓库+密钥拉取可baidu查询“kubernetes配置secret拉取私仓镜像“获取详细配置信息

![javascript](/img/jenkins-7.png)
