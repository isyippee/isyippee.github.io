---
title: Radosgw-agent 使用
tags: [ceph]
date: 2016-01-19 10:39:14
---

## [](https://ly798.github.io/2016/01/19/Radosgw-agent-%E4%BD%BF%E7%94%A8/#u4ECB_u7ECD "介绍")介绍

radosgw-agent 同步的数据分为 metasync 和 datasync
 <!-- more --> 

radosgw-agent 数据同步场景有两个：

*   同一个 region 中，同步 Master zone 的数据到 Secondary zone*   region 之间，同步 Master region 的数据到 Secondary region，且只能是 metasync 

## [](https://ly798.github.io/2016/01/19/Radosgw-agent-%E4%BD%BF%E7%94%A8/#zone__u4E4B_u95F4_u7684_radosgw-agent__u90E8_u7F72_u6CE8_u610F "zone 之间的 radosgw-agent 部署注意")zone 之间的 radosgw-agent 部署注意

*   在 region set 和 regionmap update 之后，需要重启 ceph-radosgw，否则通过 api 获得的 region map 是未被更新的。
*   在未成功配置 zone 的 master 和 slave 之前创建的 user、bucket、object 等都不能被 radosgw-agent 同步。 

## [](https://ly798.github.io/2016/01/19/Radosgw-agent-%E4%BD%BF%E7%94%A8/#u670D_u52A1_u542F_u52A8 "服务启动")服务启动

1.手动启动，显示的指定配置文件，运行的输出也会打印带屏幕
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># radosgw-agent -c &lt;config_file&gt;</span></span>
</pre></td></tr></table></figure>

2.服务的方式启动，默认的配置文件是在/etc/ceph/radosgw-agent/default.conf，手动创建并启动
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># /etc/init.d/radosgw-agent start</span></span>
</pre></td></tr></table></figure>

## [](https://ly798.github.io/2016/01/19/Radosgw-agent-%E4%BD%BF%E7%94%A8/#u914D_u7F6E_u6587_u4EF6 "配置文件")配置文件

配置文件demo
<figure class="highlight lasso"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">src_access_key: NDOO5JZAAM2Z6WA9FIXG</span>
<span class="line">src_secret_key: <span class="number">4</span>rWHptoevPLPlOwDITljxPuJTqoMQCMM9ICBjGgv</span>
<span class="line"><span class="variable">#source</span>: http:<span class="comment">//cd-1:80</span></span>
<span class="line"><span class="variable">#src_zone</span>: cd<span class="attribute">-center1</span></span>
<span class="line">dest_access_key: FLDVEMPT8PKY1F2UD3F7</span>
<span class="line">dest_secret_key: <span class="number">8</span>HnGUZIveuHAWePuGXEDXYTR985m7voSUuQDGPkx</span>
<span class="line">destination: http:<span class="comment">//cd-2:80</span></span>
<span class="line">log_file: /<span class="built_in">var</span>/<span class="keyword">log</span>/radosgw/radosgw<span class="attribute">-sync</span><span class="attribute">-cd</span><span class="attribute">-center1</span><span class="attribute">-center2</span><span class="built_in">.</span><span class="keyword">log</span></span>
<span class="line"><span class="variable">#verbose</span>: <span class="literal">True</span>/<span class="literal">False</span></span>
<span class="line"><span class="variable">#versioned</span>: <span class="literal">True</span>/<span class="literal">False</span></span>
<span class="line"><span class="variable">#sync_scope</span>: <span class="literal">full</span>/incremental</span>
</pre></td></tr></table></figure>

*   verbose：是否打开 debug
*   versioned：集群是否开启对象的多版本
*   sync_scope：同步方式，full（完全），incremental（增量） 

## [](https://ly798.github.io/2016/01/19/Radosgw-agent-%E4%BD%BF%E7%94%A8/#u95EE_u9898 "问题")问题

在 radosgw-agent 启动的时候有日志输出：
<figure class="highlight markdown"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">[<span class="link_label">radosgw_agent</span>][<span class="link_reference">INFO </span>] http://cd-1:80 endpoint does not support versioning</span>
<span class="line">[<span class="link_label">radosgw_agent</span>][<span class="link_reference">WARNIN</span>] encountered issues reaching to endpoint http://cd-1:80</span>
<span class="line">[<span class="link_label">radosgw_agent</span>][<span class="link_reference">WARNIN</span>] HTTP Error 403:</span>
</pre></td></tr></table></figure>

对应代码：
radosgw-agent/cli.py
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">check_versioning</span><span class="params">(endpoint)</span>:</span></span>
<span class="line"> date = time.asctime(time.gmtime())</span>
<span class="line"> signed_string = sign_string(endpoint.secret_key, date=date)</span>
<span class="line"> url = str(endpoint) + <span class="string">'/?versions'</span></span>
<span class="line"> headers = &#123;</span>
<span class="line"> <span class="string">'Authorization'</span>: <span class="string">'AWS '</span> + endpoint.access_key + <span class="string">':'</span> + signed_string,</span>
<span class="line"> <span class="string">'Date'</span>: date</span>
<span class="line"> &#125;</span>
<span class="line"> data = <span class="keyword">None</span></span>
<span class="line"> req = urllib2.Request(url, data, headers)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> response = urllib2.urlopen(req)</span>
<span class="line"> response.read()</span>
<span class="line"> log.debug(<span class="string">'%s endpoint supports versioning'</span> % endpoint)</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">True</span></span>
<span class="line"> <span class="keyword">except</span> HTTPError <span class="keyword">as</span> error:</span>
<span class="line"> <span class="keyword">if</span> error.code == <span class="number">403</span>:</span>
<span class="line"> log.info(<span class="string">'%s endpoint does not support versioning'</span> % endpoint)</span>
<span class="line"> log.warning(<span class="string">'encountered issues reaching to endpoint %s'</span> % endpoint)</span>
<span class="line"> log.warning(error)</span>
<span class="line"> <span class="keyword">except</span> URLError <span class="keyword">as</span> error:</span>
<span class="line"> log.error(<span class="string">"was unable to connect to %s"</span> % url)</span>
<span class="line"> log.error(error)</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">False</span></span>
</pre></td></tr></table></figure>

因为我们没有在配置文件中配置`versioned`，则会调用方法`check_versioning`，当前版本的 radosgw 不支持 versioning，所以这个报错是正常的。
修改配置文件项
<figure class="highlight http"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="attribute">versioned</span>: <span class="string">False</span></span>
</pre></td></tr></table></figure>
