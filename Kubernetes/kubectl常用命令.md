# kubectl常用命令



**pod**

获取pod详细信息

kubectl get pods -o wide

进入pod

kubectl exec -ti pod/train-faq-net-test-t48zq -n default -- /bin/sh

**label**

\#查看当前node $ kubectl get node -o wide

\#给k8s-node1 k8s-node2打上标签websvr $ kubectl label nodes k8s-node1 k8s-node2 type=websvr

\#查看type=websvr标签的node $ kubectl get node -l type=websvr

\#以下附带标签的其他操作：

\#修改标签

$ kubectl label nodes k8s-node1 k8s-node2 type=webtest --overwrite

\#查看node标签

$ kubectl get nodes k8s-node1 k8s-node2 --show-labels

\#删除标签

$ kubectl label nodes k8s-node1 k8s-node2 type-

**job**

创建job

kubectl apply -f nvidia-test.yml

**namespace**

获取所有namespace

kubectl get namespaces

**token**

join节点命令生成

kubeadm token create --print-join-command

**dashboard**

获取dashboard管理员角色token

kubectl get secret -n kube-system

获取token

kubectl describe secret dashboard-admin-token-jc8t5 -n kube-system