# k8-cluster-Hard-way-



                                                ***Setting up a multi-node Kubernetes cluster***
                                                   Kubernetes Multi-Node CLuster on AWS


1)  launch aws instance min 2GB RAM

2)  install dokcer and enable it

3) install kubeadm tool

4)  disable the selinux vi /etc/selinux/config  -->  SELINUX=permissive

5) sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

6)  docker cgroup driver 

7)  disable the swap or comment it  --> # vi /etc/fstab

8)  systemctl enable docker && systemctl start docker  and  systemctl enable kubelet && systemctl start kubelet

9) set the hostname # hostnamectl set-hostname master 

10) make the entrires in all the master , node01 ,node02   -->  /etc/hosts and check ping connectivity to each other 

11) #  kubeadm init --pod-network-cidr=10.10.0.1/16  -----> Setting up Kubernetes Control Plane on the master node

12)  # kubectl apply  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.ym
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

NOTE:Kubernetes is a software system that allows you to deploy and manage containerized apps. It abstracts away the underlying infrastructure to simplify development, deployment and management for both dev and ops teams. There are many benefits and components provided by the Kubernetes platform with many options about installing and setting up the Kubernetes cluster such as single-node, multiple-node and cloud-based deployments. In this article, we will focus on setting up multiple-node cluster on a laptop (such as 2.7 GHz Intel Core i5, 8G RAM should be sufficient for demonstration purpose)


1) create one instance on AWS with min 2GB RAM

[root@master ~]# free -m
              total        used        free      shared  buff/cache   available
Mem:           1794         770         268          16         755         881
Swap:             0           0           0
[root@master ~]#

________________________________________________________________________________________________________________________________________________________
                                             ****Installing Docker and Kubernetes****

2) install docker on aws instance

[root@master yum.repos.d]# ls
docker.repo  kubernetes.repo  redhat-rhui-beta.repo.disabled  redhat-rhui-client-config.repo  redhat-rhui.repo

**creating a docker.repo file to the /etc/yum.repos.d/ directory, shown in the following***

[root@master yum.repos.d]# cat docker.repo
[docker]

baseurl=https://download.docker.com/linux/centos/7/x86_64/stable/   
gpgcheck=0

[root@master yum.repos.d]# yum repolist
Repository 'docker' is missing name in configuration, using id.
repo id                                                    repo name
docker                                                     docker

# yum install docker-ce --nobest -y

# service docker start

# service docker enable 
_____________________________________________________________________________________________________________________________________________________

3) Installing kubeadm

***creating a kubernetes.repo file to the /etc/yum.repos.d/ directory, shown in the following***

cat <<EOF | sudo tee /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-\$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF


# ls

kubernetes.repo

3 ) vi /etc/selinux/config

SELINUX=permissive
 
and save the file :wq!


# sudo setenforce 0
# sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config

# sudo yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

# sudo systemctl enable --now kubelet
________________________________________________________________________________________________________________________________________________

4) docker cgroup driver 

# vi /etc/docker/daemon.json
 
{
  "storage-driver": "overlay2"
}

# systemctl restart docker
_______________________________________________________________________________________________________________________________________________


--------------------------------------------------------------------------------------------------------------------------------------------------

5) disable the swap or comment it

# vi /etc/fstab

# yum install iproute-tc

or And, disable swap with the following command:
# swapoff -a && sed -i '/ swap / s/^/#/' /etc/fstab

________________________________________________________________________________________________________________________________________________

6) After the installation, you need to enable the docker and kubelet services:
systemctl enable docker && systemctl start docker
systemctl enable kubelet && systemctl start kubelet

_________________________________________________________________________________________________________________________________________________

7) set the hostname # hostnamectl set-hostname master 
   and same on change the hostname node01 and node02

____________________________________________________________________________________________________________________________________________________

8) make the entrires in all the master , node01 ,node02

[root@master yum.repos.d]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6


172.31.42.224     master

172.31.44.202    node01

172.31.35.81     node02
[root@master yum.repos.d]#


updated cluster ips
172.31.33.142   master
172.31.46.2        node1
172.31.47.199   node2  

__________________________________________________________________________________________________________________________________________________

9) check ping connectivity to each other 

ping node01 , ping node02, 

____________________________________________________________________________________________________________________________________________________

                            *****Setting up Kubernetes Control Plane on the master node****

You can use the kubeadm tool to initialize the Kubernetes master and deploy all the Control Plane components, including etcd, the API server, Scheduler, kube-proxy and Control Manager. You will need to take a note of the last line of the output from running the following command.

10) # kubeadm init --pod-network-cidr=10.10.0.1/16

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.31.33.142:6443 --token khpjk5.mp9a8h74z27frnbk \
    --discovery-token-ca-cert-hash sha256:5d59fecb9c4abe7e4d173387930802866ce085665a2957826d3a5f9c198ed8b2


___________________________________________________________________________________________________________________________________________

[root@master ~]# kubectl get ns
NAME              STATUS   AGE
default           Active   40h
kube-node-lease   Active   40h
kube-public       Active   40h
kube-system       Active   40h
[root@master ~]#


kubectl get nods

____________________________________________________________________________________________________________________________________

finally 


 https://kubernetes.io/docs/concepts/cluster-administration/addons/ 
 
# kubectl apply  -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml 

[root@master ~]#  kubectl get pods --all-namespaces
NAMESPACE     NAME                             READY   STATUS        RESTARTS   AGE
default       graphtest-6755f69597-d582d       1/1     Running       0          12h
default       mypod1                           1/1     Terminating   0          39h
default       promtest-5569bb7967-dn58w        1/1     Running       0          12h
kube-system   coredns-66bff467f8-29w2h         1/1     Running       3          40h
kube-system   coredns-66bff467f8-jsmb6         1/1     Running       3          40h
kube-system   etcd-master                      1/1     Running       3          40h
kube-system   kube-apiserver-master            1/1     Running       3          40h
kube-system   kube-controller-manager-master   1/1     Running       3          40h
kube-system   kube-flannel-ds-amd64-b25qb      1/1     Running       1          39h
kube-system   kube-flannel-ds-amd64-b46n8      1/1     Running       0          39h
kube-system   kube-flannel-ds-amd64-x7vlj      1/1     Running       4          40h
kube-system   kube-proxy-2lmb5                 1/1     Running       1          39h
kube-system   kube-proxy-8z6l5                 1/1     Running       0          39h
kube-system   kube-proxy-ckxct                 1/1     Running       3          40h
kube-system   kube-scheduler-master            1/1     Running       3          40h
[root@master ~]#


# kubectl get pods     to see your multinode cluster


_____________________________________________________

__________________________________________________________________________________

                             ****Deploying Grafana & Prometheus Over Kubernetes (Persistent Volume)***

What is Prometheus?
Prometheus is an open-source systems monitoring and alerting toolkit originally built at SoundCloud. Since its inception in 2012, many companies and organizations have adopted Prometheus, and the project has a very active developer and user community. It is now a standalone open source project and maintained independently of any company. To emphasize this, and to clarify the project's governance structure, Prometheus joined the Cloud Native Computing Foundation in 2016 as the second hosted project, after Kubernetes.

Now we have many use cases where we want to deploy it over container engine and the best way to manage this is using Kubernetes. Here we face some issues with storage as prometheus requires to store the data. To solve this we will use the Persistent Volume Claim(PVC) feature of kubernetes, you can compare PVC as volume in docker but it do many more things for you and is more dynamic in nature.

So in this article we will see about how to deploy prometheus deploy over kubernetes and how we can make the data of it permanent using kubernetes yml files.	

________________________________________________________________________________________________________________________________________________

What is Grafana?
Grafana is open source visualization and analytics software. It allows you to query, visualize, alert on, and explore your metrics no matter where they are stored. In plain English, it provides you with tools to turn your time-series database (TSDB) data into beautiful graphs and visualizations.

________________________________________________________________________________________________________________________________________________

                        STEPS:--->  *****Deploying Grafana & Prometheus Over Kubernetes (Persistent Volume)***

[root@master project]# kubectl create deployment promtest1 --image=prom/prometheus:latest    
deployment.apps/promtest created  

[root@master project]# kubectl create deployment graphtest1  --image=grafana/grafana:latest
deployment.apps/graphtest created


[root@master project]# kubectl get pods
NAME                         READY   STATUS        RESTARTS   AGE
graphtest-6755f69597-d582d   1/1     Running       0          11s
mypod1                       1/1     Terminating   0          27h
promtest-5569bb7967-dn58w    1/1     Running       0          76s

--------------------------------------------------------------------------------------------------------------------------------------------------------
[root@master project]# kubectl expose deployment promtest1 --type=LoadBalancer --port=9090
service/promtest exposed

[root@master project]# kubectl expose deployment graphtest1 --type=LoadBalancer --port=3000
service/graphtest exposed

--------------------------------------------------------------------------------------------------------------------------------------------------------------
[root@master project]# kubectl get po -o wide
NAME                         READY   STATUS        RESTARTS   AGE    IP          NODE     NOMINATED NODE   READINESS GATES
graphtest-6755f69597-d582d   1/1     Running       0          118s   10.10.2.4   node02   <none>           <none>
mypod1                       1/1     Terminating   0          27h    10.10.1.2   node01   <none>           <none>
promtest-5569bb7967-dn58w    1/1     Running       0          3m3s   10.10.2.3   node02   <none>           <none>


[root@master project]# kubectl get all
NAME                             READY   STATUS        RESTARTS   AGE
pod/graphtest-6755f69597-d582d   1/1     Running       0          3m13s
pod/mypod1                       1/1     Terminating   0          27h
pod/promtest-5569bb7967-dn58w    1/1     Running       0          4m18s


NAME                 TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE
service/graphtest    LoadBalancer   10.104.83.142   <pending>     3000:32760/TCP   92s
service/kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP          27m
service/promtest     LoadBalancer   10.96.67.98     <pending>     9090:31023/TCP   111s

NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/graphtest   1/1     1            1           3m13s
deployment.apps/promtest    1/1     1            1           4m18s

NAME                                   DESIRED   CURRENT   READY   AGE
replicaset.apps/graphtest-6755f69597   1         1         1       3m13s
replicaset.apps/promtest-5569bb7967    1         1         1       4m18s
[root@master project]#



go and check node2 public -> http://13.233.244.88:31023/  --> promo

                             http://13.233.244.88:32760/   --> graphna


note : use # kubectl get po -o wide 
-----> check on which node graphna and prommo running then to access this use ip of node 01, node02 and expose port number 

e.g  http://13.233.244.88:31023/  --> promo   --> node 01
 
     http://13.233.244.98:32760/  -->  ----> node 02
)_______________________________________________________________________________

___________________________________________________________


imp link if our working node is not able to join the master then please refer this link

https://stackoverflow.com/questions/61352209/kubernetes-unable-to-join-a-remote-master-node

[root@master ~]# kubeadm token create --print-join-command

W0726 08:47:06.032367   32330 configset.go:202] WARNING: kubeadm cannot validate component configs for API groups [kubelet.config.k8s.io kubeproxy.config.k8s.io]
kubeadm join 172.31.42.224:6443 --token yf5ni4.5g1j9mmxfjcg46mg     --discovery-token-ca-cert-hash sha256:fdae9b4f87a7e702c08d9dcae6441f4acde2096b37cffa8db6d1c6e00ff6a260

[root@master ~]# kubeadm join 172.31.42.224:6443 --token yf5ni4.5g1j9mmxfjcg46mg     --discovery-token-ca-cert-hash sha256:fdae9b4f87a7e702c08d9dcae6441f4acde2096b37cffa8db6d1c6e00ff6a260

W0726 08:47:27.277253   32405 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
        [WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: [preflight] Some fatal errors occurred:
        [ERROR DirAvailable--etc-kubernetes-manifests]: /etc/kubernetes/manifests is not empty
        [ERROR FileAvailable--etc-kubernetes-kubelet.conf]: /etc/kubernetes/kubelet.conf already exists
        [ERROR Port-10250]: Port 10250 is in use
        [ERROR FileAvailable--etc-kubernetes-pki-ca.crt]: /etc/kubernetes/pki/ca.crt already exists
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher

[root@master ~]#
[root@master ~]# kubectl get no
NAME     STATUS                        ROLES    AGE   VERSION
master   Ready                         master   26d   v1.18.5
node01   NotReady,SchedulingDisabled   <none>   26d   v1.18.5
node02   Ready                         <none>   26d   v1.18.5
node03   Ready                         <none>   26s   v1.18.6
[root@master ~]#




