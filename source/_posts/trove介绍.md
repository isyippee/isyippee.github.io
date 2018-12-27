---
title: trove介绍
tags: [openstack trove]
date: 2016-11-08 10:23:08
---

## [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#u4ECB_u7ECD "介绍")介绍

H版成为核心项目。
 <!-- more --> 

nova - 实例
cinder - 数据盘
neutron - net
glance - image

该实例中运行database和trove-guestagent，连接rabbitmq获取操作命令。

## [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#u57FA_u672C_u7528_u6CD5 "基本用法")基本用法

### [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#datastore "datastore")datastore
<figure class="highlight gherkin"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">//create a datastore </span>
<span class="line">//trove-manage datastore_update <span class="variable">&lt;datastore_name&gt;</span> <span class="variable">&lt;default_version&gt;</span> </span>
<span class="line"><span class="comment"># trove-manage datastore_update mysql "" </span></span>
<span class="line"></span>
<span class="line">//add a version to the datastore </span>
<span class="line">//trove-manage datastore_version_update <span class="variable">&lt;datastore_name&gt;</span> <span class="variable">&lt;version_name&gt;</span> <span class="variable">&lt;datastore_manager&gt;</span> <span class="variable">&lt;glance_id&gt;</span> <span class="variable">&lt;packages&gt;</span> <span class="variable">&lt;active:0/1&gt;</span> </span>
<span class="line"><span class="comment"># trove-manage datastore_version_update mysql 5.6 mysql 333ba4b1-638e-4fac-b516-28dad691e319 "" 1 </span></span>
<span class="line"></span>
<span class="line"><span class="comment"># trove datastore-list </span></span>
<span class="line">+--------------------------------------|<span class="string">------------+</span>
<span class="line"></span>|<span class="string"> ID </span>|<span class="string"> Name </span>|</span>
<span class="line">+--------------------------------------|<span class="string">------------+</span>
<span class="line"></span>|<span class="string"> f30f5a8c-51b2-4c19-9d13-26bb644dca3e </span>|<span class="string"> mysql </span>|</span>
<span class="line">+--------------------------------------|<span class="string">------------+</span>
<span class="line"></span>
<span class="line"># trove datastore-version-list mysql </span>
<span class="line">+--------------------------------------</span>|<span class="string">-----------------+</span>
<span class="line"></span>|<span class="string"> ID </span>|<span class="string"> Name </span>|</span>
<span class="line">+--------------------------------------|<span class="string">-----------------+</span>
<span class="line"></span>|<span class="string"> 645d6577-e140-450f-b01f-c07314e17131 </span>|<span class="string"> 5.6 </span>|</span>
<span class="line">+--------------------------------------|<span class="string">-----------------+</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#instance_28database_u3001user_29 "instance(database、user)")instance(database、user)
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
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># trove datastore-list </span></span>
<span class="line"><span class="preprocessor"># trove datastore-version-list mysql</span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># trove create mariadb1 <span class="number">6</span> --size <span class="number">2</span> --datastore mysql --datastore_version <span class="number">5.6</span> --users admin:admin --databases init_db --configuration &lt;&gt; </span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># trove user-create d5f2a516-<span class="number">0472</span>-<span class="number">4e8</span>f-<span class="number">9682</span>-f77f691b13a8 admin admin </span></span>
<span class="line"><span class="preprocessor"># trove user-list d5f2a516-<span class="number">0472</span>-<span class="number">4e8</span>f-<span class="number">9682</span>-f77f691b13a </span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># trove database-create d5f2a516-<span class="number">0472</span>-<span class="number">4e8</span>f-<span class="number">9682</span>-f77f691b13a8 initdb </span></span>
<span class="line"><span class="preprocessor"># trove database-list d5f2a516-<span class="number">0472</span>-<span class="number">4e8</span>f-<span class="number">9682</span>-f77f691b13a8 </span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># trove resize-volume d5f2a516-<span class="number">0472</span>-<span class="number">4e8</span>f-<span class="number">9682</span>-f77f691b13a8 <span class="number">2</span></span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#backup/restore "backup/restore")backup/restore
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># trove backup-create 019e0efb-8c27-428f-ad98-2cb801a42af2 backup.0001</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#relication "relication")relication
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"># trove create replica_1 9 --size=1 --datastore_version centos-mariadb --datastore mysql --replica_of b9697ca9-ecc4-4216-a3d7-37f7a7a8f2c8</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#configuration "configuration")configuration
<figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># trove-manage db_load_datastore_config_parameters mysql mysql-5.6 /usr/lib/python2.7/site-packages/trove/templates/mysql/validation-rules.json </span></span>
<span class="line"><span class="preprocessor"># trove configuration-parameter-list mysql-5.6 --datastore mysql </span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># trove configuration-create test '&#123;"max_connections":80&#125;' --datastore mysql --datastore_version mysql-5.6 </span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># trove configuration-attach 15a0c217-4a2b-4d64-9522-d8ef08016a39 9d345196-247d-473d-acdc-cb0825fb3d64</span></span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2016/11/08/trove-%E4%BB%8B%E7%BB%8D/#u5404_u7248_u672C "各版本")各版本

*   juno

        *   MySQL, PostgreSQL
    *   Apache Cassandra, MongoDB, Couchbase and Redis
    *   MYSQL replication
    *   MongoDB cluster 

*   liberty

        *   redis cluster*   mitaka

        *   mariadb cluster*   trove logging
