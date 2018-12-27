---
title: 小米mini 科学上网
tags: [linux]
date: 2016-01-02 10:38:14
---

## [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#u4ECB_u7ECD "介绍")介绍

入手个小米mini，正好也有一个 HongKong vps，采用 mini+PandoraBox+ss 设置路由器智能上外网，实现国内外网络连接分流，即保证速度，也保证 vps 流量消耗。
 <!-- more --> 

## [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#u5237_u5165_u7CFB_u7EDF "刷入系统")刷入系统

1.  [http://www1.miwifi.com/miwifi_download.html](http://www1.miwifi.com/miwifi_download.html) 下载对应的最新的 mini 开发版本的ROM，通过 web 界面刷入开发版本的 ROM
2.  [http://www1.miwifi.com/miwifi_open.html](http://www1.miwifi.com/miwifi_open.html) 开启ssh，打开22端口，获得 root 密码
3.  通过 ssh 工具（windows putty）进入 mini 系统
4.  [PandoraBox R2 14.09 / LuCI Trunk (0.12+svn-r1024)](http://downloads.openwrt.org.cn/PandoraBox/Xiaomi-Mini-R1CM/stable/PandoraBox-ralink-mt7620-xiaomi-mini-squashfs-sysupgrade-r1024-20150608.bin) 下载到 mini 系统 /tmp 下，下载该版本是为了避免诸多软件源问题
5.  进入 mini 系统，开始刷入 PandoraBox 命令 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">mtd -r write /tmp/PandoraBox-ralink-mt7620-xiaomi-mini-squashfs-sysupgrade-r1024-<span class="number">20150608</span>.bin firmwarei</span>
</pre></td></tr></table></figure>6.  可以登入系统 PandoraBox 系统了，默认用户 root，密码 admin 

## [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#u914D_u7F6E_u7CFB_u7EDF_u548C_u670D_u52A1 "配置系统和服务")配置系统和服务

### [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#u8DEF_u7531_u5668_u7CFB_u7EDF_u57FA_u672C_u73AF_u5883 "路由器系统基本环境")路由器系统基本环境

1.  修改系统密码，也是 web 登陆密码，`passwd`
2.  默认ip是192.168.1.1，为避免ip与家庭上级路由器冲突，拔掉路由器的网线，并登陆修改 LAN ip
3.  插网线保证路由器能够上网，修改软件源为：(可在web界面设置)<figure class="highlight groovy"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line">dest root /</span>
<span class="line">dest ram /tmp</span>
<span class="line">lists_dir ext <span class="regexp">/var/</span>opkg-lists</span>
<span class="line">option overlay_root /overlay</span>
<span class="line">src<span class="regexp">/gz 14.09_base http:/</span><span class="regexp">/downloads.pandorabox.org.cn/</span>pandorabox<span class="regexp">/ralink/</span>mt7621<span class="regexp">/packages/</span>base</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#vps__u4E0A_u914D_u7F6E_shadowsocks-libev__u670D_u52A1_u7AEF "vps 上配置 shadowsocks-libev 服务端")vps 上配置 shadowsocks-libev 服务端

1.  安装编译环境
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">yum install -y gcc automake autoconf libtool make build-essential autoconf libtool curl curl-devel unzip zlib-devel openssl-devel perl perl-devel cpio expat-devel gettext-devel</span>
</pre></td></tr></table></figure>2.  获取源码
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">wget https://github.com/shadowsocks/shadowsocks-libev/archive/master.zip</span>
</pre></td></tr></table></figure>3.  编译安装
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line">unzip master.zip</span>
<span class="line"><span class="built_in">cd</span> shadowsocks-libev-master/</span>
<span class="line">./autogen.sh</span>
<span class="line">./configure --prefix=/usr &amp;&amp; make./configure --prefix=/usr &amp;&amp; make./configure --prefix=/usr &amp;&amp; make</span>
<span class="line">make install</span>
</pre></td></tr></table></figure>4.  配置
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">mkdir -p /etc/shadowsocks-libev</span>
<span class="line">cp ./rpm/SOURCES/etc/init.d/shadowsocks-libev /etc/init.d/shadowsocks-libev</span>
<span class="line">cp ./debian/config.json /etc/shadowsocks-libev/config.json</span>
<span class="line">chmod +x /etc/init.d/shadowsocks-libev</span>
<span class="line">vim /etc/shadowsocks-libev/config.json <span class="comment">#ss 配置移步[https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)</span></span>
<span class="line">service shadowsocks-libev start</span>
<span class="line">service shadowsocks-libev <span class="built_in">enable</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#PandoraBox__u4E0A_u914D_u7F6E_shadowsocks-libev__u5BA2_u6237_u7AEF "PandoraBox 上配置 shadowsocks-libev 客户端")PandoraBox 上配置 shadowsocks-libev 客户端

1.  安装
shadowsocks-libev，该系统软件源中已有此软件，安装即可，另外安装 wget、curl
2.  配置 dnsmasq 转发国内域名
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">mkdir /etc/dnsmasq.d</span>
<span class="line"><span class="built_in">echo</span> <span class="string">"conf-dir=/etc/dnsmasq.d"</span> &gt;&gt; /etc/dnsmasq.conf</span>
<span class="line"><span class="built_in">cd</span> /etc/dnsmasq.d</span>
<span class="line"><span class="comment"># 感谢该作者提供 dnsmasq-china-list，若使用 wget 不能下载，可以暂时自行下载在上传</span></span>
<span class="line">wget -<span class="number">4</span> --no-check-certificate -O /etc/dnsmasq.d/accelerated-domains.china.conf https://github.com/felixonmars/dnsmasq-china-list/raw/master/accelerated-domains.china.conf</span>
<span class="line">wget -<span class="number">4</span> --no-check-certificate -O /etc/dnsmasq.d/bogus-nxdomain.china.conf https://github.com/felixonmars/dnsmasq-china-list/raw/master/accelerated-domains.china.conf</span>
<span class="line">/etc/dnsmasq.d<span class="comment"># echo "server=/#/127.0.0.1#3210" &gt; gfwlist.conf #其他的域名通过ss</span></span>
</pre></td></tr></table></figure>3.  修改启动脚本
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">vi /etc/init.d/shadowsocks</span>
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
<span class="line">25</span>
<span class="line">26</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment">#!/bin/sh /etc/rc.common</span></span>
<span class="line"><span class="comment"># Copyright (C) 2006-2011 OpenWrt.org</span></span>
<span class="line"></span>
<span class="line">START=<span class="number">95</span></span>
<span class="line"></span>
<span class="line">SERVICE_USE_PID=<span class="number">1</span></span>
<span class="line">SERVICE_WRITE_PID=<span class="number">1</span></span>
<span class="line">SERVICE_DAEMONIZE=<span class="number">1</span></span>
<span class="line"></span>
<span class="line"><span class="function"><span class="title">start</span></span>() &#123;</span>
<span class="line"> sed -i <span class="string">'s/^#conf-dir=\/etc\/dnsmasq.d/conf-dir=\/etc\/dnsmasq.d/'</span> /etc/dnsmasq.conf</span>
<span class="line"> /etc/init.d/dnsmasq restart</span>
<span class="line"></span>
<span class="line"> service_start /usr/bin/ss-redir -b <span class="number">0.0</span>.<span class="number">0.0</span> -c /etc/shadowsocks.json <span class="operator">-f</span> /var/run/shadowsocks.pid</span>
<span class="line"> service_start /usr/bin/ss-tunnel -b <span class="number">0.0</span>.<span class="number">0.0</span> -c /etc/shadowsocks.json <span class="operator">-l</span> <span class="number">3210</span> -L <span class="number">8.8</span>.<span class="number">8.8</span>:<span class="number">53</span> -u </span>
<span class="line"> /usr/bin/shadowsocks-firewall</span>
<span class="line">&#125;</span>
<span class="line"></span>
<span class="line"><span class="function"><span class="title">stop</span></span>() &#123;</span>
<span class="line"> sed -i <span class="string">'s/^conf-dir=\/etc\/dnsmasq.d/#conf-dir=\/etc\/dnsmasq.d/'</span> /etc/dnsmasq.conf</span>
<span class="line"> /etc/init.d/dnsmasq restart</span>
<span class="line"></span>
<span class="line"> service_stop /usr/bin/ss-redir</span>
<span class="line"> service_stop /usr/bin/ss-tunnel</span>
<span class="line"> /etc/init.d/firewall restart</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>4.  配置iptables防火墙转发IP和端口
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="built_in">cd</span> /usr/bin</span>
<span class="line">touch shadowsocks-firewall</span>
<span class="line">chmod +x shadowsocks-firewall</span>
<span class="line">vi shadowsocks-firewall</span>
</pre></td></tr></table></figure>
     注意下面的 1.0.9.8 修改为vps 的 ip
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
<span class="line">25</span>
<span class="line">26</span>
<span class="line">27</span>
<span class="line">28</span>
<span class="line">29</span>
<span class="line">30</span>
<span class="line">31</span>
<span class="line">32</span>
<span class="line">33</span>
<span class="line">34</span>
<span class="line">35</span>
<span class="line">36</span>
<span class="line">37</span>
<span class="line">38</span>
<span class="line">39</span>
<span class="line">40</span>
<span class="line">41</span>
<span class="line">42</span>
<span class="line">43</span>
<span class="line">44</span>
<span class="line">45</span>
<span class="line">46</span>
<span class="line">47</span>
<span class="line">48</span>
<span class="line">49</span>
<span class="line">50</span>
<span class="line">51</span>
<span class="line">52</span>
<span class="line">53</span>
<span class="line">54</span>
<span class="line">55</span>
<span class="line">56</span>
<span class="line">57</span>
<span class="line">58</span>
<span class="line">59</span>
<span class="line">60</span>
<span class="line">61</span>
<span class="line">62</span>
<span class="line">63</span>
<span class="line">64</span>
<span class="line">65</span>
<span class="line">66</span>
<span class="line">67</span>
<span class="line">68</span>
<span class="line">69</span>
<span class="line">70</span>
<span class="line">71</span>
<span class="line">72</span>
<span class="line">73</span>
<span class="line">74</span>
<span class="line">75</span>
<span class="line">76</span>
<span class="line">77</span>
<span class="line">78</span>
<span class="line">79</span>
<span class="line">80</span>
<span class="line">81</span>
<span class="line">82</span>
<span class="line">83</span>
<span class="line">84</span>
<span class="line">85</span>
<span class="line">86</span>
<span class="line">87</span>
<span class="line">88</span>
<span class="line">89</span>
<span class="line">90</span>
</pre></td><td class="code"><pre><span class="line"><span class="shebang">#!/bin/sh</span>
<span class="line"></span></span>
<span class="line"><span class="comment"># Author: https://github.com/softwaredownload/openwrt-fanqiang</span></span>
<span class="line"><span class="comment"># Date: 2015-12-23</span></span>
<span class="line"></span>
<span class="line"><span class="comment">#create a new chain named SHADOWSOCKS</span></span>
<span class="line">iptables -t nat -N SHADOWSOCKS</span>
<span class="line">iptables -t nat -N SHADOWSOCKS_WHITELIST</span>
<span class="line"></span>
<span class="line"><span class="comment"># Ignore your shadowsocks server's addresses</span></span>
<span class="line"><span class="comment"># It's very IMPORTANT, just be careful.</span></span>
<span class="line"></span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">1.0</span>.<span class="number">9.8</span> -j RETURN</span>
<span class="line"></span>
<span class="line"><span class="comment">#for hulu.com</span></span>
<span class="line">iptables -t nat -A SHADOWSOCKS -p tcp --dport <span class="number">1935</span> -j REDIRECT --to-ports <span class="number">7654</span></span>
<span class="line">iptables -t nat -A SHADOWSOCKS -p udp --dport <span class="number">1935</span> -j REDIRECT --to-ports <span class="number">7654</span></span>
<span class="line"></span>
<span class="line"><span class="comment"># Ignore LANs IP address</span></span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">0.0</span>.<span class="number">0.0</span>/<span class="number">8</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">10.0</span>.<span class="number">0.0</span>/<span class="number">8</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">127.0</span>.<span class="number">0.0</span>/<span class="number">8</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">169.254</span>.<span class="number">0.0</span>/<span class="number">16</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">172.16</span>.<span class="number">0.0</span>/<span class="number">12</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">192.168</span>.<span class="number">0.0</span>/<span class="number">16</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">224.0</span>.<span class="number">0.0</span>/<span class="number">4</span> -j RETURN</span>
<span class="line">iptables -t nat -A SHADOWSOCKS <span class="operator">-d</span> <span class="number">240.0</span>.<span class="number">0.0</span>/<span class="number">4</span> -j RETURN</span>
<span class="line"></span>
<span class="line"><span class="comment"># Check whitelist</span></span>
<span class="line">iptables -t nat -A SHADOWSOCKS -j SHADOWSOCKS_WHITELIST</span>
<span class="line">iptables -t nat -A SHADOWSOCKS -m mark --mark <span class="number">1</span> -j RETURN</span>
<span class="line"></span>
<span class="line"><span class="comment"># Anything else should be redirected to shadowsocks's local port</span></span>
<span class="line">iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-ports <span class="number">7654</span></span>
<span class="line"><span class="comment"># Apply the rules</span></span>
<span class="line">iptables -t nat -A PREROUTING -p tcp -j SHADOWSOCKS</span>
<span class="line"></span>
<span class="line"><span class="comment"># Ignore China IP address</span></span>
<span class="line"><span class="keyword">for</span> white_ip <span class="keyword">in</span> `cat /etc/chinadns_chnroute.txt`;</span>
<span class="line"><span class="keyword">do</span></span>
<span class="line"> iptables -t nat -A SHADOWSOCKS_WHITELIST <span class="operator">-d</span> <span class="string">"<span class="variable">$&#123;white_ip&#125;</span>"</span> -j MARK --set-mark <span class="number">1</span></span>
<span class="line"><span class="keyword">done</span></span>
<span class="line"></span>
<span class="line"><span class="comment"># Ignore Asia IP address</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 1.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 14.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 27.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 36.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 39.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 42.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 49.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 58.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 59.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 60.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 61.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 101.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 103.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 106.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 110.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 111.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 112.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 113.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 114.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 115.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 116.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 117.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 118.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 119.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 120.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 121.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 122.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 123.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 124.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 125.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 126.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 169.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 175.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 180.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 182.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 183.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 202.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 203.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 210.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 211.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 218.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 219.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 220.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 221.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 222.0.0.0/8 -j MARK --set-mark 1</span></span>
<span class="line"><span class="comment">#iptables -t nat -A SHADOWSOCKS_WHITELIST -d 223.0.0.0/8 -j MARK --set-mark 1</span></span>
</pre></td></tr></table></figure> 

*   启动服务<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">/etc/init.d/shadowsocks start</span>
<span class="line">/etc/init.d/shadowsocks <span class="built_in">enable</span></span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2016/01/02/%E5%B0%8F%E7%B1%B3mini-%E7%A7%91%E5%AD%A6%E4%B8%8A%E7%BD%91/#u6700_u540E "最后")最后

连接路由器的所有终端设备都可以智能的科学上网，国内外分流。
