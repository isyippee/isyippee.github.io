---
title: Radosgw 多 region 部署以及 radosgw-agent 同步
tags: [ceph]
date: 2016-01-26 10:40:00
---

## [](https://ly798.github.io/2016/01/26/Radosgw-%E5%A4%9A-region-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/#u4ECB_u7ECD "介绍")介绍

接着 [Radosgw 多 zone 部署以及 radosgw-agent 同步](http://lyang.top/2016/01/18/Radosgw-%E5%A4%9A-zone-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/) 部署两个 region 的集群。

整体的结构是：

*   cd 作为 Master Region，cd-1 作为该 Region 中的 Master Zone，cd-2 作为该 Region 中的 Secondary Zone
*   bj 作为 Secondary Region，bj-1 作为该 Region 中的 Master Zone，bj-2 作为该 Region 中的 Secondary Zone <!-- more --> 

region 之间的同步，只能是 master region 的 master zone 作为源，secondary region 的 master zone 作为目的。

## [](https://ly798.github.io/2016/01/26/Radosgw-%E5%A4%9A-region-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/#u90E8_u7F72 "部署")部署

前提已经部署了两个单 region、多 zone 的 ceph 集群，并分别在四个节点上 部署了 rgw。

1.  修改新部署的 bj 集群的 region file

     bj.json
 <figure class="highlight json"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">&#123; "<span class="attribute">name</span>": <span class="value"><span class="string">"bj"</span></span>,</span>
<span class="line"> "<span class="attribute">api_name</span>": <span class="value"><span class="string">"bj"</span></span>,</span>
<span class="line"> "<span class="attribute">is_master</span>": <span class="value"><span class="string">"false"</span></span>,</span>
<span class="line"> "<span class="attribute">endpoints</span>": <span class="value">[</span>
<span class="line"> <span class="string">"http:\/\/bj-1:80\/"</span>]</span>,</span>
<span class="line"> "<span class="attribute">master_zone</span>": <span class="value"><span class="string">"bj-center1"</span></span>,</span>
<span class="line"> "<span class="attribute">zones</span>": <span class="value">[</span>
<span class="line"> &#123; "<span class="attribute">name</span>": <span class="value"><span class="string">"bj-center1"</span></span>,</span>
<span class="line"> "<span class="attribute">endpoints</span>": <span class="value">[</span>
<span class="line"> <span class="string">"http:\/\/bj-1:80\/"</span>]</span>,</span>
<span class="line"> "<span class="attribute">log_meta</span>": <span class="value"><span class="string">"true"</span></span>,</span>
<span class="line"> "<span class="attribute">log_data</span>": <span class="value"><span class="string">"true"</span></span>&#125;,</span>
<span class="line"> &#123; "<span class="attribute">name</span>": <span class="value"><span class="string">"bj-center2"</span></span>,</span>
<span class="line"> "<span class="attribute">endpoints</span>": <span class="value">[</span>
<span class="line"> <span class="string">"http:\/\/bj-2:80\/"</span>]</span>,</span>
<span class="line"> "<span class="attribute">log_meta</span>": <span class="value"><span class="string">"true"</span></span>,</span>
<span class="line"> "<span class="attribute">log_data</span>": <span class="value"><span class="string">"true"</span></span>&#125;]</span>,</span>
<span class="line"> "<span class="attribute">placement_targets</span>": <span class="value">[</span>
<span class="line"> &#123;</span>
<span class="line"> "<span class="attribute">name</span>": <span class="value"><span class="string">"default-placement"</span></span>,</span>
<span class="line"> "<span class="attribute">tags</span>": <span class="value">[]</span>
<span class="line"> </span>&#125;</span>
<span class="line"> ]</span>,</span>
</pre></td></tr></table></figure>
     注意该 region 作为 secondary region，`&quot;is_master&quot;: &quot;false&quot;`
2.  配置

     每个节点相互拷贝 region、zone 配置文件，保证每个节点的配置文件如下：
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># ll /etc/ceph/</span></span>
<span class="line">总用量 <span class="number">40</span></span>
<span class="line">-rw-r--r-- <span class="number">1</span> root root <span class="number">793</span> <span class="number">1</span>月 <span class="number">26</span> <span class="number">16</span>:<span class="number">33</span> bj-center1.json</span>
<span class="line">-rw-r--r-- <span class="number">1</span> root root <span class="number">793</span> <span class="number">1</span>月 <span class="number">26</span> <span class="number">16</span>:<span class="number">33</span> bj-center2.json</span>
<span class="line">-rw-r--r-- <span class="number">1</span> root root <span class="number">676</span> <span class="number">1</span>月 <span class="number">26</span> <span class="number">16</span>:<span class="number">33</span> bj.json</span>
<span class="line">-rw-r--r-- <span class="number">1</span> root root <span class="number">793</span> <span class="number">1</span>月 <span class="number">12</span> <span class="number">15</span>:<span class="number">30</span> <span class="built_in">cd</span>-center1.json</span>
<span class="line">-rw-r--r-- <span class="number">1</span> root root <span class="number">793</span> <span class="number">1</span>月 <span class="number">18</span> <span class="number">14</span>:<span class="number">02</span> <span class="built_in">cd</span>-center2.json</span>
<span class="line">-rw-r--r-- <span class="number">1</span> root root <span class="number">682</span> <span class="number">1</span>月 <span class="number">18</span> <span class="number">12</span>:<span class="number">15</span> cd.json</span>
<span class="line">...</span>
</pre></td></tr></table></figure>
     在 secondary region 两个节点上创建 master region、master zone，并更新 regionmap
bj-1
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># cd /etc/ceph</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center1-bj-1 region set --infile cd.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center1-bj-1 zone set --rgw-zone=cd-center1 --infile cd-center1.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center1-bj-1 zone set --rgw-zone=cd-center2 --infile cd-center2.json</span></span>
<span class="line"><span class="comment">#清空 region-map，radosgw-agent 读取的 regionmap 出自这里</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center1-bj-1 region-map set --infile bj.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center1-bj-1 regionmap update</span></span>
</pre></td></tr></table></figure>
     bj-2
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># cd /etc/ceph</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 region set --infile cd.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 zone set --rgw-zone=cd-center1 --infile cd-center1.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 zone set --rgw-zone=cd-center2 --infile cd-center2.json</span></span>
<span class="line"><span class="comment">#清空 region-map，radosgw-agent 读取的 regionmap 出自这里</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 region-map set --infile bj.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 regionmap update</span></span>
</pre></td></tr></table></figure>
     在 master region 两个节点上创建 secondary region、secondary zone，并更新 regionmap
cd-1
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># cd /etc/ceph</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center1-cd-1 region set --infile bj.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center1-cd-1 zone set --rgw-zone=bj-center1 --infile bj-center1.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center1-cd-1 zone set --rgw-zone=bj-center2 --infile bj-center2.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center1-cd-1 region-map set --infile cd.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center1-cd-1 regionmap update</span></span>
</pre></td></tr></table></figure>
     cd-2
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># cd /etc/ceph</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center2-cd-2 region set --infile bj.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center2-cd-2 zone set --rgw-zone=bj-center1 --infile bj-center1.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center2-cd-2 zone set --rgw-zone=bj-center2 --infile bj-center2.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center2-cd-2 region-map set --infile cd.json</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.cd-center2-cd-2 regionmap update</span></span>
</pre></td></tr></table></figure>3.  重启 ceph-radosgw 服务，验证

     重启各个节点的 ceph-radosgw 服务

     master region 验证 region
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># radosgw-admin regionmap get --name client.radosgw.cd-center2-cd-2</span></span>
<span class="line">&#123; <span class="string">"regions"</span>: [</span>
<span class="line"> &#123; <span class="string">"key"</span>: <span class="string">"bj"</span>,</span>
<span class="line"> <span class="string">"val"</span>: &#123; <span class="string">"name"</span>: <span class="string">"bj"</span>,</span>
<span class="line"> <span class="string">"api_name"</span>: <span class="string">"bj"</span>,</span>
<span class="line"> <span class="string">"is_master"</span>: <span class="string">"false"</span>,</span>
<span class="line"> <span class="string">"endpoints"</span>: [</span>
<span class="line"> <span class="string">"http:\/\/bj-1:80\/"</span>],</span>
<span class="line"> <span class="string">"hostnames"</span>: [</span>
<span class="line"> <span class="string">"cd-1"</span>],</span>
<span class="line"> <span class="string">"master_zone"</span>: <span class="string">"bj-center1"</span>,</span>
<span class="line"> <span class="string">"zones"</span>: [</span>
<span class="line"> &#123; <span class="string">"name"</span>: <span class="string">"bj-center1"</span>,</span>
<span class="line"> <span class="string">"endpoints"</span>: [</span>
<span class="line"> <span class="string">"http:\/\/bj-1:80\/"</span>],</span>
<span class="line"> <span class="string">"log_meta"</span>: <span class="string">"true"</span>,</span>
<span class="line"> <span class="string">"log_data"</span>: <span class="string">"true"</span>&#125;,</span>
<span class="line"> &#123; <span class="string">"name"</span>: <span class="string">"bj-center2"</span>,</span>
<span class="line"> <span class="string">"endpoints"</span>: [</span>
<span class="line"> <span class="string">"http:\/\/bj-2:80\/"</span>],</span>
<span class="line"> <span class="string">"log_meta"</span>: <span class="string">"true"</span>,</span>
<span class="line"> <span class="string">"log_data"</span>: <span class="string">"true"</span>&#125;],</span>
<span class="line"> <span class="string">"placement_targets"</span>: [</span>
<span class="line"> &#123; <span class="string">"name"</span>: <span class="string">"default-placement"</span>,</span>
<span class="line"> <span class="string">"tags"</span>: []&#125;],</span>
<span class="line"> <span class="string">"default_placement"</span>: <span class="string">"default-placement"</span>&#125;&#125;,</span>
<span class="line"> &#123; <span class="string">"key"</span>: <span class="string">"cd"</span>,</span>
<span class="line"> <span class="string">"val"</span>: &#123; <span class="string">"name"</span>: <span class="string">"cd"</span>,</span>
<span class="line"> <span class="string">"api_name"</span>: <span class="string">"cd"</span>,</span>
<span class="line"> <span class="string">"is_master"</span>: <span class="string">"true"</span>,</span>
<span class="line"> <span class="string">"endpoints"</span>: [</span>
<span class="line"> <span class="string">"http:\/\/cd-1:80\/"</span>],</span>
<span class="line"> <span class="string">"hostnames"</span>: [</span>
<span class="line"> <span class="string">"cd-1"</span>],</span>
<span class="line"> <span class="string">"master_zone"</span>: <span class="string">"cd-center1"</span>,</span>
<span class="line"> <span class="string">"zones"</span>: [</span>
<span class="line"> &#123; <span class="string">"name"</span>: <span class="string">"cd-center1"</span>,</span>
<span class="line"> <span class="string">"endpoints"</span>: [</span>
<span class="line"> <span class="string">"http:\/\/cd-1:80\/"</span>],</span>
<span class="line"> <span class="string">"log_meta"</span>: <span class="string">"true"</span>,</span>
<span class="line"> <span class="string">"log_data"</span>: <span class="string">"true"</span>&#125;,</span>
<span class="line"> &#123; <span class="string">"name"</span>: <span class="string">"cd-center2"</span>,</span>
<span class="line"> <span class="string">"endpoints"</span>: [</span>
<span class="line"> <span class="string">"http:\/\/cd-2:80\/"</span>],</span>
<span class="line"> <span class="string">"log_meta"</span>: <span class="string">"true"</span>,</span>
<span class="line"> <span class="string">"log_data"</span>: <span class="string">"true"</span>&#125;],</span>
<span class="line"> <span class="string">"placement_targets"</span>: [</span>
<span class="line"> &#123; <span class="string">"name"</span>: <span class="string">"default-placement"</span>,</span>
<span class="line"> <span class="string">"tags"</span>: []&#125;],</span>
<span class="line"> <span class="string">"default_placement"</span>: <span class="string">"default-placement"</span>&#125;&#125;],</span>
<span class="line"> <span class="string">"master_region"</span>: <span class="string">"cd"</span>,</span>
<span class="line"> <span class="string">"bucket_quota"</span>: &#123; <span class="string">"enabled"</span>: <span class="literal">false</span>,</span>
<span class="line"> <span class="string">"max_size_kb"</span>: -<span class="number">1</span>,</span>
<span class="line"> <span class="string">"max_objects"</span>: -<span class="number">1</span>&#125;,</span>
<span class="line"> <span class="string">"user_quota"</span>: &#123; <span class="string">"enabled"</span>: <span class="literal">false</span>,</span>
<span class="line"> <span class="string">"max_size_kb"</span>: -<span class="number">1</span>,</span>
<span class="line"> <span class="string">"max_objects"</span>: -<span class="number">1</span>&#125;&#125;</span>
</pre></td></tr></table></figure>
     其他节点类似，保证 regionmap 完整即可 

## [](https://ly798.github.io/2016/01/26/Radosgw-%E5%A4%9A-region-%E9%83%A8%E7%BD%B2%E4%BB%A5%E5%8F%8A-radosgw_agent-%E5%90%8C%E6%AD%A5/#u914D_u7F6E_radosgw-agent "配置 radosgw-agent")配置 radosgw-agent

新建文件/etc/ceph/radosgw-agent/default.conf，在节点 bj-1 上
<figure class="highlight http"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line"><span class="attribute">src_access_key</span>: <span class="string">NDOO5JZAAM2Z6WA9FIXG</span></span>
<span class="line"><span class="attribute">src_secret_key</span>: <span class="string">4rWHptoevPLPlOwDITljxPuJTqoMQCMM9ICBjGgv</span></span>
<span class="line"><span class="attribute">source</span>: <span class="string">http://cd-1:80</span></span>
<span class="line"><span class="attribute">dest_access_key</span>: <span class="string">2B28YY7D2VH67EAC8AAR</span></span>
<span class="line"><span class="attribute">dest_secret_key</span>: <span class="string">RpF0yzJehzFaxsVLi5yYpC4OgdE77hqHYTyfA9un</span></span>
<span class="line"><span class="attribute">destination</span>: <span class="string">http://bj-1:80</span></span>
<span class="line"><span class="attribute">log_file</span>: <span class="string">/var/log/radosgw/radosgw-sync-cd-bj.log</span></span>
<span class="line"><span class="attribute">verbose</span>: <span class="string">False</span></span>
<span class="line"><span class="attribute">versioned</span>: <span class="string">True</span></span>
<span class="line"><span class="attribute">metadata_only</span>: <span class="string">True</span></span>
</pre></td></tr></table></figure>

注意 region 之间的同步只能是元数据同步，所以`metadata_only: True`

启动服务
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># mkdir /var/run/ceph/radosgw-agent/</span></span>
<span class="line"><span class="comment"># /etc/init.d/ceph-radosgw start</span></span>
</pre></td></tr></table></figure>

可以查看同步日志
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># tailf /var/log/radosgw/radosgw-sync-cd-bj.log</span></span>
</pre></td></tr></table></figure>

此时，在 secondary region 任意节点可以查看 metadata
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 metadata list user</span></span>
<span class="line">[</span>
<span class="line"> <span class="string">"bj-center2-bj-2"</span>,</span>
<span class="line"> <span class="string">"cd-center1-cd-1"</span>,</span>
<span class="line"> <span class="string">"bj-center1-bj-1"</span>,</span>
<span class="line"> <span class="string">"testuser"</span>,</span>
<span class="line"> <span class="string">"cd-center2-cd-2"</span>]</span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 metadata list bucket</span></span>
<span class="line">[</span>
<span class="line"> <span class="string">"demo"</span>,</span>
<span class="line"> <span class="string">"test"</span>,</span>
<span class="line"> <span class="string">"aaa"</span>]</span>
<span class="line"><span class="comment">#因为只同步了元数据，查看 bucket 内的对象则为空</span></span>
<span class="line"><span class="comment"># /home/ceph/src/.libs/radosgw-admin --name client.radosgw.bj-center2-bj-2 bucket list --bucket=aaa</span></span>
<span class="line">[]</span>
</pre></td></tr></table></figure>
