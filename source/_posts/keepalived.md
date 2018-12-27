---
title: keepalived
tags: [linux]
date: 2016-04-15 10:32:11
---

## [](https://ly798.github.io/2016/04/15/keepalived/#u4ECB_u7ECD "介绍")介绍

keepalived 用于服务的高可用,解决单点故障。基于网络的 VRRP 协议。
 <!-- more --> 

## [](https://ly798.github.io/2016/04/15/keepalived/#demo "demo")demo

### [](https://ly798.github.io/2016/04/15/keepalived/#env "env")env

三台虚拟机，分别有2张网卡，对应地址如下：

ha1

*   eth1: 172.16.101.4/24
ha1
*   eth1: 172.16.101.5/24
ha1
*   eth1: 172.16.101.6/24

选择一个 vip，172.16.101.10/24

### [](https://ly798.github.io/2016/04/15/keepalived/#setting "setting")setting

#### [](https://ly798.github.io/2016/04/15/keepalived/#ha1 "ha1")ha1

配置文件 /etc/keepalived/keepalived.conf
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
<span class="line">23</span>
<span class="line">24</span>
</pre></td><td class="code"><pre><span class="line">vrrp_script check_running &#123;</span>
<span class="line"> script <span class="string">"/etc/keepalived/check_http.sh"</span> <span class="preprocessor"># 执行检查的甲本</span></span>
<span class="line"> interval <span class="number">2</span> </span>
<span class="line"> weight -<span class="number">20</span> <span class="preprocessor"># priority 减少<span class="number">20</span></span></span>
<span class="line">&#125;</span>
<span class="line"></span>
<span class="line">vrrp_instance http &#123;</span>
<span class="line"> state BACKUP</span>
<span class="line"> priority <span class="number">100</span> <span class="preprocessor"># 可理解为权重，高的会被选为master，<span class="number">1</span>-<span class="number">255</span></span></span>
<span class="line"> interface eth1 <span class="preprocessor"># 用来发送VRRP的网卡</span></span>
<span class="line"> virtual_router_id <span class="number">47</span> <span class="preprocessor"># 用来区分多个instance的VRRP组播，<span class="number">0</span>-<span class="number">255</span>，同个集群中的主备这里设置成一样</span></span>
<span class="line"> advert_int <span class="number">3</span> <span class="preprocessor"># 发送VRRP的时间间隔，即进行一次检查</span></span>
<span class="line"> authentication &#123; <span class="preprocessor"># 认证</span></span>
<span class="line"> auth_type PASS</span>
<span class="line"> auth_pass <span class="number">1234</span></span>
<span class="line"> &#125;</span>
<span class="line"> nopreempt <span class="preprocessor"># 设置当故障了重启之后不抢占现在master</span></span>
<span class="line"> virtual_ipaddress &#123; <span class="preprocessor"># VIP</span></span>
<span class="line"> <span class="number">172.16</span><span class="number">.101</span><span class="number">.10</span>/<span class="number">24</span></span>
<span class="line"> &#125;</span>
<span class="line"> virtual_routes&#123;&#125; <span class="preprocessor"># 当IP漂过来之后需要添加的路由</span></span>
<span class="line"> track_script &#123;</span>
<span class="line"> check_running</span>
<span class="line"> &#125;</span>
</pre></td></tr></table></figure>

该配置文件有多个配置区域

*   `global_defs`、
主要配置故障时通知信息，可忽略
*   `static_ipaddress`、
本节点的ip配置，服务器一般都会有配置，可忽略
*   `static_routes`、
本节点的route配置，服务器一般都会有配置，可忽略
*   `vrrp_script`、
健康检查
*   `vrrp_instance`、
配置vip机器属性
*   `vrrp_sync_group`
配置vrrp_instance组，当某个vrrp_instance切换时，组内都会切换 

检查脚本/etc/keepalived/check_http.sh
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line"><span class="shebang">#!/usr/bin/env bash</span></span>
<span class="line">count=`netstat -tnlp | grep <span class="number">80</span> | wc <span class="operator">-l</span>`</span>
<span class="line"><span class="keyword">if</span> [ <span class="variable">$count</span> <span class="operator">-gt</span> <span class="number">0</span> ]</span>
<span class="line"><span class="keyword">then</span></span>
<span class="line"> <span class="built_in">exit</span> <span class="number">0</span></span>
<span class="line"><span class="keyword">else</span></span>
<span class="line"> <span class="comment"># 因为我们设置了 nopreempt 不抢占，即使检查到 80 端口不在，也不会抢占为master，这里杀掉keepalived即可。</span></span>
<span class="line"> `systemctl stop keepalived` </span>
<span class="line"> <span class="built_in">exit</span> <span class="number">1</span></span>
<span class="line"><span class="keyword">fi</span></span>
</pre></td></tr></table></figure>
 <figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># systemctl start keepalived</span></span>
</pre></td></tr></table></figure> 

#### [](https://ly798.github.io/2016/04/15/keepalived/#ha2_u3001ha3 "ha2、ha3")ha2、ha3

与 ha1 不同的是 /etc/keepalived/keepalived.conf
<figure class="highlight lasso"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">vrrp_instance http &#123;</span>
<span class="line"> <span class="attribute">...</span></span>
<span class="line"> priority <span class="number">90</span></span>
<span class="line"> <span class="attribute">...</span></span>
</pre></td></tr></table></figure>

### [](https://ly798.github.io/2016/04/15/keepalived/#result "result")result

查看三台系统的 ip，可发现其中ha1 eth1 绑定了 vip
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># ip a</span></span>
<span class="line">eth1: &lt;BROADCAST,MULTICAST,UP,LOWER_UP&gt; mtu <span class="number">1500</span> qdisc pfifo_fast state UP qlen <span class="number">1000</span></span>
<span class="line"> link/ether <span class="number">52</span>:<span class="number">54</span>:<span class="number">00</span>:<span class="number">6f</span>:<span class="number">0</span>b:<span class="number">78</span> brd ff:ff:ff:ff:ff:ff</span>
<span class="line"> inet <span class="number">172.16</span><span class="number">.101</span><span class="number">.4</span>/<span class="number">24</span> brd <span class="number">172.16</span><span class="number">.101</span><span class="number">.255</span> scope global eth1</span>
<span class="line"> valid_lft forever preferred_lft forever</span>
<span class="line"> inet <span class="number">172.16</span><span class="number">.101</span><span class="number">.10</span>/<span class="number">24</span> scope global secondary eth1</span>
<span class="line"> valid_lft forever preferred_lft forever</span>
<span class="line"> inet6 fe80::<span class="number">5054</span>:ff:fe6f:b78/<span class="number">64</span> scope link </span>
<span class="line"> valid_lft forever preferred_lft forever</span>
</pre></td></tr></table></figure>

停掉 httpd
<figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># systemctl stop httpd</span></span>
</pre></td></tr></table></figure>

在另外两台有一台被选举为 master，绑定了 vip
