# Kubernetes dashboard搭建

## 1.下载yaml文件

下载kubernetes-dashboard.yaml文件

```
wget https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml
```



## 2、修改kubernetes-dashboard.yaml文件

```

--- Dashboard Service

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  type: NodePort     // 添加类型
  ports:
    - port: 443
      targetPort: 8443
      nodePort: 30001     // 访问端口
  selector:
    k8s-app: kubernetes-dashboard


--- Dashboard Deployment

kind: Deployment
apiVersion: apps/v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kubernetes-dashboard
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
        - name: kubernetes-dashboard
            // 修改下面镜像
          image: registry.cn-hangzhou.aliyuncs.com/kube_containers/kubernetes-dashboard-amd64
          imagePullPolicy: Always
          ports:
            - containerPort: 8443
              protocol: TCP
          args:
            - --auto-generate-certificates
            - --namespace=kubernetes-dashboard
```



## 3.部署

```
kubectl apply -f kubernetes-dashboard.yaml
```

## 4.查看运行

```
kubectl get all -n kubernetes-dashboard
```



## 5.创建kubernetes-dashboard管理员角色

```
vi k8s-admin.yaml
# 添加如下：
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kube-system
---
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: dashboard-admin
subjects:
  - kind: ServiceAccount
    name: dashboard-admin
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
```



## 6、加载管理员角色

```
kubectl create -f k8s-admin.yaml
```

## 7、获取dashboard管理员角色token

```
kubectl get secret -n kube-system

获取token

kubectl describe secret dashboard-admin-token-jc8t5 -n kube-system
```

