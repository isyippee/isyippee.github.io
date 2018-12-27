---
title: trove image
tags: [openstack trove]
date: 2016-11-24 10:23:24
---

## [](https://ly798.github.io/2016/11/24/trove-image/#u7248_u672C_u4FE1_u606F "版本信息")版本信息

image系统：centos7

mysql官方版本，mysql-community, 5.5和5.6
 <!-- more --> 

## [](https://ly798.github.io/2016/11/24/trove-image/#u955C_u50CF_u5236_u4F5C "镜像制作")镜像制作

### [](https://ly798.github.io/2016/11/24/trove-image/#u57FA_u672C_u73AF_u5883 "基本环境")基本环境

与基本的镜像一样，cloud-init等。

另外为了调试方便，可以将公钥提前放到镜像中，若出现问题可以登陆实例查询问题。

### [](https://ly798.github.io/2016/11/24/trove-image/#Trove_u76F8_u5173 "Trove相关")Trove相关

*   在实例中需要运行trove-guestagent服务, 则需要在镜像中配置与stack环境一致的rdo源，安装openstack-trove-guestagent, 具体有：
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum -y install openstack-trove-guestagent python-troveclient python-netifaces pexpect python-oslo-serialization</span>
</pre></td></tr></table></figure>*   trove 启动instance时会将`guest_info`、`trove-guestagent.conf`注入到实例，而注入的path是`/etc/`下，需要与trove-guesagent的systemd服务文件对应，添加软链接或者修改服务文件。
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">ln -s /etc/trove-guestagent.conf /etc/trove/trove-guestagent.conf</span>
</pre></td></tr></table></figure>*   将`openstack-trove-guestagent.service`设置开机启动：
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">systemctl enable openstack-trove-guestagent.service &#10;systemctl start openstack-trove-guestagent.service</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/11/24/trove-image/#Mysql__u76F8_u5173 "Mysql 相关")Mysql 相关

*   安装官方的mysql，配置源和选择版本：
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">wget http://repo.mysql.com/mysql-community-release-el7-5.noarch.rpm &#10;rpm -ivh mysql-community-release-el7-5.noarch.rpm</span>
</pre></td></tr></table></figure>
    编辑`/etc/yum.repos.d/mysql-community.repo`选择相应版本的mysql源，安装：
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum -y install mysql-server</span>
</pre></td></tr></table></figure> 

进行相应配置：
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">mkdir -p /etc/mysql &#10;chown mysql:mysql -R /etc/mysql &#10;mkdir -p /etc/my.cnf.d/conf.d &#10;ln -s /etc/my.cnf.d/conf.d /etc/mysql &#10;ln -s /etc/my.cnf /etc/mysql/my.cnf &#10;&#10;if [ ! -d /var/run/mysqld ]; then&#10; mkdir -p /var/run/mysqld&#10; chown mysql. /var/run/mysqld&#10;fi</span>
</pre></td></tr></table></figure>

*   安装备份工具`percona-xtrabackup` <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum -y install http://www.percona.com/downloads/percona-release/redhat/0.1-3/percona-release-0.1-3.noarch.rpm &#10;echo exclude=Percona-SQL* Percona-Server* Percona-XtraDB-Cluster* &#62;&#62; /etc/yum.conf &#10;yum -y install percona-xtrabackup</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2016/11/24/trove-image/#u5236_u4F5C_u5DE5_u5177 "制作工具")制作工具

一个自动构建工具:
[https://github.com/denismakogon/trove-guest-image-elements](https://github.com/denismakogon/trove-guest-image-elements)

进行部分的修改来适合juno版本mysql镜像制作：
[https://github.com/ly798/trove-guest-image-elements/tree/juno-mysql](https://github.com/ly798/trove-guest-image-elements/tree/juno-mysql)
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">./create_trove_image.sh -d centos -s mysql -i CentOS-7-x86_64-GenericCloud.qcow2</span>
</pre></td></tr></table></figure>
