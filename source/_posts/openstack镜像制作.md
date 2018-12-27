---
title: openstack镜像制作
tags: [openstack]
date: 2017-12-01 10:15:45
---

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#introduction "introduction")introduction

整理 openstack 镜像制作过程
 <!-- more --> 

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u76F4_u63A5_u4FEE_u6539_u5B98_u65B9_u955C_u50CF "直接修改官方镜像")直接修改官方镜像

常用 linux 官方提供 cloud 镜像，可以直接基于其镜像修改，以 ubuntu16.04 为例

下载 ubuntu16.04 cloud image
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">axel -n 2 https://cloud-images.ubuntu.com/xenial/current/xenial-server-cloudimg-amd64-disk1.img</span>
</pre></td></tr></table></figure> 

使用 guestfish，安装环境
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">sudo apt install libguestfs-tools -y</span>
</pre></td></tr></table></figure> 

启动镜像
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">export LIBGUESTFS_BACKEND=direct&#10;guestfish --rw -a ./xenial-server-cloudimg-amd64-disk1.img #&#23454;&#38469;&#19978;&#21551;&#21160;&#20102;&#19968; qemu &#34394;&#25311;&#26426;</span>
</pre></td></tr></table></figure> 

进入了 guestfish
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">&#62;&#60;fs&#62; run&#10;&#62;&#60;fs&#62; list-filesystems&#10;&#62;&#60;fs&#62; mount /dev/sda1 /&#10;&#62;&#60;fs&#62; vi /etc/cloud/cloud.cfg</span>
</pre></td></tr></table></figure> 

接下来修改 cloudinit 来达到新建用户的目的，/etc/cloud/cloud.cfg
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">users:&#10; - default&#10;..&#10;&#10;ssh_pwauth: true&#10;manage_etc_hosts: true&#10;..&#10;cloud_init_modules:&#10;..&#10; - ssh&#10; - bootcmd&#10;&#10;bootcmd:&#10; - addgroup test&#10; - usermod -d /home/test -m -g test -l test ubuntu&#10; - echo &#34;test\ntest&#34; | passwd test</span>
</pre></td></tr></table></figure> 

最后退出，转换格式为 raw 并上传
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">&#62;&#60;fs&#62; exit&#10;qemu-img convert -O raw xenial-server-cloudimg-amd64-disk1.img xenial-server-cloudimg-amd64-disk1.raw&#10;openstack image create --disk-format raw --container-format bare --public --file ./xenial-server-cloudimg-amd64-disk1.raw xenial-server-cloudimg-amd64-disk1.raw</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u624B_u52A8_u521B_u5EFA_image "手动创建 image")手动创建 image

TODO

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u4F7F_u7528_u5DE5_u5177_u5236_u4F5C_image "使用工具制作 image")使用工具制作 image

TODO

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u4F7F_u7528_snapshot__u5236_u4F5C_image "使用 snapshot 制作 image")使用 snapshot 制作 image

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u521B_u5EFA_u4E00_u4E2A_u4E91_u4E3B_u673A "创建一个云主机")创建一个云主机
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># nova list&#10;+--------------------------------------|----------------|---------|------------|-------------|-----------------------------------------+&#10;| ID | Name | Status | Task State | Power State | Networks |&#10;+--------------------------------------|----------------|---------|------------|-------------|-----------------------------------------+&#10;| 550db3bb-bd47-4b5b-8a1d-ed465d147618 | centos-tempate | ACTIVE | - | Running | demo-net=10.0.0.24 |&#10;+--------------------------------------|----------------|---------|------------|-------------|-----------------------------------------+</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u5B89_u88C5_u9700_u8981_u7684_u8F6F_u4EF6 "安装需要的软件")安装需要的软件

到云主机安装需要的软件配置等

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u6E05_u7406_u7CFB_u7EDF "清理系统")清理系统

清理系统不需要文件和空洞
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># rm -f ~/.ssh/id_rsa*&#10;# rm -f /home/centos/.ssh/id_rsa*&#10;# sudo rm -rf /var/lib/cloud/*&#10;# dd if=/dev/zero of=zerofile # &#33457;&#36153;&#30340;&#26102;&#38388;&#36739;&#38271;&#65292;&#24314;&#35758;&#21019;&#24314;&#34394;&#25311;&#26426;&#30340;&#26102;&#20505;&#30913;&#30424;&#19981;&#35201;&#22826;&#22823;&#10;dd: writing to &#8216;zerofile&#8217;: No space left on device&#10;30952873+0 records in&#10;30952872+0 records out&#10;15847870464 bytes (16 GB) copied, 548.355 s, 28.9 MB/s&#10;# rm zerofile</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u5173_u673A "关机")关机

关机后保证云主机状态是 shutdown

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u521B_u5EFA_share_image "创建 share image")创建 share image

从云主机的卷创建 image
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># openstack volume set --state available e0d90bde-6f16-4ef5-ad96-40163c1deabe&#10;# openstack image create new-centos-image --volume e0d90bde-6f16-4ef5-ad96-40163c1deabe&#10;# openstack volume set --state in-use e0d90bde-6f16-4ef5-ad96-40163c1deabe</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u4E0B_u8F7D_share_image__u5E76_u8F6C_u6362 "下载 share image 并转换")下载 share image 并转换
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># openstack image save --file new-centos-image.raw a029c88b-3c4b-4ccf-a927-4b6da7c389a6&#10;# openstack image delete a029c88b-3c4b-4ccf-a927-4b6da7c389a6</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u5220_u9664_tmp__u8D44_u6E90 "删除 tmp 资源")删除 tmp 资源

删除云主机、tmp volume、tmp image
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># openstack volume server centos-tempate&#10;# openstack image delete a029c88b-3c4b-4ccf-a927-4b6da7c389a6</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u4E0A_u4F20_images "上传 images")上传 images
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># openstack image create \&#10;--disk-format raw \&#10;--container-format bare \&#10;--property skip_atmosphere=yes \&#10;--file new-centos-image.raw \&#10;--public new-centos-image</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u5236_u4F5C_windows_image "制作 windows image")制作 windows image

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u521B_u5EFA_u78C1_u76D8 "创建磁盘")创建磁盘
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">qemu-img create -f qcow2 win7.qcow2 20G</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u542F_u52A8_u5E76_u5B89_u88C5_u7CFB_u7EDF "启动并安装系统")启动并安装系统
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">/usr/libexec/qemu-kvm -m 1024 -cdrom win7.iso -drive file=win7.qcow2,if=virtio,boot=on -fda virtio-win_amd64.vfd -boot d -nographic -vnc :3</span>
</pre></td></tr></table></figure> 

使用 vnc 连接进入安装系统界面，安装 virtio 磁盘驱动并安装系统
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">vncviewer 127.0.01:5903</span>
</pre></td></tr></table></figure> 

安装完成后关机

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u914D_u7F6E_u7CFB_u7EDF "配置系统")配置系统
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">/usr/libexec/qemu-kvm -m 1024 -cdrom virtio-win.iso -drive file=win7.qcow2,if=virtio,boot=on -fda virtio-win_amd64.vfd -net nic,model=virtio -net user -boot c -nographic -vnc :3</span>
</pre></td></tr></table></figure> 

使用 vnc 连接进入安装系统

*   设备管理器安装 virtio 网络驱动
*   系统高级设置打开远程连接
*   下载 cloudbase-init 并安装(密码设置插件、磁盘自动分区等功能) 

配置完成后关机

### [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#u8F6C_u6362_u683C_u5F0F "转换格式")转换格式
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">qemu-img convert -f qcow2 -O raw win7.qcow2 win7.raw</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#image_tools "image tools")image tools

直接操作 image，例如本地挂载

TODO

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#convert_format "convert format")convert format
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># qemu-img convert -f raw -O qcow2 image.img image.qcow2</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#FAQ "FAQ")FAQ

TODO

## [](https://ly798.github.io/2017/12/01/openstack%E9%95%9C%E5%83%8F%E5%88%B6%E4%BD%9C/#reference "reference")reference

*   [https://docs.openstack.org/image-guide/](https://docs.openstack.org/image-guide/)
*   [https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/72716305/Creating+snapshots+and+new+Glance+images+from+the+command+line](https://iujetstream.atlassian.net/wiki/spaces/JWT/pages/72716305/Creating+snapshots+and+new+Glance+images+from+the+command+line)
*   [http://khmel.org/?p=1188](http://khmel.org/?p=1188)
*   [http://www.tk4479.net/liukuan73/article/details/47252561](http://www.tk4479.net/liukuan73/article/details/47252561)
