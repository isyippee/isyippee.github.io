---
title: rgw swift as trove store
tags: [openstack trove]
date: 2016-11-24 10:24:00
---

## [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#u4ECB_u7ECD "介绍")介绍

要使用rgw 的swift接口，只需要创建对应的swift user和key即可。
 <!-- more --> 

## [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#u96C6_u6210keystone "集成keystone")集成keystone

### [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#controller_u8282_u70B9 "controller节点")controller节点

#### [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#nss "nss")nss
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># mkdir -p /var/ceph/nss &#10;# openssl x509 -in /etc/keystone/ssl/certs/ca.pem -pubkey | \&#10;certutil -d /var/ceph/nss -A -n ca -t &#34;TCu,Cu,Tuw&#34;&#10;# openssl x509 -in /etc/keystone/ssl/certs/signing_cert.pem -pubkey | \&#10;certutil -A -d /var/ceph/nss -n signing_cert -t &#34;P,P,P&#34;</span>
</pre></td></tr></table></figure> 

将`/var/ceph/nss`拷贝到rgw节点的`/var/ceph/nss`

#### [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#u6DFB_u52A0swift_endpoint "添加swift endpoint")添加swift endpoint

1.  若有使用openstack swift, 需要先删掉原endpoint，原endpoint如下：
 <figure class="highlight gherkin"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">|<span class="string"> 1d7aca58ca47401bbdefb8efe4b5f65d </span>|<span class="string"> RegionOne </span>|<span class="string"> http://192.168.6.125:8080/v1/AUTH_%(tenant_id)s </span>|<span class="string"> http://192.168.6.125:8080/v1/AUTH_%(tenant_id)s </span>|<span class="string"> http://192.168.6.125:8080/ </span>|<span class="string"> 3a408d0525d546c7aca888b3e4d36833 </span>|</span>
</pre></td></tr></table></figure>2.  添加新的endpoint：
192.168.6.127为rgw地址
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">keystone endpoint-create --region RegionOne \ &#10;--service-id 3a408d0525d546c7aca888b3e4d36833 \ &#10;--publicurl http://192.168.6.127/swift/v1 \ &#10;--internalurl http://192.168.6.127/swift/v1 \&#10;--adminurl http://192.168.6.127/swift/v1</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#rgw__u8282_u70B9 "rgw 节点")rgw 节点

在rgw节点修改配置文件`/etc/ceph/ceph.conf`, 添加如下配置：
 <figure class="highlight sql"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">rgw keystone url = http://192.168.6.125:5000 </span>
<span class="line">rgw keystone admin token = 85cc6b3914c042fe8be37032ff03fa49</span>
<span class="line">rgw keystone accepted roles = Member, _member_, admin, SwiftOperator </span>
<span class="line">rgw keystone token <span class="operator"><span class="keyword">cache</span> <span class="keyword">size</span> = <span class="number">500</span></span>
<span class="line">rgw keystone revocation <span class="built_in">interval</span> = <span class="number">500</span> </span>
<span class="line">rgw s3 auth <span class="keyword">use</span> keystone = <span class="literal">true</span></span>
<span class="line">rgw nss db <span class="keyword">path</span> = /<span class="keyword">var</span>/ceph/nss</span></span>
</pre></td></tr></table></figure> 

重启rgdosgw

## [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#trove__u8C03_u6574 "trove 调整")trove 调整

修改配置文件`/etc/trove/trove.conf`、`/etc/trove/trove-guestmanager.conf`, 注释掉`swift_url`该配置项。

原该项配置value是`http://192.168.6.125:8080/v1/AUTH_`, 正确的地址应该是`http://192.168.6.127/swift/v1`, 即endpoint配置的url。当注释掉该项配置，会从endpoint中查询url

## [](https://ly798.github.io/2016/11/24/rgw-swift-as-trove-store/#u53C2_u8003 "参考")参考

[http://docs.ceph.com/docs/master/radosgw/keystone/](http://docs.ceph.com/docs/master/radosgw/keystone/)
