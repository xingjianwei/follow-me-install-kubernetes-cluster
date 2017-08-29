<!-- toc -->

tags: nextcloud

# 使用pods部署 nextcloud

## 创建存储池

如果已经在nexus部署时执行过，则直接转到[创建 PVC](#jump1)：
```
# 创建存储池
rados mkpool kube
```
k8s会直接使用kube pool，以下命令无需执行：
```
# 创建 image (10GB)
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


## <span id="jump1">创]建 PVC</span>

`kubectl create -f ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/cephrbd/nextcloud.sc.pvc.yml`

## 创建 nextcloud
使用nextcloud目录下文件进行安装：
`kubectl create -f ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/nextcloud/`

进入容器：
```
mkdir /var/lib/nextcloud/data
chown -R www-data:www-data /var/lib/nextcloud/data
```
根据上面的地址打开用户界面，进行初始设置创建用户和数据库。
```
nextcloud
password
nextcloud_db
nextcloud-postgresql:5432
```
有新service增加时，修改ingress.yaml文件后可以使用`kubectl replace -f  ~/github/xingjianwei/follow-me-install-kubernetes-cluster/manifests/traefik-ingress/ingress.yaml`来更新。