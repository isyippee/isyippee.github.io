---
title: trove mysql replication
tags: [openstack trove]
date: 2017-01-20 10:18:42
---

## [](https://ly798.github.io/2017/01/20/trove-mysql-replication/#introduction "introduction")introduction

trove 版本：newton

mysql replication 的方式:

*   binlog
*   GTID <!-- more --> 

## [](https://ly798.github.io/2017/01/20/trove-mysql-replication/#operations "operations")operations

创建一个普通的 trove instance:
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">[root@trove-service-<span class="number">1</span>]<span class="comment">#trove create master 6 --size 1 --datastore mysql --datastore_version 5.5</span></span>
</pre></td></tr></table></figure> 

创建一个从实例，以上一个实例为主：
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">[root@trove-service-<span class="number">1</span>]<span class="comment">#trove create slave_1 6 --size 1 --datastore mysql --datastore_version 5.5 --replica_of 8f43762e-e324-4e1f-b628-a4ec41bc9035</span></span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2017/01/20/trove-mysql-replication/#problem "problem")problem

mysql5.5 只支持 binlog，而 trove 默认使用 GTID, 如果需要使用 binlog 的方式，修改配置文件 `trove-guestagent.conf` ：
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">[mysql]</span>
<span class="line">replication_strategy = MysqlBinlogReplication</span>
<span class="line">replication_namespace = trove.guestagent.strategies.replication.mysql_binlog</span>
</pre></td></tr></table></figure> 

在进行 replication 时需要设置 master, 即 enable<sub>asmaster</sub>, 会根据不同的 datastore<sub>version</sub> 去 patch 不同的 config template trove/taskmanager/models.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">_render_replica_source_config</span><span class="params">(self, flavor)</span>:</span></span>
<span class="line"> config = template.ReplicaSourceConfigTemplate(</span>
<span class="line"> self.datastore_version, flavor, self.id)</span>
<span class="line"> config.render()</span>
<span class="line"> <span class="keyword">return</span> config</span>
</pre></td></tr></table></figure> 

如果定义的 datastore<sub>version</sub> 不是 5.5 或者 5.6，则会使用了默认的 GTID 的 config, 若你使用的是 mysql5.5 则会导致配置文件错误 mysql 启动失败, 从而主从失败, 如：
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line">[root@trove-service-<span class="number">1</span>]<span class="comment"># trove datastore-version-list mysql</span></span>
<span class="line">+--------------------------------------|-----------+</span>
<span class="line">| ID | Name |</span>
<span class="line">+--------------------------------------|-----------+</span>
<span class="line">| <span class="number">67</span>e93968-<span class="number">2</span>d67-<span class="number">4</span>ed4-<span class="number">81</span>e4-<span class="number">4688</span>d539d3c2 | <span class="number">5.5</span> |</span>
<span class="line">| ac3b88b1<span class="operator">-d</span>349-<span class="number">4</span>b23-<span class="number">9</span>b50-<span class="number">6</span>edb3b55dcbc | mysql-<span class="number">5.5</span> |</span>
<span class="line">+--------------------------------------|-----------+</span>
</pre></td></tr></table></figure> 

当以 version 是 mysql-5.5 创建的 trove instance 作为 master 来创建从节点的时候就会失败。
