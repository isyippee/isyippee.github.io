---
title: kolla-deploy
tags: [openstack]
date: 2017-06-26 10:18:21
---

## [](https://ly798.github.io/2017/06/26/kolla-deploy/#introduction "introduction")introduction

*   kolla 包含各种 openstack 服务 images 的构建相关的东西，dockerfile、build 工具等等
*   kolla-ansible 是一个用来部署 kolla 的工具，使用 ansible 来检查部署环境、pull 已经构建好的 images、启动 docker openstack 服务 <!-- more --> 

## [](https://ly798.github.io/2017/06/26/kolla-deploy/#single "single")single

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#u73AF_u5883 "环境")环境

*   centos7
*   8G 内存
*   2 张网卡，一张配置固定 ip（10.0.20.100）用作管理网络，另一张 **ip link set eth1 up** 用作 external 

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#u4F9D_u8D56 "依赖")依赖
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum install -y epel-release&#10;yum install -y python-pip&#10;pip install -U pip&#10;&#10;yum install -y python-devel libffi-devel gcc openssl-devel&#10;&#10;yum install -y ansible</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#docker "docker")docker

1.  安装
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">curl -sSL https://get.docker.io | bash</span>
</pre></td></tr></table></figure>
    或者参考 [https://docs.docker.com/engine/installation/linux/centos/#install-using-the-repository](https://docs.docker.com/engine/installation/linux/centos/#install-using-the-repository)
2.  配置 daocloud 加速器

    参考 [https://www.daocloud.io/mirror](https://www.daocloud.io/mirror)
3.  配置 docker
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">mkdir -p /etc/systemd/system/docker.service.d&#10;tee /etc/systemd/system/docker.service.d/kolla.conf &#60;&#60;&#39;EOF&#39;&#10;[Service]&#10;MountFlags=shared&#10;EOF&#10;&#10;systemctl daemon-reload&#10;systemctl restart docker</span>
</pre></td></tr></table></figure>4.  python-docker-py
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum install -y python-docker-py</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#ntp_u3001libvirt "ntp、libvirt")ntp、libvirt
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum install -y ntp&#10;systemctl enable ntpd.service&#10;systemctl start ntpd.service</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#kolla-ansible "kolla-ansible")kolla-ansible

1.  安装
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">pip install kolla-ansible&#10;cp -r /usr/share/kolla-ansible/etc_examples/kolla /etc/kolla/&#10;cp /usr/share/kolla-ansible/ansible/inventory/* .</span>
</pre></td></tr></table></figure>2.  配置

    修改 **/etc/kolla/globals.yml**
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">openstack_release: &#34;4.0.0&#34; #&#20114;&#32852;&#32593;&#19978;&#30340; images &#21482;&#26377; 4.0.0&#65292;&#33509;&#38656;&#35201;&#26368;&#26032;&#29256;&#30340;&#38656;&#35201;&#33258;&#24049; build&#10;docker_registry: &#34;docker.mirrors.ustc.edu.cn&#34; &#10;network_interface: &#34;eth0&#34;&#10;neutron_external_interface: &#34;eth1&#34;</span>
</pre></td></tr></table></figure>3.  生成集群各种密码
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-genpwd</span>
</pre></td></tr></table></figure>
    文件位置 **/etc/kolla/passwords.yml** , 可根据需要修改
4.  检查环境
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-ansible -i &#60;&#60;inventory file&#62;&#62; bootstrap-servers</span>
</pre></td></tr></table></figure>5.  build images

    两种方式：

        *   从 kolla 的 dockerhub 拉取构建好的测试镜像 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-ansible pull -vvv</span>
</pre></td></tr></table></figure>
        *   重新构建上游 images
    参考 [http://docs.openstack.org/developer/kolla/image-building.html](http://docs.openstack.org/developer/kolla/image-building.html)

    构建后 **docker images** 可查询到
6.  部署

    修改 **/etc/kolla/globals.yml** 配置参数 **kolla_internal_vip_address** 为选择的 vip

    检查是否支持 **vmx**
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">egrep -c &#39;(vmx|svm)&#39; /proc/cpuinfo</span>
</pre></td></tr></table></figure>
    若不支持，需要做如下配置：
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">mkdir -p /etc/kolla/config/nova&#10;cat &#60;&#60; EOF &#62; /etc/kolla/config/nova/nova-compute.conf&#10;[libvirt]&#10;virt_type = qemu&#10;cpu_mode = none&#10;EOF</span>
</pre></td></tr></table></figure>
    执行检查和部署
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-ansible prechecks -i ./all-in-one&#10;kolla-ansible deploy -i ./all-in-one</span>
</pre></td></tr></table></figure>
    部署成功后，生成 **admin-openrc.sh**
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-ansible post-deploy</span>
</pre></td></tr></table></figure>
    该文件位置在 **_etc/kolla_**
7.  验证
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">. /etc/kolla/admin-openrc.sh&#10;cd /usr/share/kolla-ansible&#10;./init-runonce</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/06/26/kolla-deploy/#multinode "multinode")multinode

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#u73AF_u5883_u4ECB_u7ECD "环境介绍")环境介绍

*   3 个节点，系统 centos7，另外一块磁盘
*   4G 内存
*   2 张网卡，一张配置固定 ip 用作管理网络，另一张 **ip link set eth1 up** 用作 external <table> <thead> <tr> <th>hostname</th> <th>eth0</th> <th>eth1</th> </tr> </thead> <tbody> <tr> <td>kolla1</td> <td>10.0.20.100</td> <td>up</td> </tr> <tr> <td>kolla2</td> <td>10.0.20.101</td> <td>up</td> </tr> <tr> <td>kolla3</td> <td>10.0.20.102</td> <td>up</td> </tr> </tbody> </table> 

*   3 个节点均担任 controller、network、compute 角色 

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#u90E8_u7F72 "部署")部署

1.  配置

    各个节点都需要安装基础的环境 docker、ntp 等, 与上述单节点类似，总结如下
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum install -y epel-release&#10;yum install -y python-pip&#10;pip install -U pip&#10;&#10;yum install -y python-devel libffi-devel gcc openssl-devel ansible&#10;&#10;curl -sSL https://get.docker.io | bash&#10;&#10;mkdir -p /etc/systemd/system/docker.service.d&#10;tee /etc/systemd/system/docker.service.d/kolla.conf &#60;&#60;&#39;EOF&#39;&#10;[Service]&#10;MountFlags=shared&#10;EOF&#10;&#10;systemctl daemon-reload&#10;systemctl restart docker&#10;&#10;yum install -y python-docker-py&#10;&#10;yum install -y ntp&#10;systemctl enable ntpd.service&#10;systemctl start ntpd.service</span>
</pre></td></tr></table></figure>
    在 3 个节点中, 需要选择一个作为部署节点（选择 kolla1）, 在该节点需要安装 **kolla-ansible** ，并部署节点需要免密码登陆所有节点
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">pip install kolla-ansible&#10;ssh-copy-id root@kolla1&#10;ssh-copy-id root@kolla2&#10;ssh-copy-id root@kolla3</span>
</pre></td></tr></table></figure>
    拷贝配置文件
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cp -r /usr/share/kolla-ansible/etc_examples/kolla /etc/kolla/</span>
</pre></td></tr></table></figure>
    以家目录（/root）为 kolla-ansible 的工作目录，将 inventory 配置文件拷贝到此
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cp /usr/share/kolla-ansible/ansible/inventory/* .</span>
</pre></td></tr></table></figure>
    关于 **/etc/kolla/globals.yml** 基本的配置与单节点类似，需要注意的是 ceph 和 cinder 部分
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">enable_ceph: &#34;yes&#34;&#10;enable_ceph_rgw: &#34;yes&#34;&#10;enable_cinder: &#34;yes&#34;&#10;glance_backend_ceph: &#34;yes&#34;</span>
</pre></td></tr></table></figure>
    需要说明的一个参数
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># docker_registry: &#34;10.0.20.1:5000&#34; # &#26368;&#22909;&#25442;&#25104; local registry</span>
</pre></td></tr></table></figure>
    指向的是 docker images 的 namespace，有三种方案

        *   使用互联网上其他 mirror 的 images
    *   本地搭建 registry，参考 [https://docs.docker.com/registry/](https://docs.docker.com/registry/) (有一定规模的时候推荐)
    *   使用提供的 docker images 归档文件，进行 load 到本地(节点较少时推荐) <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">docker load -i kolla-4.0.2.tar.gz</span>
</pre></td></tr></table></figure>
    ceph osd 是通过磁盘 lable 来识别的，需要在各个节点上操作，使用第二块磁盘来作为 osd，至于查找第二块磁盘
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">lsblk</span>
</pre></td></tr></table></figure>
    打上 label
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">parted /dev/vdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP 1 -1</span>
</pre></td></tr></table></figure>
    生成集群各种密码
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-genpwd</span>
</pre></td></tr></table></figure>
    文件位置 **/etc/kolla/passwords.yml** , 可根据需要修改

    编辑 **multinode** 文件，进行基本角色分配
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">[control]&#10;kolla1&#10;kolla2&#10;kolla3&#10;[network]&#10;kolla1&#10;kolla2&#10;kolla3&#10;[compute]&#10;kolla1&#10;kolla2&#10;kolla3&#10;[monitoring]&#10;kolla1&#10;kolla2&#10;kolla3&#10;[storage]&#10;kolla1&#10;kolla2&#10;kolla3</span>
</pre></td></tr></table></figure>2.  检测并部署
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-ansible -i multinode bootstrap-servers&#10;kolla-ansible prechecks -i multinode&#10;kolla-ansible deploy -i multinode</span>
</pre></td></tr></table></figure>3.  调整日志

    方便查询日志，可选步骤
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">ln -sf /var/lib/docker/volumes/kolla_logs/_data/ /var/log/kolla</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/06/26/kolla-deploy/#u5E38_u89C1_u9519_u8BEF "常见错误")常见错误

### [](https://ly798.github.io/2017/06/26/kolla-deploy/#u9047_u5230_u9519_u8BEF_u91CD_u65B0_u90E8_u7F72_kolla "遇到错误重新部署 kolla")遇到错误重新部署 kolla

清理上次部署的资源
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">kolla-ansilbe destory -i multinode --yes-i-really-really-mean-it</span>
</pre></td></tr></table></figure> 

确认三个节点没有遗留 docker volume，若有将其删除
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">docker volume ls&#10;docker volume rm &#60;name&#62;</span>
</pre></td></tr></table></figure> 

若有 osd，将对应的磁盘重置, 如
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">umount /dev/vdb&#10;fdisk /dev/vdb&#10;d d w</span>
</pre></td></tr></table></figure> 

再给其打上 osd 的 lable
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">parted /dev/vdb -s -- mklabel gpt mkpart KOLLA_CEPH_OSD_BOOTSTRAP 1 -1</span>
</pre></td></tr></table></figure> 

重启系统清除 iptables 规则

## [](https://ly798.github.io/2017/06/26/kolla-deploy/#u53C2_u8003 "参考")参考

*   [https://docs.openstack.org/developer/kolla-ansible/quickstart.html#recommended-environment](https://docs.openstack.org/developer/kolla-ansible/quickstart.html#recommended-environment)
*   [https://docs.openstack.org/developer/kolla-ansible/quickstart.html](https://docs.openstack.org/developer/kolla-ansible/quickstart.html)
*   [https://docs.openstack.org/developer/kolla-ansible/ceph-guide.html](https://docs.openstack.org/developer/kolla-ansible/ceph-guide.html)
*   [https://bugs.launchpad.net/kolla/+bug/1558853](https://bugs.launchpad.net/kolla/+bug/1558853)
