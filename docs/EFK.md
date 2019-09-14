# EFK插件部署----后端使用NFS做存储持久化

## 1.搭建NFS共享存储服务器
```
[root@nfs ~]# mkdir -p /data/k8s
[root@nfs ~]# yum install nfs-utils -y
[root@nfs ~]# vim /etc/exports
/data/k8s/ 192.168.200.0/24(rw,no_root_squash)
[root@nfs ~]# systemctl restart rpcbind && systemctl enable rpcbind
[root@nfs ~]# systemctl start nfs-server && systemctl enable nfs-server
[root@nfs ~]# showmount -e
```

## 2.搭建nfs-StorageClass后端存储

```
[root@linux-node1 ~]# cd /srv/addons/EFK/nfs
[root@linux-node1 nfs]# kubectl apply -f .
storageclass.storage.k8s.io/course-nfs-storage created
serviceaccount/nfs-client-provisioner created
clusterrole.rbac.authorization.k8s.io/nfs-client-provisioner-runner created
clusterrolebinding.rbac.authorization.k8s.io/run-nfs-client-provisioner created
deployment.extensions/nfs-client-provisioner created
[root@linux-node1 nfs]# kubectl get sc
NAME                 PROVISIONER      AGE
course-nfs-storage   fuseim.pri/ifs   99s
[root@linux-node1 nfs]# kubectl get sa
NAME                     SECRETS   AGE
default                  1         94d
nfs-client-provisioner   1         2m52s
[root@linux-node1 nfs]# kubectl get pod |grep nfs-client-provisioner
nfs-client-provisioner-644658bd67-jtzpx               1/1     Running   0          2m1s
```

## 3.部署EFK

```
[root@linux-node1 ~]# kubectl apply -f /srv/addons/EFK/elasticsearch/elasticsearch-statefulset.yaml
statefulset.apps/es-cluster created
[root@linux-node1 ~]# kubectl apply -f /srv/addons/EFK/elasticsearch/elasticsearch-svc.yaml
service/elasticsearch created
[root@linux-node1 ~]# kubectl get pod -n logging |grep es
NAME           READY   STATUS    RESTARTS   AGE
es-cluster-0   1/1     Running   0          53s
es-cluster-1   1/1     Running   0          41s
es-cluster-2   1/1     Running   0          29s

[root@linux-node1 ~]# kubectl get pv,pvc -n logging
NAME                                                        CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS         REASON   AGE
persistentvolume/pvc-a2480a6b-d696-11e9-869b-00505682addb   10Gi       RWO            Delete           Bound    logging/data-es-cluster-0      course-nfs-storage            5m55s
persistentvolume/pvc-a248e6ae-d696-11e9-869b-00505682addb   10Gi       RWO            Delete           Bound    logging/plugins-es-cluster-0   course-nfs-storage            5m56s
persistentvolume/pvc-a98f34ea-d696-11e9-869b-00505682addb   10Gi       RWO            Delete           Bound    logging/data-es-cluster-1      course-nfs-storage            5m43s
persistentvolume/pvc-a9900e36-d696-11e9-869b-00505682addb   10Gi       RWO            Delete           Bound    logging/plugins-es-cluster-1   course-nfs-storage            5m43s
persistentvolume/pvc-b0b3bd23-d696-11e9-869b-00505682addb   10Gi       RWO            Delete           Bound    logging/data-es-cluster-2      course-nfs-storage            5m32s
persistentvolume/pvc-b0b4be54-d696-11e9-869b-00505682addb   10Gi       RWO            Delete           Bound    logging/plugins-es-cluster-2   course-nfs-storage            5m31s

NAME                                         STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS         AGE
persistentvolumeclaim/data-es-cluster-0      Bound    pvc-a2480a6b-d696-11e9-869b-00505682addb   10Gi       RWO            course-nfs-storage   5m56s
persistentvolumeclaim/data-es-cluster-1      Bound    pvc-a98f34ea-d696-11e9-869b-00505682addb   10Gi       RWO            course-nfs-storage   5m44s
persistentvolumeclaim/data-es-cluster-2      Bound    pvc-b0b3bd23-d696-11e9-869b-00505682addb   10Gi       RWO            course-nfs-storage   5m32s
persistentvolumeclaim/plugins-es-cluster-0   Bound    pvc-a248e6ae-d696-11e9-869b-00505682addb   10Gi       RWO            course-nfs-storage   5m56s
persistentvolumeclaim/plugins-es-cluster-1   Bound    pvc-a9900e36-d696-11e9-869b-00505682addb   10Gi       RWO            course-nfs-storage   5m44s
persistentvolumeclaim/plugins-es-cluster-2   Bound    pvc-b0b4be54-d696-11e9-869b-00505682addb   10Gi       RWO            course-nfs-storage   5m32s


[root@linux-node1 ~]# kubectl apply -f /srv/addons/EFK/kibana/kibana.yaml
service/kibana created
deployment.apps/kibana created

[root@linux-node1 ~]# kubectl get pod -n logging |grep kibana
kibana-779fb4cb79-pfl4n   1/1     Running   0          49s

[root@linux-node1 ~]# kubectl apply -f /srv/addons/EFK/fluentd/fluentd-daemonset.yaml
serviceaccount/fluentd-es created
clusterrole.rbac.authorization.k8s.io/fluentd-es created
clusterrolebinding.rbac.authorization.k8s.io/fluentd-es created
daemonset.apps/fluentd-es created

[root@linux-node1 ~]# kubectl apply -f /srv/addons/EFK/fluentd/fluentd-configmap.yaml
configmap/fluentd-config created

[root@linux-node1 ~]# kubectl get pod -n logging |grep fluentd
fluentd-es-45thz          1/1     Running   0          67s
fluentd-es-4qcb7          1/1     Running   0          67s
fluentd-es-7852g          1/1     Running   0          67s
fluentd-es-cmsgf          1/1     Running   0          67s
```

## 4.查看EFK的pod运行情况
```
[root@k8s-master01 ~]# kubectl get pod -n logging
NAME                      READY   STATUS    RESTARTS   AGE
es-cluster-0              1/1     Running   0          7m5s
es-cluster-1              1/1     Running   0          6m53s
es-cluster-2              1/1     Running   0          6m41s
fluentd-es-45thz          1/1     Running   0          3m8s
fluentd-es-4qcb7          1/1     Running   0          3m8s
fluentd-es-7852g          1/1     Running   0          3m8s
fluentd-es-cmsgf          1/1     Running   0          3m8s
kibana-779fb4cb79-pfl4n   1/1     Running   0          5m20s
```
## 5.打开kibana管理界面
```
[root@linux-node1 ~]# kubectl get svc -n logging |grep kibana
kibana          NodePort    10.93.2.26   <none>        5601:37278/TCP      6m6s

在浏览器输入http://nodeip:37278   索引为logstash-*
```
## 6. 日志收集截图

![EFK](https://github.com/sky-daiji/salt-k8s-ha-v2/blob/master/picture/kibana.png)

## 7. 根据具体日志收集场景，调整`elasticsearch-statefulset.yaml`此文件里面的上下限的资源参数
