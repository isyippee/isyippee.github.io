---
title: magnum部署k8s
tags: [openstack]
date: 2017-07-31 10:17:55
---

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#introduction "introduction")introduction

使用 openstack magnum 项目来创建稳定的 k8s 集群。
 <!-- more --> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u7248_u672C_u4FE1_u606F "版本信息")版本信息
<table> <thead> <tr> <th>name</th> <th>version</th> </tr> </thead> <tbody> <tr> <td>magnum</td> <td>ocata</td> </tr> <tr> <td>k8s</td> <td>v1.5.3</td> </tr> <tr> <td>kubectl</td> <td>v1.5.3</td> </tr> </tbody> </table> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u90E8_u7F72_u9700_u8981_u7684_u6240_u6709_u955C_u50CF "部署需要的所有镜像")部署需要的所有镜像

*   google_containers/kube-ui:v4
*   google_containers/pause-amd64:3.0
*   google_containers/hyperkube:v1.5.3
*   google_containers/pause:0.8.0
*   google_containers/kubedns-amd64:1.9
*   google_containers/dnsmasq-metrics-amd64:1.0
*   google_containers/kube-dnsmasq-amd64:1.4
*   google_containers/exechealthz-amd64:1.2
*   google_containers/defaultbackend:1.0
*   google_containers/nginx-ingress-controller:0.9.0-beta.11 

上述所有 docker 镜像打包为 `k8s-ocata.tar.gz` ，使用 local registry 时将其 push 即可

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u73AF_u5883_u8BBE_u7F6E "环境设置")环境设置

将 k8s 部署的需要的租户、网络都固定下来，约定如下

*   租户: admin
*   网络: kycloud-net
*   子网: magnum-subnet, 172.16.0.0/24 

创建并连接路由

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#heat__u914D_u7F6E "heat 配置")heat 配置
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># openstack role add --project admin --user admin heat_stack_owner</span>
</pre></td></tr></table></figure> 

上述操作保证 heat stack-create 操作成功

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u4E0A_u4F20_u955C_u50CF "上传镜像")上传镜像

镜像地址
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">https://fedorapeople.org/groups/magnum/fedora-atomic-ocata.qcow2</span>
</pre></td></tr></table></figure> 

转换为 raw 格式
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># qemu-img convert -f qcow2 -O raw fedora-atomic-ocata.qcow2 fedora-atomic-ocata.raw</span>
</pre></td></tr></table></figure> 

上传镜像
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># glance image-create --name fedora-atomic-ocata-raw --disk-format raw --container-format bare --file fedora-atomic-ocata.raw --os-distro fedora-atomic --progress</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u521B_u5EFA_u7F51_u7EDC "创建网络")创建网络

创建一个独立的网络及其子网用于 k8s 集群

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u914D_u7F6E_u672C_u5730_registry "配置本地 registry")配置本地 registry

使用 magnum 创建 k8s 需要从 gcr.io 拉取 docker images，可以使用两种办法解决

*   配置翻墙
*   使用 local registry 

我们采用使用 local registry 的方式实现。

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u90E8_u7F72 "部署")部署

这里使用 k8s 集群的网络创建一个虚拟机，并安装 local registry，具体的步骤是

安装 docker
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># curl -sSL https://get.docker.io | bash&#10;# curl -sSL https://get.daocloud.io/daotools/set_mirror.sh | sh -s http://db9b6d5c.m.daocloud.io</span>
</pre></td></tr></table></figure> 

部署 local registry
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># docker run -d -p 5000:5000 --restart=always --name registry \&#10; -v `pwd`/data:/var/lib/registry \&#10; registry:2</span>
</pre></td></tr></table></figure> 

注意需要打开安全组 5000 端口

将需要的 images 导入到本地
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># docker load -i k8s-ocata.tar.gz</span>
</pre></td></tr></table></figure> 

假设 local registry url 是 192.168.21.10:5000, 给这些镜像打上 tag
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># registry_url=192.168.21.10:5000&#10;# docker tag gcr.io/google_containers/kube-ui:v4 $registry_url/google_containers/kube-ui:v4&#10;# docker tag gcr.io/google_containers/pause-amd64:3.0 $registry_url/google_containers/pause-amd64:3.0&#10;# docker tag gcr.io/google-containers/hyperkube:v1.5.3 $registry_url/google_containers/hyperkube:v1.5.3&#10;# docker tag gcr.io/google_containers/pause:0.8.0 $registry_url/google_containers/pause:0.8.0</span>
</pre></td></tr></table></figure> 

编辑 /etc/docker/daemon.json，配置 registry client(这里的 client 和 registry 都在同个虚拟机中)
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">&#123; &#10; &#34;registry-mirrors&#34;: [&#34;http://db9b6d5c.m.daocloud.io&#34;], &#10; &#34;insecure-registries&#34;:[&#34;192.168.21.10:5000&#34;]&#10;&#125;</span>
</pre></td></tr></table></figure> 

重启 docker
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">systemctl restart docker</span>
</pre></td></tr></table></figure> 

push 到 registry
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># docker push $registry_url/google_containers/kube-ui:v4&#10;# docker push $registry_url/google_containers/pause-amd64:3.0&#10;# docker push $registry_url/google_containers/hyperkube:v1.5.3&#10;# docker push $registry_url/google_containers/pause:0.8.0</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u9A8C_u8BC1 "验证")验证

验证的过程是先删除本地的一个 images，在 pull 一次看是否成功
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># docker rmi $registry_url/google_containers/kube-ui:v4&#10;# docker pull $registry_url/google_containers/kube-ui:v4</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u521B_u5EFA_k8s "创建 k8s")创建 k8s

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#template "template")template

创建 template
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># magnum cluster-template-create k8s-cluster \&#10;--image fedora-atomic-ocata-raw \&#10;--keypair kolla1 \&#10;--external-network-id cbd41bb4-7e9c-4fb3-bcc8-3c23f7de0587 \&#10;--dns-nameserver 8.8.8.8 \&#10;--master-flavor m1.small \&#10;--flavor m1.large \&#10;--docker-volume-size 40 \&#10;--network-driver flannel \&#10;--coe kubernetes \&#10;--volume-driver cinder \&#10;--floating-ip-enabled \&#10;--fixed-network 4ca571c6-a37c-4035-8746-44c02e9c63bc \&#10;--fixed-subnet 3304e1d2-c3a4-48f3-9691-53072510ec70 \&#10;--insecure-registry 192.168.21.10:5000</span>
</pre></td></tr></table></figure> 

根据实际情况配置参数

*   flavor 使用默认的 m1.large
*   insecure-registry 配置 local registry
*   keypair 用来登录到各个节点作自定义配置
*   3 个 net 配置使用 id 的方式，否则使用 loadbalancer 会出错 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#k8s "k8s")k8s

创建 k8s
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># magnum cluster-create --node-count 1 --cluster-template k8s-cluster</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u914D_u7F6E_k8s__u96C6_u7FA4 "配置 k8s 集群")配置 k8s 集群

为了能够使用现有 iaas 的 loadbalance 和 cinder as storage 等 feature, 需要对已经创建的 k8s 集群手动作配置改动

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u914D_u7F6E_u8282_u70B9_hosts "配置节点 hosts")配置节点 hosts

在每个节点配置各个节点的 ip –&gt; hostname，e.g.
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo vi /etc/hosts&#10;172.16.0.55 ph-xmmx4bjtbp-0-asladitc3bww-kube-master-gzhiyy5bnysn&#10;172.16.0.52 ph-rnyzvwp3z5-0-n4ijjqf2cah4-kube-minion-lblxlwli5oj6</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#cloud-config "cloud-config")cloud-config

配置集群每个节点的 cloud-config， `/etc/sysconfig/kube_openstack_config` ，将 auth 用户换成 admin 用户
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">[Global]&#10;auth-url=http://192.168.21.100:5000/v2.0/ &#10;username=admin &#10;password=CAk723w8VaEpOpCzSm0SHWg5LK8LI4bEu0sT3o2H &#10;tenant-name=admin &#10;region=RegionOne &#10;trust-id=</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#master__u8282_u70B9 "master 节点")master 节点

k8s 的 controller-manager 使用 docker 启动，之需要修改对应的 manifests 文件 修改 `/etc/kubernetes/manifests/kube-controller-manager.yaml`
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">- controller-manager&#10;- --cloud-provider=openstack&#10;- --cloud-config=/etc/sysconfig/kube_openstack_config</span>
</pre></td></tr></table></figure> 

注意一定要将上述两个参数紧接着放在 `controller-manager` 之后

k8s 的 apiserver、kubelet 使用 systemd 管理，作如下修改

*   /etc/kubernetes/apiserver
*   /etc/kubernetes/kubelet 

在 `KUBE_API_ARGS` 后添加
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">--cloud-provider=openstack --cloud-config=/etc/sysconfig/kube_openstack_config</span>
</pre></td></tr></table></figure> <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo systemctl restart kube-apiserver.service&#10;$ sudo systemctl restart kubelet.service</span>
</pre></td></tr></table></figure> 

查询服务状态，确保正确
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo systemctl status kube-apiserver.service&#10;$ sudo systemctl status kubelet.service</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#slave__u8282_u70B9 "slave 节点")slave 节点

slave 只需要对 kubelet 作修改

*   /etc/kubernetes/kubelet <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">--cloud-provider=openstack --cloud-config=/etc/sysconfig/kube_openstack_config</span>
</pre></td></tr></table></figure> <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo systemctl restart kubelet.service</span>
</pre></td></tr></table></figure> 

查询服务状态，确保正确
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo systemctl status kubelet.service</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u9A8C_u8BC1-1 "验证")验证

use cinder in k8s [https://docs.openstack.org/magnum/latest/userguide.html#using-cinder-in-kubernetes](https://docs.openstack.org/magnum/latest/userguide.html#using-cinder-in-kubernetes)

use lbaas in k8s [https://github.com/openstack/magnum/blob/master/doc/source/dev/kubernetes-load-balancer.rst](https://github.com/openstack/magnum/blob/master/doc/source/dev/kubernetes-load-balancer.rst)

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#k8s-dns "k8s-dns")k8s-dns

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u90E8_u7F72-1 "部署")部署

需要的 docker 镜像

*   gcr.io/google_containers/kubedns-amd64:1.9
*   gcr.io/google_containers/dnsmasq-metrics-amd64:1.0
*   gcr.io/google_containers/kube-dnsmasq-amd64:1.4
*   gcr.io/google_containers/exechealthz-amd64:1.2 

(参照上面部分的方法将其放到 local registry)

修改 k8s 集群中节点的 kubelet 启动参数
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ vi /etc/kubernetes/kubelet&#10;KUBELET_ARGS=&#34;--cluster_dns=10.254.0.10 --cluster_domain=cluster.local&#34;</span>
</pre></td></tr></table></figure> <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo systemctl restart kubelet</span>
</pre></td></tr></table></figure> 

上传 k8s 的 config 等文件(即 magnum config xx 得到的文件)到集群各个节点
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># scp -r auth/ fedora@x.x.x.x:~/</span>
</pre></td></tr></table></figure> 

另一种方法使用 secret 来存储这些文件并 mount 到 container 内也可实现，需要修改描述文件中相关部分，secret 创建如下
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl create secret generic client-auth-files --from-file=config=./config --from-file=ca.pem=./ca.pem --from-file=cert.pem=./cert.pem --from-file=key.pem=./key.pem -n kube-system</span>
</pre></td></tr></table></figure> 

部署 dns，注意配置 skydns-rc.yaml 中的 –kube-master-url=[https://172.16.0.107:6443](https://172.16.0.107:6443)
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl create -f skydns-deployment.yaml</span>
</pre></td></tr></table></figure> 

yaml 参考 [https://github.com/ly798/k8s-deploy](https://github.com/ly798/k8s-deploy)

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u9A8C_u8BC1-2 "验证")验证

创建 busybox pod
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl create -f busybox.yaml</span>
</pre></td></tr></table></figure> 

使用该 pod 验证
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl exec busybox nslookup kubernetes</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#ingress "ingress")ingress

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u90E8_u7F72-2 "部署")部署

需要的 docker 镜像

*   gcr.io/google_containers/defaultbackend:1.0
*   gcr.io/google_containers/nginx-ingress-controller:0.9.0-beta.11 

(参照上面部分的方法将其放到 local registry)

部署 imgress
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl create -f ingress-deployment.yaml</span>
</pre></td></tr></table></figure> 

yaml 参考 [https://github.com/ly798/k8s-deploy](https://github.com/ly798/k8s-deploy)

通过`kubectl -n kube-system describe svc ingress-nginx`找到 loadbalance，在 openstack 中找到并为其添加一个 floatingip

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u9A8C_u8BC1-3 "验证")验证

创建一个 nginx 和一个 ingress 进行验证
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl create ./nginx-ingress-path/&#10;# kubectl get ing --all-namespaces&#10;NAMESPACE NAME HOSTS ADDRESS PORTS AGE&#10;default nginx-ingress * 192.168.21.236 80 2m&#10;# curl 192.168.21.236/nginx&#10;&#60;html&#62;&#10;&#60;head&#62;&#60;title&#62;301 Moved Permanently&#60;/title&#62;&#60;/head&#62;&#10;&#60;body bgcolor=&#34;white&#34;&#62;&#10;&#60;center&#62;&#60;h1&#62;301 Moved Permanently&#60;/h1&#62;&#60;/center&#62;&#10;&#60;hr&#62;&#60;center&#62;nginx/1.13.3&#60;/center&#62;&#10;&#60;/body&#62;&#10;&#60;/html&#62;</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#k8s-dashboard "k8s-dashboard")k8s-dashboard

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u90E8_u7F72-3 "部署")部署

需要的 docker 镜像

*   gcr.io/google_containers/kubernetes-dashboard-amd64:v1.5.1 

(注意版本对应，参照上面部分的方法将其放到 local registry)

部署 imgress
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl create -f kube-dashboard-deployment.yaml</span>
</pre></td></tr></table></figure> 

yaml 参考 [https://github.com/ly798/k8s-deploy](https://github.com/ly798/k8s-deploy)

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u9A8C_u8BC1-4 "验证")验证

使用 proxy 实现访问
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># kubectl proxy</span>
</pre></td></tr></table></figure> 

然后是使用浏览器访问 [http://127.0.0.1:8001/ui](http://127.0.0.1:8001/ui) 即可

还可以使用 ingress 的方式来实现访问，但有安全问题，ingress 如下
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">apiVersion: extensions/v1beta1&#10;kind: Ingress&#10;metadata:&#10; name: kubernetes-dashboard-ingress&#10; namespace: kube-system&#10;spec:&#10; rules:&#10; - host: ui.myk8s.com&#10; http:&#10; paths:&#10; - backend:&#10; serviceName: kubernetes-dashboard&#10; servicePort: 80</span>
</pre></td></tr></table></figure> 

访问 [http://ui.myk8s.com](http://ui.myk8s.com) 即可

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#FAQ "FAQ")FAQ

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u6B63_u5E38_u96C6_u7FA4_u7684_docker_images__u548C_u670D_u52A1 "正常集群的 docker images 和服务")正常集群的 docker images 和服务

master node:
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">[fedora@k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz ~]$ sudo docker images&#10;REPOSITORY TAG IMAGE ID CREATED SIZE&#10;gcr.io/google_containers/hyperkube v1.5.3 08fdc4720263 5 months ago 557.5 MB&#10;gcr.io/google_containers/pause-amd64 3.0 99e59f495ffa 14 months ago 746.9 kB&#10;[fedora@k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz ~]$ sudo docker ps&#10;CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES&#10;9cd5a7a5b52c gcr.io/google_containers/hyperkube:v1.5.3 &#34;/hyperkube controlle&#34; 4 minutes ago Up 4 minutes k8s_kube-controller-manager.d1de1666_kube-controller-manager-k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz_kube-system_94d16460f7db6bd63dd617ff8527deb6_fc4d74ee&#10;3c7e9189f60c gcr.io/google_containers/hyperkube:v1.5.3 &#34;/hyperkube scheduler&#34; 4 minutes ago Up 4 minutes k8s_kube-scheduler.a3560f4b_kube-scheduler-k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz_kube-system_cd57cab0d68c7d6b68604c1873b9fd23_fedadfaf&#10;9e7f59b94ecf gcr.io/google_containers/hyperkube:v1.5.3 &#34;/hyperkube proxy --m&#34; 4 minutes ago Up 4 minutes k8s_kube-proxy.adc74569_kube-proxy-k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz_kube-system_00bbe3f517c93071991098603d8809d7_e05e71a0&#10;f03216cf985b gcr.io/google_containers/pause-amd64:3.0 &#34;/pause&#34; 6 minutes ago Up 6 minutes k8s_POD.d8dbe16c_kube-controller-manager-k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz_kube-system_94d16460f7db6bd63dd617ff8527deb6_c3f76b6b&#10;420045f9d8c7 gcr.io/google_containers/pause-amd64:3.0 &#34;/pause&#34; 7 minutes ago Up 6 minutes k8s_POD.d8dbe16c_kube-scheduler-k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz_kube-system_cd57cab0d68c7d6b68604c1873b9fd23_d9333644&#10;825df619192b gcr.io/google_containers/pause-amd64:3.0 &#34;/pause&#34; 7 minutes ago Up 7 minutes k8s_POD.d8dbe16c_kube-proxy-k2-tgqfxoqymo-0-2lgmxw53emmp-kube-master-tfqau4357qaz_kube-system_00bbe3f517c93071991098603d8809d7_be91ac8e</span>
</pre></td></tr></table></figure> 

miniom node:
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">[fedora@k2-qbjizye63f-0-dopyfnyolgvh-kube-minion-ib42uvm3myve ~]$ sudo docker images&#10;REPOSITORY TAG IMAGE ID CREATED SIZE&#10;gcr.io/google_containers/hyperkube v1.5.3 08fdc4720263 5 months ago 557.5 MB&#10;gcr.io/google_containers/pause-amd64 3.0 99e59f495ffa 14 months ago 746.9 kB&#10;gcr.io/google_containers/kube-ui v4 762cc97d5aa1 19 months ago 5.677 MB&#10;[fedora@k2-qbjizye63f-0-dopyfnyolgvh-kube-minion-ib42uvm3myve ~]$ sudo docker ps&#10;CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES&#10;1c5b7b9a90fd gcr.io/google_containers/kube-ui:v4 &#34;/kube-ui&#34; 2 minutes ago Up 2 minutes k8s_kube-ui.53edfc94_kube-ui-v4-8vczr_kube-system_de2938a2-7104-11e7-8a3f-fa163e74b5a1_25809512&#10;49328e69c8b3 gcr.io/google_containers/hyperkube:v1.5.3 &#34;/hyperkube proxy --m&#34; 3 minutes ago Up 2 minutes k8s_kube-proxy.e8846ca1_kube-proxy-k2-qbjizye63f-0-dopyfnyolgvh-kube-minion-ib42uvm3myve_kube-system_dc6c88a4f0d6b3341f1b942f3dbf8885_dd7f1dc1&#10;c00f5c8e8bd7 gcr.io/google_containers/pause-amd64:3.0 &#34;/pause&#34; 4 minutes ago Up 3 minutes k8s_POD.1e570369_kube-ui-v4-8vczr_kube-system_de2938a2-7104-11e7-8a3f-fa163e74b5a1_518ad58d&#10;6cc5cfda187f gcr.io/google_containers/pause-amd64:3.0 &#34;/pause&#34; 5 minutes ago Up 4 minutes k8s_POD.d8dbe16c_kube-proxy-k2-qbjizye63f-0-dopyfnyolgvh-kube-minion-ib42uvm3myve_kube-system_dc6c88a4f0d6b3341f1b942f3dbf8885_0f31b601</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#k8s__u96C6_u7FA4_u7684_u6269_u5C55 "k8s 集群的扩展")k8s 集群的扩展

1.  直接使用 magnum update 方式扩展节点个数
2.  节点 docker pool 的扩展

    可直接给对应 node 添加 cinder 卷，对 docker-pool 使用 lvm 进行扩展 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#k8s__u96C6_u7FA4_u8282_u70B9_u91CD_u542F "k8s 集群节点重启")k8s 集群节点重启

重启后到各节点上启动 docker 实例
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ sudo docker start `sudo docker ps -a -q`</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#u4F7F_u7528_LoadBalancer__u65F6_u62A5_u9519 "使用 LoadBalancer 时报错")使用 LoadBalancer 时报错

在 master 节点报错
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">Aug 09 07:16:56 ph-xmmx4bjtbp-0-asladitc3bww-kube-master-gzhiyy5bnysn.novalocal dockerd-current[2295]: I0809 07:16:56.799446 1 event.go:217] Event(api.ObjectReference&#123;Kind:&#34;ReplicaSet&#34;, Namespace:&#34;default&#34;, Name:&#34;nginx-deployment-1045676374&#34;, UID:&#34;ba01e2a2-7cd2-11e7-82c2-fa163ed8eebc&#34;, APIVersion:&#34;extensions&#34;, ResourceVersion:&#34;1484&#34;, FieldPath:&#34;&#34;&#125;): type: &#39;Normal&#39; reason: &#39;SuccessfulCreate&#39; Created pod: nginx-deployment-1045676374-618mn&#10;Aug 09 07:16:56 ph-xmmx4bjtbp-0-asladitc3bww-kube-master-gzhiyy5bnysn.novalocal dockerd-current[2295]: E0809 07:16:56.932792 1 servicecontroller.go:760] Failed to process service. Retrying in 5s: Failed to create load balancer for service default/nginxservice-loadbalancer: Error creating loadbalancer ab9e95e8d7cd211e782c2fa163ed8eeb: Error creating loadbalancer &#123;ab9e95e8d7cd211e782c2fa163ed8eeb Kubernetes external service ab9e95e8d7cd211e782c2fa163ed8eeb magnum-subnet &#60;nil&#62; &#125;: Expected HTTP response code [201 202] when accessing [POST http://192.168.21.100:9696/v2.0/lbaas/loadbalancers], but got 400 instead&#10;Aug 09 07:16:56 ph-xmmx4bjtbp-0-asladitc3bww-kube-master-gzhiyy5bnysn.novalocal dockerd-current[2295]: &#123;&#34;NeutronError&#34;: &#123;&#34;message&#34;: &#34;Invalid input for vip_subnet_id. Reason: &#39;magnum-subnet&#39; is not a valid UUID.&#34;, &#34;type&#34;: &#34;HTTPBadRequest&#34;, &#34;detail&#34;: &#34;&#34;&#125;&#125;</span>
</pre></td></tr></table></figure> 

template 中的 net 相关配置不能使用 name，要使用 uuid

## [](https://ly798.github.io/2017/07/31/magnum%E9%83%A8%E7%BD%B2k8s/#reference "reference")reference

*   [https://docs.openstack.org/magnum/latest/](https://docs.openstack.org/magnum/latest/)
*   [https://mritd.me/2017/03/04/how-to-use-nginx-ingress/](https://mritd.me/2017/03/04/how-to-use-nginx-ingress/)
*   [https://github.com/kubernetes/dashboard/blob/v1.5.1/src/deploy/kubernetes-dashboard.yaml](https://github.com/kubernetes/dashboard/blob/v1.5.1/src/deploy/kubernetes-dashboard.yaml)
*   [https://github.com/kubernetes/dashboard/blob/master/docs/user-guide/troubleshooting.md](https://github.com/kubernetes/dashboard/blob/master/docs/user-guide/troubleshooting.md)
