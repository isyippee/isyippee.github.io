---
title: devstack trove
tags: [openstack trove]
date: 2016-11-24 10:23:44
---

## [](https://ly798.github.io/2016/11/24/devstack-trove/#u4ECB_u7ECD "介绍")介绍

快速搭建一个trove的环境，按照trove wiki来，[http://docs.openstack.org/developer/trove/dev/install.html](http://docs.openstack.org/developer/trove/dev/install.html)

源代码 [https://github.com/openstack/trove-integration](https://github.com/openstack/trove-integration)

trove-integration使用了devstack来搭建openstack环境，另外对trove进行配置，包含trove image制作工具。
 <!-- more --> 

## [](https://ly798.github.io/2016/11/24/devstack-trove/#u5B89_u88C5 "安装")安装

系统ubuntu 14.04

简单的几步

配置网络、配置apt源；
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">vim /etc/ssh/sshd_config</span>
<span class="line"><span class="comment">#PermitRootLogin prohibit-password</span></span>
<span class="line">PermitRootLogin yes</span>
</pre></td></tr></table></figure> <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
</pre></td><td class="code"><pre><span class="line">vim /etc/network/interfaces</span>
<span class="line"><span class="comment"># The primary network interface</span></span>
<span class="line">auto enp0s3</span>
<span class="line">iface enp0s3 inet static</span>
<span class="line">address <span class="number">192.168</span>.<span class="number">31.150</span></span>
<span class="line">netmask <span class="number">255.255</span>.<span class="number">255.0</span></span>
<span class="line">gateway <span class="number">192.168</span>.<span class="number">31.1</span></span>
<span class="line">dns-nameserver <span class="number">114.114</span>.<span class="number">114.114</span></span>
</pre></td></tr></table></figure> <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
<span class="line">11</span>
<span class="line">12</span>
</pre></td><td class="code"><pre><span class="line">cp /etc/apt/sources.list /etc/apt/sources.list.bak</span>
<span class="line"><span class="built_in">echo</span> &gt; /etc/apt/sources.list</span>
<span class="line">vim /etc/apt/sources.list</span>
<span class="line"><span class="comment"># 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释</span></span>
<span class="line">deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main restricted universe multiverse</span>
<span class="line"><span class="comment"># deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial main main restricted universe multiverse</span></span>
<span class="line">deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse</span>
<span class="line"><span class="comment"># deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-updates main restricted universe multiverse</span></span>
<span class="line">deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse</span>
<span class="line"><span class="comment"># deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-backports main restricted universe multiverse</span></span>
<span class="line">deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse</span>
<span class="line"><span class="comment"># deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ xenial-security main restricted universe multiverse</span></span>
</pre></td></tr></table></figure> 

配置pip源

~/.pip/pip.conf
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line">[global]</span>
<span class="line">index-url = http:<span class="comment">//mirrors.aliyun.com/pypi/simple</span></span>
<span class="line"><span class="id">#index-url</span> = https:<span class="comment">//pypi.tuna.tsinghua.edu.cn/simple</span></span>
<span class="line">[install]</span>
<span class="line">trusted-host = mirrors<span class="class">.aliyun</span><span class="class">.com</span></span>
</pre></td></tr></table></figure>
 <figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># apt-get update</span></span>
<span class="line"><span class="preprocessor"># apt-get install git-core -y</span></span>
</pre></td></tr></table></figure> <figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># adduser ubuntu</span></span>
<span class="line"><span class="preprocessor"># visudo</span></span>
</pre></td></tr></table></figure> 

添加
<figure class="highlight apache"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">ubuntu</span> <span class="literal">ALL</span>=(<span class="literal">ALL</span>) NOPASSWD: <span class="literal">ALL</span></span>
</pre></td></tr></table></figure>

关于 Manila 参考[https://wiki.openstack.org/wiki/Manila/ManilaDevstack](https://wiki.openstack.org/wiki/Manila/ManilaDevstack)
<!-- cd /home git clone http://git.trystack.cn/openstack-dev/devstack.git cd devstack/tools/ ./create-stack-user.sh chown -R stack:stack /home/devstack chmod 777 /dev/pts/0 su stack cd /home/devstack vim local.conf <figure class="highlight ruby"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
<span class="line">11</span>
<span class="line">12</span>
<span class="line">13</span>
<span class="line">14</span>
<span class="line">15</span>
<span class="line">16</span>
<span class="line">17</span>
<span class="line">18</span>
<span class="line">19</span>
<span class="line">20</span>
<span class="line">21</span>
<span class="line">22</span>
</pre></td><td class="code"><pre><span class="line">[[local|localrc]]</span>
<span class="line"><span class="comment"># Define images to be automatically downloaded during the DevStack built process.</span></span>
<span class="line"><span class="constant">DOWNLOAD_DEFAULT_IMAGES</span>=<span class="constant">False</span></span>
<span class="line"><span class="constant">IMAGE_URLS</span>=<span class="string">"http://download.cirros-cloud.net/0.3.4/cirros-0.3.4-x86_64-disk.img"</span></span>
<span class="line"></span>
<span class="line"><span class="comment">#change github to trystack</span></span>
<span class="line"><span class="constant">GIT_BASE</span>=<span class="variable">$&#123;</span><span class="constant">GIT_BASE</span><span class="symbol">:-http</span><span class="symbol">://git</span>.trystack.cn&#125;</span>
<span class="line"><span class="constant">NOVNC_REPO</span>=<span class="variable">$&#123;</span><span class="constant">NOVNC_REPO</span><span class="symbol">:-http</span><span class="symbol">://git</span>.trystack.cn/kanaka/noVNC.git&#125;</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="comment"># Credentials</span></span>
<span class="line"><span class="constant">DATABASE_PASSWORD</span>=<span class="number">125390</span></span>
<span class="line"><span class="constant">ADMIN_PASSWORD</span>=admin</span>
<span class="line"><span class="constant">SERVICE_PASSWORD</span>=<span class="number">125390</span></span>
<span class="line"><span class="constant">SERVICE_TOKEN</span>=<span class="number">125390</span></span>
<span class="line"><span class="constant">RABBIT_PASSWORD</span>=<span class="number">125390</span></span>
<span class="line"></span>
<span class="line"><span class="comment">#PUBLIC_INTERFACE=enp0s3</span></span>
<span class="line"></span>
<span class="line"><span class="constant">SERVICE_IP_VERSION</span>=<span class="number">4</span></span>
<span class="line"></span>
<span class="line">enable_plugin manila <span class="symbol">http:</span>/<span class="regexp">/git.trystack.cn/openstack</span><span class="regexp">/manila</span></span>
</pre></td></tr></table></figure>

拷贝cirros-0.3.4-x86_64-disk.img manila-service-image-master.qcow2 get-pip.py到files下
 –&gt;
使用ubuntu用户来进行安装
 <figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># git clone http:<span class="comment">//git.trystack.cn/openstack/trove-integration</span></span></span>
<span class="line"><span class="preprocessor"># http:<span class="comment">//git.trystack.cn/openstack-dev/devstack</span></span></span>
<span class="line">$ git clone https:<span class="comment">//github.com/openstack/trove-integration.git</span></span>
<span class="line">$ cd trove-integration/scripts/</span>
</pre></td></tr></table></figure> 

编辑`redstack.rc`可以进行一些配置

打开neutron
<figure class="highlight ini"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="setting">NEUTRON_DEFAULT=<span class="value"><span class="keyword">true</span></span></span></span>
<span class="line"><span class="comment"># NEUTRON_DEFAULT=false</span></span>
</pre></td></tr></table></figure>

改改admin的密码
<figure class="highlight fix"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="attribute">ADMIN_PASSWORD</span>=<span class="string">admin</span></span>
</pre></td></tr></table></figure>

git.openstack 慢可以改成国内的
<figure class="highlight ruby"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># GIT_BASE=$&#123;GIT_BASE:-git://git.openstack.org&#125;</span></span>
<span class="line"><span class="constant">GIT_BASE</span>=<span class="variable">$&#123;</span><span class="constant">GIT_BASE</span><span class="symbol">:-http</span><span class="symbol">://git</span>.trystack.cn&#125;</span>
</pre></td></tr></table></figure>

编辑`localrc.rc`
<figure class="highlight ruby"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="constant">GIT_BASE</span>=<span class="variable">$&#123;</span><span class="constant">GIT_BASE</span><span class="symbol">:-http</span><span class="symbol">://git</span>.trystack.cn&#125;</span>
<span class="line"><span class="constant">NOVNC_REPO</span>=<span class="variable">$&#123;</span><span class="constant">NOVNC_REPO</span><span class="symbol">:-http</span><span class="symbol">://git</span>.trystack.cn/kanaka/noVNC.git&#125;</span>
</pre></td></tr></table></figure>

安装
 <figure class="highlight cmake"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ ./redstack <span class="keyword">install</span></span>
</pre></td></tr></table></figure> 

可以设置变量`DEVSTACK_BRANCH`来指定安装分支，`stable/kilo`，默认`master`，接下来坐等安装完成

创建一个trove mysql image
<figure class="highlight crmsh"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">$ ./redstack kick-<span class="literal">start</span> mysql</span>
</pre></td></tr></table></figure>

若上传镜像失败，可以手动上传，镜像路径在`~/images`，参考`http://docs.openstack.org/mitaka/zh_CN/install-guide-rdo/trove-verify.html`

## [](https://ly798.github.io/2016/11/24/devstack-trove/#u53C2_u8003 "参考")参考

[http://docs.openstack.org/developer/trove/dev/install.html](http://docs.openstack.org/developer/trove/dev/install.html)
[http://docs.openstack.org/mitaka/zh_CN/install-guide-rdo/trove-verify.html](http://docs.openstack.org/mitaka/zh_CN/install-guide-rdo/trove-verify.html)
 --></p>
