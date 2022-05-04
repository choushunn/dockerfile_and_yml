## 安装 K8S
1. 配置主机名
```
hostnamectl set-hostname master01
vi /etc/hostname
hostnamectl set-hostname node01
```

2. 配置hosts
```
cat >> /etc/hosts << EOF
192.168.2.221    master01
192.168.2.222    node01
192.168.2.223    node02
EOF
```
3. 关闭 swap
```
swapoff -a && sed -i.bak '/swap/s/^/#/' /etc/fstab
```

4.关闭防火墙
```
firewall-cmd --state        #查看防火墙状态
systemctl stop firewalld.service        #停止firewall
systemctl disable firewalld.service     #禁止firewall开机启动
```

5.关闭selinux
```
getenforce  #查看selinux状态
setenforce 0    #临时关闭selinux
sed -i 's/^ *SELINUX=enforcing/SELINUX=disabled/g' /etc/selinux/config  #永久关闭（需重启系统）
```
6.安装依赖
```
yum install -y yum-utils  device-mapper-persistent-data   lvm2 net-tools bash-completion
source /etc/profile.d/bash_completion.sh
```
7.安装 docker

8.安装 k8s
```
cat > /etc/yum.repos.d/kubernetes.repo << EOF 
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

yum install -y kubelet kubeadm kubectl

systemctl start kubelet
systemctl enable kubelet 

yum remove -y kubelet kubeadm kubectl
9.k8s 镜像下载 image.sh
```sh
#!/bin/bash
url=registry.cn-hangzhou.aliyuncs.com/google_containers
version=v1.21.0

images=(`kubeadm config images list --kubernetes-version=$version|awk -F '/' '{print $2}'`)

for imagename in ${images[@]} ; do
    if [ $imagename != 'coredns' ]
    then
      docker pull $url/$imagename
	  docker tag $url/$imagename k8s.gcr.io/$imagename
	  docker rmi -f $url/$imagename
    else
      docker pull coredns/coredns:1.8.0
      docker tag coredns/coredns:1.8.0 k8s.gcr.io/coredns/coredns:v1.8.0
      docker rmi coredns/coredns:1.8.0
    fi
done
```
chmod u+x image.sh

10.k8s master 初始化
```
kubeadm init --apiserver-advertise-address 0.0.0.0 --pod-network-cidr=10.0.0.0/16
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
11.加入 Node 
```
kubeadm join 192.168.2.220:6443 --token bp7bzr.wj4h4asjdzvzolcp \
	--discovery-token-ca-cert-hash sha256:e7150fa8b5b5e3bf05aac5947fa15699bc22c972b1530e228d4801141e7b4b7f 
```

12.dashboard部署
```
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.0.0/aio/deploy/recommended.yaml

cat >> recommended.yaml << EOF
---
# -------- dashboard-admin -------- #
apiVersion: v1
kind: ServiceAccount
metadata:
  name: dashboard-admin
  namespace: kube-system
  
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: dashboard-admin
subjects:
- kind: ServiceAccount
  name: dashboard-admin
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
EOF 


kubectl get secret -n kubernetes-dashboard
kubectl  describe  secret  kubernetes-dashboard-token-ngcmg  -n  kubernetes-dashboard
kubectl get nodes
kubectl describe deployments

cat > dashboard-adminuser.yaml << EOF
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard

---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard  
EOF

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```
