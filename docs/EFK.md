# EFK插件部署

## 1.创建logging命名空间
```
[root@linux-node1 ~]# kubectl create ns logging
```
## 2.给所有节点打上标签

```
[root@linux-node1 ~]# kubectl label nodes beta.kubernetes.io/fluentd-ds-ready=true --all
```

## 3.部署EFK

```
[root@linux-node1 ~]# kubectl apply -f /srv/addons/EFK/

```

## 4.查看EFK的pod运行情况
```
[root@linux-node11 ~]# kubectl get pod -n logging
NAME                              READY   STATUS    RESTARTS   AGE
elasticsearch-logging-0           1/1     Running   0          31m
elasticsearch-logging-1           1/1     Running   0          29m
elasticsearch-logging-2           1/1     Running   0          28m
fluentd-es-v2.2.0-64gsn           1/1     Running   0          12m
fluentd-es-v2.2.0-8d4db           1/1     Running   0          12m
fluentd-es-v2.2.0-8nlrb           1/1     Running   0          12m
fluentd-es-v2.2.0-hddwq           1/1     Running   0          12m
kibana-logging-58578d696f-95f87   1/1     Running   0          31m

```
## 5.打开kibana管理界面
```
[root@linux-node1 ~]# kubectl get svc -n logging
NAME                    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
elasticsearch-logging   ClusterIP   10.93.127.27   <none>        9200/TCP         37m
kibana-logging          NodePort    10.93.115.48   <none>        5601:39615/TCP   37m

在浏览器输入http://nodeip:39615   索引为fluentd-k8s-*
```
## 6. 日志收集截图

![EFK](https://github.com/sky-daiji/salt-k8s-ha-v2/blob/master/picture/kibana.png)
