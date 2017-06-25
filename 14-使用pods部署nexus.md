<!-- toc -->

tags: nexus

# 使用pods部署nexus

## 创建块设备

```
# 创建存储池
rados mkpool kube
# 创建 image
rbd create kube --size 10240 -p kube
# 关闭不支持特性
rbd feature disable kube exclusive-lock, object-map, fast-diff, deep-flatten -p kube
# 映射(每个节点都要映射)
rbd map kube --name client.admin -p kube
# 格式化块设备(单节点即可)
mkfs.ext4 /dev/rbd0
```



## Use Ceph Authentication Secret

If Ceph authentication secret is provided, the secret should be first be base64 encoded, then encoded string is placed in a secret yaml. For example, getting Ceph user kube's base64 encoded secret can use the following command:

```
# grep key /etc/ceph/ceph.client.kube.keyring |awk '{printf "%s", $NF}'|base64
QVFBTWdYaFZ3QkNlRGhBQTlubFBhRnlmVVNhdEdENGRyRldEdlE9PQ==
```

`# kubectl create -f ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/cephrbd/ceph-secret.yaml`

`# kubectl create -f ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/cephrbd/ceph-secret-system.yaml`

测试样例：
`kubectl create -f test-rbd-with-secret.json`

## 创建 StorageClass
`kubectl create -f  ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/cephrbd/ceph-storage-kube_rbd.yml`

## 创建 PVC
`kubectl create -f ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/cephrbd/nexus.sc.pvc.yml`

## image修改
```
git clone https://github.com/sonatype/docker-nexus3.git
```

修改`Dockerfile`，去掉默认的`user nexus`，改由`root`用户启动。否则没有权限使用pvc写入文件。

## 创建 nexus
使用nexus-builder.yaml文件进行安装：
`kubectl create -f ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/nexus/nexus-builder.yaml`

安装完成后，查找对应端口进行访问：
`kubectl describe svc -n default nexus3 | grep 'NodePort:'`

根据上面的地址打开用户界面。点击右上角Sign in登陆，默认账号admin，密码admin123。

