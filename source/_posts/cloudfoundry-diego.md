---
title: cloudfoundry & diego
tags: [cloudfoundry]
date: 2017-12-01 10:10:05
---

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#introduction "introduction")introduction

cloudfoundry 现已支持 diego，目前官方有两种部署方式

*   cf-deployment (较新)
*   cf-release (将被弃用，不过相对充分验证&lt;包括 openstack&gt;)

<!-- more -->
部署过程太复杂，翻开官网一看，我靠

![](panda_01.png)

撸起袖子开搞

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u73AF_u5883_u63CF_u8FF0 "环境描述")环境描述

*   opnstack: ocata
*   cf 版本: 截至目前的 release 版本 v280([https://github.com/cloudfoundry/cf-release/releases](https://github.com/cloudfoundry/cf-release/releases))

stemcell 和其他需要的 release 使用其推荐的兼容版本，如下:
 <table> <thead> <tr> <th>name</th> <th>version</th> <th>url</th> </tr> </thead> <tbody> <tr> <td>cf release</td> <td>v280</td> <td>[https://bosh.io/d/github.com/cloudfoundry/cf-release?v=280](https://bosh.io/d/github.com/cloudfoundry/cf-release?v=280)</td> </tr> <tr> <td>Diego release</td> <td>v1.30.0</td> <td>[https://bosh.io/d/github.com/cloudfoundry/diego-release?v=1.30.0](https://bosh.io/d/github.com/cloudfoundry/diego-release?v=1.30.0)</td> </tr> <tr> <td>Garden-Runc release</td> <td>v1.9.6</td> <td>[https://bosh.io/d/github.com/cloudfoundry/garden-runc-release?v=1.9.6](https://bosh.io/d/github.com/cloudfoundry/garden-runc-release?v=1.9.6)</td> </tr> <tr> <td>cflinuxfs2 release</td> <td>v1.168.0</td> <td>[https://bosh.io/d/github.com/cloudfoundry/cflinuxfs2-release?v=1.168.0](https://bosh.io/d/github.com/cloudfoundry/cflinuxfs2-release?v=1.168.0)</td> </tr> <tr> <td>etcd release</td> <td>v117</td> <td>[https://bosh.io/d/github.com/cloudfoundry-incubator/etcd-release?v=117](https://bosh.io/d/github.com/cloudfoundry-incubator/etcd-release?v=117)</td> </tr> <tr> <td>stemcell</td> <td>3468.5</td> <td>[https://s3.amazonaws.com/bosh-core-stemcells/openstack/bosh-stemcell-3468.5-openstack-kvm-ubuntu-trusty-go_agent-raw.tgz](https://s3.amazonaws.com/bosh-core-stemcells/openstack/bosh-stemcell-3468.5-openstack-kvm-ubuntu-trusty-go_agent-raw.tgz)</td> </tr> </tbody> </table> 

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#openstack__u73AF_u5883_u51C6_u5907 "openstack 环境准备")openstack 环境准备

不具体描述，参考 [http://docs.cloudfoundry.org/deploying/cf-release/openstack/index.html](http://docs.cloudfoundry.org/deploying/cf-release/openstack/index.html)， 主要步骤包括

*   设置安全组
*   设置 DNS(用于解析 cf app)
*   确认设置 flavor 

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u90E8_u7F72_bosh "部署 bosh")部署 bosh

bosh 是一种部署工具，cf 可以通过 bosh 来部署。通常 bosh 包括两部分

*   bosh director(这里是在 openstack 中的一个虚拟机，用来接受 bosh client 的命令，并操作集群，如 cf)
*   bosh client(任意可访问 openstack 的一个机器，安装 bosh cli 工具，后续的部署工作都是在该机器进行) 

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u51C6_u5907_openstack__u73AF_u5883 "准备 openstack 环境")准备 openstack 环境

这里的准备工作不冲突之前的，这是为部署 bosh director 准备，不具体描述，参考 [https://bosh.io/docs/init-openstack-v1.html](https://bosh.io/docs/init-openstack-v1.html)

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u51C6_u5907_bosh_director__u7684_u914D_u7F6E_u6587_u4EF6 "准备 bosh director 的配置文件")准备 bosh director 的配置文件

根据需求修改，如 openstack 环境认证、虚拟机的密码等，参考 [https://gist.github.com/ly798/624c964dffcbd86d4f9fa01a7d5c4dba](https://gist.github.com/ly798/624c964dffcbd86d4f9fa01a7d5c4dba)

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u90E8_u7F72 "部署")部署

安装 bosh-init，参考 [https://bosh.io/docs/install-bosh-init.html](https://bosh.io/docs/install-bosh-init.html)， 用来部署 bosh director

部署 bosh director
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">bosh-init deploy ./bosh.yml</span>
</pre></td></tr></table></figure> 

安装 bosh-cli，参考 [https://bosh.io/docs/bosh-cli.html](https://bosh.io/docs/bosh-cli.html)， 用来操作 bosh director

登陆到 bosh director
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">bosh target 173.81.16.12 #&#35813; ip &#26159;&#22312;&#37197;&#32622;&#25991;&#20214;&#20013;&#37197;&#32622;&#30340; director &#30340; floatingip&#10;bosh vms #&#26597;&#35810;&#38598;&#32676;&#30340; vm&#65292;&#29616;&#22312;&#20026;&#31354;</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u90E8_u7F72_cloud_foundry "部署 cloud foundry")部署 cloud foundry

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u51C6_u5907_cf__u7684_u914D_u7F6E_u6587_u4EF6 "准备 cf 的配置文件")准备 cf 的配置文件

参考 [http://docs.cloudfoundry.org/deploying/cf-release/openstack/cf-stub.html](http://docs.cloudfoundry.org/deploying/cf-release/openstack/cf-stub.html)

cf 的配置文件(暂命令为 cf-deployment.yml)非常庞大，所以使用脚本工具来生成，不过也需要一个基本的 manifest stub 配置文件(咱命令为 cf-stub.yml)

首先获取 cf-release 代码仓库
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">git clone https://github.com/cloudfoundry/cf-release.git&#10;git checkout v258&#10;https://github.com/cloudfoundry/diego-release.git&#10;git checkout v1.30.0</span>
</pre></td></tr></table></figure> 

新建配置文件 cf-stub.yml，参考 [https://gist.github.com/ly798/1a393a7450b54a6e78bf62ba6c00a249](https://gist.github.com/ly798/1a393a7450b54a6e78bf62ba6c00a249)， 根据自己的环境配置，另外由于 bug 需要注意的地方
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">properties:&#10; etcd:&#10; require_ssl: false</span>
</pre></td></tr></table></figure> 

关于上述配置文件中的 cert、key 需要使用仓库中 script 目录下脚本实现，可以参考 [https://github.com/cloudfoundry/cf-release/blob/master/on-tls-certificates.md](https://github.com/cloudfoundry/cf-release/blob/master/on-tls-certificates.md)， 这些脚本有依赖，安装如下(ubuntu 为例):
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">git clone https://github.com/square/certstrap&#10;cd certstrap&#10;./build&#10;sudo cp ./bin/certstrap-v1.1.1-linux-amd64 /usr/bin/certstrap&#10;sudo apt install golang&#10;mkdir ~/go&#10;export GOPATH=~/go</span>
</pre></td></tr></table></figure> 

关于上述配置文件中的 dns 配置暂时使用内部解析的 dns 服务器，可以使用 dnsmasq，将配置文件中的 domain 与 floatingip 进行解析

使用脚本工具生成 cf-deployment.yml
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cd cf-release&#10;./scripts/generate_deployment_manifest openstack ../cf-stub.yml ../diego-release/stubs-for-cf-release/enable_diego_ssh_in_cf.yml &#62; ../cf-deployment.yml</span>
</pre></td></tr></table></figure> 

生成的 cf-deployment.yml 还需要一些修改(由于 bug)
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">uaa:&#10; require_https: false</span>
</pre></td></tr></table></figure> 

替换虚拟机镜像
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">sed -i &#39;s/bosh-openstack-kvm-ubuntu-trusty-go_agent/bosh-openstack-kvm-ubuntu-trusty-go_agent-raw/g&#39; ../cf-deployment.yml</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u90E8_u7F72-1 "部署")部署

参考 [http://docs.cloudfoundry.org/deploying/cf-release/common/deploy.html](http://docs.cloudfoundry.org/deploying/cf-release/common/deploy.html)

上传 cf-release、stemcell
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">bosh upload release ~/cf-release-v280.zip&#10;bosh upload stemcell ~/bosh-stemcell-3468.5-openstack-kvm-ubuntu-trusty-go_agent-raw.tgz</span>
</pre></td></tr></table></figure> 

一切准备就绪，开始 <del>装逼</del> 安装
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">bosh deployment ../cf-deployment.yml&#10;bosh deploy</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u9A8C_u8BC1 "验证")验证

验证 cf 服务信息
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">curl api.INCORRECT-SYSTEM-DOMAIN/info # INCORRECT-SYSTEM-DOMAIN &#27515;&#22312; cf-stub.yml &#20013;&#30340;&#37197;&#32622;</span>
</pre></td></tr></table></figure> 

登陆 cf
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cf login --skip-ssl-validation -a https:/INCORRECT-SYSTEM-DOMAIN -u admin&#10;cf a</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u90E8_u7F72_diego "部署 diego")部署 diego

diego 在 openstack 上配置资料较少，翻遍官网和网络都 emmmm..
![](panghu_01.png)

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u51C6_u5907_u914D_u7F6E_u6587_u4EF6 "准备配置文件")准备配置文件

参考 diego-release 代码仓库 aws 和 bosh-lite 的配置文件整理了一份，具体如下
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cd diego-release&#10;mkdir myop_config&#10;cp manifest-generation/examples/openstack/iaas-settings.yml .&#10;cp manifest-generation/bosh-lite-stubs/property-overrides.yml .&#10;cp ./manifest-generation/bosh-lite-stubs/postgres/diego-sql.yml .&#10;cp ./manifest-generation/bosh-lite-stubs/instance-count-overrides.yml .</span>
</pre></td></tr></table></figure> 

需要根据自己环境配置这四个文件，需要注意

*   property-overrides.yml 中的 cert 等还是需要参考部署 cf 是使用的 cert 等
*   property-overrides.yml 中的 etcd.ssl 相关设置为 false(部署 cf 是也使用了 false)
*   iaas-settings.yml 中 stemcell 使用部署 cf 的即可(raw)
*   diego-sql 的信息取自 cf 配置文件 

生成配置文件 diego-deployment.yml
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cd diego-release&#10;./scripts/generate-deployment-manifest -c ../../cf-deployment.yml \&#10; -i ./myop_config/iaas-settings.yml \&#10; -p ./myop_config/property-overrides.yml \&#10; -n ./myop_config/instance-count-overrides.yml \&#10; -s ./myop_config/diego-sql.yml -r &#62; ./diego-deployment.yml</span>
</pre></td></tr></table></figure> 

生成的 diego.yml 还需要进行修改(遗漏的 uaa.secret)
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">diego.ssh_proxy.uaa_secret: xxx #&#21462;&#33258; cf &#37096;&#32626;&#25991;&#20214;&#20013;&#30340;</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u90E8_u7F72-2 "部署")部署

上传 releases
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">bosh upload release garden-runc-release-v1.10.0.zip&#10;bosh upload release cflinuxfs2-release-v1.170.0.zip&#10;bosh upload release diego-release-v1.30.0.zip&#10;bosh upload release etcd-release-v117.zip</span>
</pre></td></tr></table></figure> 

部署 diego
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">bosh deployment diego-deployment.yml&#10;bosh -n deploy</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#u9A8C_u8BC1-1 "验证")验证

打开 docker flag
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">cf enable-feature-flag diego_docker</span>
</pre></td></tr></table></figure> 

部署一个 docker
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># cf push my-app --docker-image daocloud.io/library/nginx:latest&#10;name requested state instances memory disk urls&#10;my-app started 1/1 1G 1G my-app.test.cf-app.com</span>
</pre></td></tr></table></figure> 

历时两天跳无数个坑，终于是成功了，我只能说
![](dog_01.png)

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#FAQ "FAQ")FAQ

## [](https://ly798.github.io/2017/12/01/cloudfoundry-and-diego/#reference "reference")reference

*   [http://docs.cloudfoundry.org/deploying/cf-release/openstack/index.html](http://docs.cloudfoundry.org/deploying/cf-release/openstack/index.html)
