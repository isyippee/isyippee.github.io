---
title: Radosgw 多 zone 部署以及 radosgw-agent 同步
tags: [ceph]
date: 2016-01-18 10:39:02
---

## [](https://ly798.github.io/2016/01/18/Radosgw-%E5%A4%9A-zone-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/#u4ECB_u7ECD "介绍")介绍

部署一个 ceph 集群，创建一个 federated radosgw，包含一个 region，两个 zone，
分别对应节点 cd-1 上的 Master rgw 和节点 cd-2 上的 Secondary rgw。
 <!-- more --> 

## [](https://ly798.github.io/2016/01/18/Radosgw-%E5%A4%9A-zone-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/#u90E8_u7F72 "部署")部署

前提已经部署了 ceph 集群，并分别在两个节点上 部署了 rgw。

1.  修改部署 rgw 是生成的 region infile
cd.json
 <figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">&#123; "name": "cd",</span>
<span class="line"> "api_name": "cd",</span>
<span class="line"> "is_master": "true",</span>
<span class="line"> "endpoints": [</span>
<span class="line"> "http:\/\/cd-1:80\/"],</span>
<span class="line"> "master_zone": "cd-center1",</span>
<span class="line"> "zones": [</span>
<span class="line"> &#123; "name": "cd-center1",</span>
<span class="line"> "endpoints": [</span>
<span class="line"> "http:\/\/cd-1:80\/"],</span>
<span class="line"> "log_meta": "true",</span>
<span class="line"> "log_data": "true"&#125;,</span>
<span class="line"> &#123; "name": "cd-center2",</span>
<span class="line"> "endpoints": [</span>
<span class="line"> "http:\/\/cd-2:80\/"],</span>
<span class="line"> "log_meta": "true",</span>
<span class="line"> "log_data": "true"&#125;],</span>
<span class="line"> "placement_targets": [</span>
<span class="line"> &#123;</span>
<span class="line"> "name": "default-placement",</span>
<span class="line"> "tags": []</span>
<span class="line"> &#125;</span>
<span class="line"> ],</span>
<span class="line"> "default_placement": "default-placement",</span>
<span class="line"> "hostnames": ["cd-1"]&#125;,</span>
</pre></td></tr></table></figure>2.  配置
节点cd-1
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># scp cd-center1.json cd-2:/etc/ceph/</span></span>
<span class="line"><span class="comment"># radosgw-admin --name client.radosgw.cd-center1-cd-1 region set --infile cd-v2.json</span></span>
<span class="line"><span class="comment"># radosgw-admin zone set --rgw-zone=cd-center2 --infile cd-center2.json --name client.radosgw.cd-center1-cd-1</span></span>
<span class="line"><span class="comment"># rados -p .us.rgw.root rm region_info.default</span></span>
<span class="line"><span class="comment"># radosgw-admin --name client.radosgw.cd-center1-cd-1 region default --rgw-region=cd</span></span>
<span class="line"><span class="comment"># radosgw-admin regionmap update --name client.radosgw.cd-center1-cd-1</span></span>
</pre></td></tr></table></figure>
     节点cd-2
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># scp /etc/ceph/cd-center2.json cd-1:/etc/ceph/</span></span>
<span class="line"><span class="comment"># radosgw-admin region set --infile cd-v2.json --name client.radosgw.cd-center2-cd-2</span></span>
<span class="line"><span class="comment"># radosgw-admin zone set --rgw-zone=cd-center1 --infile cd-center1.json --name client.radosgw.cd-center2-cd-2</span></span>
<span class="line"><span class="comment"># rados -p .us.rgw.root rm region_info.default</span></span>
<span class="line"><span class="comment"># radosgw-admin region default --rgw-region=cd --name client.radosgw.cd-center2-cd-2</span></span>
<span class="line"><span class="comment"># radosgw-admin regionmap update --name client.radosgw.cd-center2-cd-2</span></span>
</pre></td></tr></table></figure>3.  重启 ceph-radosgw 服务
cd-1
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># /etc/init.d/ceph-radosgw restart</span></span>
<span class="line"><span class="comment"># ssh cd-2 "/etc/init.d/ceph-radosgw restart"</span></span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2016/01/18/Radosgw-%E5%A4%9A-zone-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/#u914D_u7F6E_radosgw-agent "配置 radosgw-agent")配置 radosgw-agent

radosgw-agent 的默认配置文件是 /etc/ceph/radosgw-agent/default.conf

新建文件/etc/ceph/radosgw-agent/default.conf，在节点 cd-1 上
<figure class="highlight less"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line"><span class="attribute">src_access_key</span>: NDOO5JZAAM2Z6WA9FIXG</span>
<span class="line"><span class="attribute">src_secret_key</span>: <span class="number">4</span>rWHptoevPLPlOwDITljxPuJTqoMQCMM9ICBjGgv</span>
<span class="line">#<span class="attribute">source</span>: <span class="attribute">http</span>:<span class="comment">//cd-1:80</span></span>
<span class="line">#<span class="attribute">src_zone</span>: cd-center1</span>
<span class="line"><span class="attribute">dest_access_key</span>: FLDVEMPT8PKY1F2UD3F7</span>
<span class="line"><span class="attribute">dest_secret_key</span>: <span class="number">8</span>HnGUZIveuHAWePuGXEDXYTR985m7voSUuQDGPkx</span>
<span class="line"><span class="attribute">destination</span>: <span class="attribute">http</span>:<span class="comment">//cd-2:80</span></span>
<span class="line"><span class="attribute">log_file</span>: /var/log/radosgw/radosgw-sync-cd-center1-center2.log</span>
<span class="line">#<span class="attribute">verbose</span>: True</span>
<span class="line">#<span class="attribute">versioned</span>: True</span>
</pre></td></tr></table></figure>

拷贝到节点 cd-2
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># scp /etc/ceph/radosgw-agent/default.conf cd-2:/etc/ceph/radosgw-agent/</span></span>
</pre></td></tr></table></figure>

启动服务
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># mkdir /var/run/ceph/radosgw-agent/</span></span>
<span class="line"><span class="comment"># ssh cd-2 "mkdir /var/run/ceph/radosgw-agent/"</span></span>
<span class="line"><span class="comment"># /etc/init.d/ceph-radosgw start</span></span>
<span class="line"><span class="comment"># ssh cd-2 "/etc/init.d/ceph-radosgw start"</span></span>
</pre></td></tr></table></figure>

可以查看同步日志
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># tailf /var/log/radosgw/radosgw-sync-cd-center1-center2.log</span></span>
</pre></td></tr></table></figure>
