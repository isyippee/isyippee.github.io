---
title: trove introduction and practice
tags: [openstack trove]
date: 2016-12-09 10:24:18
---

## [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#introduction "introduction")introduction

Trove is Database as a Service for OpenStack.
 <!-- more --> 

## [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#design "design")design
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
</pre></td><td class="code"><pre><span class="line">+-------------+ +-----------+ rabbitmq +-------------+ </span>
<span class="line">|<span class="string"> troveclient </span>|<span class="string">-----&gt;</span>|<span class="string"> trove-api </span>|<span class="string">---------&gt;</span>|<span class="string"> taskmanager </span>|<span class="string"> </span>
<span class="line">+-------------+ +-----------+ +-------------+ </span>
<span class="line"> </span>|<span class="string"> </span>
<span class="line">+-----------------------------------+ </span>|<span class="string"> rabbitmq</span>
<span class="line"></span>|<span class="string"> </span>|<span class="string"> </span>|</span>
<span class="line">|<span class="string"> trove instance +-------------------------+ +-----------------+</span>
<span class="line"></span>|<span class="string"> </span>|<span class="string"> datastore/guest </span>|<span class="string">-----&gt;</span>|<span class="string"> trove-conductor </span>|</span>
<span class="line">|<span class="string"> +-------------------------+ +-----------------+</span>
<span class="line">+-----------------------------------+</span></span>
</pre></td></tr></table></figure> 

*   trove-api
*   trove-taskmanager: 管理数据库实例的生命周期和在数据库实例上的操作。
*   trove-guestagent: 运行在数据库实例内，监听 RPC，执行请求操作。
*   trove-conductor: guestagent 访问数据库的代理。 

一个 trove instance 对应一个 vm，trove-guestagent 运行在 vm 内, vm内数据库的数据分区硬盘是 cinder 提供，vm 关联一个 SecurityGroup,允许数据库端口打开。
一段源码：

    create_dns_client = import_class(CONF.remote_dns_client) create_guest_client = import_class(CONF.remote_guest_client) create_nova_client = import_class(CONF.remote_nova_client) create_swift_client = import_class(CONF.remote_swift_client) create_cinder_client = import_class(CONF.remote_cinder_client) create_heat_client = import_class(CONF.remote_heat_client) create_neutron_client = import_class(CONF.remote_neutron_client) 

调用了其他的组件来完成，trove 专注与数据库的管理。

## [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#images "images")images

image 包含了操作系统、cloud-init 等，另外：

*   sql engine (mysql)
*   trove-guestagent 

engine 和 trove-guestagent 也可以通过网络动态加载，在 API 中有一个参数是packages 可以指定软件，在 prepare instance 的时候会进行安装。

*   优点：

        *   灵活，只需要一个镜像，不同的 sql，指定不同的 packages*   缺点：

        *   效率，受网络等影响，延长了 instance init 的时间
    *   packages 生成的默认配置文件等可能会被 agent 用到 

## [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#feature "feature")feature

datastore、database、instance、configuration、backup、replication、cluster 和 数据库中的database、user

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#datastore "datastore")datastore

理解为数据库系统，对支持的数据库类型及其版本的管理。mysql、postgresql、mongo、redis、cassandra等, 主要针对 mysql 的 5.5 和 5.6。

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#database_instance "database instance")database instance

对数据库实例生命周期的管理，以及 instance 和 volume 的动态 resize。

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#configuration "configuration")configuration

动态配置更新，针对不同的 datastore有一组预定义的配置，提供给用户来自定义配置，并动态的 attach到数据库实例并生效。

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#backup "backup")backup

对数据库进行备份。

针对 mysql 提供全量备份和增量备份。

mysql 备份的实现：在 guest 使用 mysql 的备份工具 mysqldump 或innobackupex 进行备份，并通过 swift api 上传到对象存储。

我们 stack 没有使用 openstack swift，测试了 rgw 集成 keystone，使用 rgw的 swift api 来备份，有一个问题：在创建备份的时候，都会为 tenant创建一个名为 `database_backups` 的 bucket，目前 rgw 的 bucket命名是全局唯一的。

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#replication "replication")replication

主从结构。

支持 mysql 的主从数据库创建，实现主从同步复制。

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#cluster "cluster")cluster

数据库集群。

juno 版支持 mongo 集群。

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#database_u3001user "database、user")database、user

API 提供对数据库系统的数据库进行管理，提供对数据库用户的管理。

## [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#example "example")example

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u67E5_u8BE2_datastore__u53CA_u5176_u7248_u672C "查询 datastore 及其版本")查询 datastore 及其版本
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># trove datastore-list </span></span>
<span class="line">+--------------------------------------|<span class="string">-------+</span>
<span class="line"></span>|<span class="string"> ID </span>|<span class="string"> Name </span>|</span>
<span class="line">+--------------------------------------|<span class="string">-------+</span>
<span class="line"></span>|<span class="string"> f30f5a8c-51b2-4c19-9d13-26bb644dca3e </span>|<span class="string"> mysql </span>|</span>
<span class="line">+--------------------------------------|<span class="string">-------+</span>
<span class="line"># trove datastore-version-list f30f5a8c-51b2-4c19-9d13-26bb644dca3e</span>
<span class="line">+--------------------------------------</span>|<span class="string">------+</span>
<span class="line"></span>|<span class="string"> ID </span>|<span class="string"> Name </span>|</span>
<span class="line">+--------------------------------------|<span class="string">------+</span>
<span class="line"></span>|<span class="string"> 42bce448-ec76-4c80-bea0-128c4eee3471 </span>|<span class="string"> 5.5 </span>|</span>
<span class="line">|<span class="string"> c0581e9a-6597-4ee9-9f34-08362ab3bcc6 </span>|<span class="string"> 5.6 </span>|</span>
<span class="line">+--------------------------------------|<span class="string">------+</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u67E5_u8BE2_flavor "查询 flavor")查询 flavor
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># trove flavor-list </span></span>
<span class="line">+----|<span class="string">-----------</span>|<span class="string">-------+</span>
<span class="line"></span>|<span class="string"> ID </span>|<span class="string"> Name </span>|<span class="string"> RAM </span>|</span>
<span class="line">+----|<span class="string">-----------</span>|<span class="string">-------+</span>
<span class="line"></span>|<span class="string"> 1 </span>|<span class="string"> m1.tiny </span>|<span class="string"> 512 </span>|</span>
<span class="line">|<span class="string"> 2 </span>|<span class="string"> m1.small </span>|<span class="string"> 2048 </span>|</span>
<span class="line">|<span class="string"> 3 </span>|<span class="string"> m1.medium </span>|<span class="string"> 4096 </span>|</span>
<span class="line">|<span class="string"> 4 </span>|<span class="string"> m1.large </span>|<span class="string"> 8192 </span>|</span>
<span class="line">|<span class="string"> 5 </span>|<span class="string"> m1.xlarge </span>|<span class="string"> 16384 </span>|</span>
<span class="line">|<span class="string"> 6 </span>|<span class="string"> DS-512M </span>|<span class="string"> 512 </span>|</span>
<span class="line">|<span class="string"> 7 </span>|<span class="string"> DS-640M </span>|<span class="string"> 640 </span>|</span>
<span class="line">|<span class="string"> 8 </span>|<span class="string"> DS-384M </span>|<span class="string"> 384 </span>|</span>
<span class="line">|<span class="string"> 9 </span>|<span class="string"> DS-1024M </span>|<span class="string"> 1024 </span>|</span>
<span class="line">+----|<span class="string">-----------</span>|<span class="string">-------+</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u521B_u5EFA_database_instance "创建 database instance")创建 database instance
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># trove create master_1 6 --size=1 --datastore_version 5.5 --datastore mysql </span></span>
<span class="line">+-------------------|<span class="string">--------------------------------------+ </span>
<span class="line"></span>|<span class="string"> Property </span>|<span class="string"> Value </span>|<span class="string"> </span>
<span class="line">+-------------------</span>|<span class="string">--------------------------------------+ </span>
<span class="line"></span>|<span class="string"> created </span>|<span class="string"> 2016-12-07T03:00:46 </span>|<span class="string"> </span>
<span class="line"></span>|<span class="string"> datastore </span>|<span class="string"> mysql </span>|</span>
<span class="line">|<span class="string"> datastore_version </span>|<span class="string"> 5.5 </span>|<span class="string"> </span>
<span class="line"></span>|<span class="string"> flavor </span>|<span class="string"> 6 </span>|</span>
<span class="line">|<span class="string"> id </span>|<span class="string"> 9788b066-a071-4aa8-bfb3-dd592596b3c6 </span>|</span>
<span class="line">|<span class="string"> name </span>|<span class="string"> master_1 </span>|<span class="string"> </span>
<span class="line"></span>|<span class="string"> status </span>|<span class="string"> BUILD </span>|<span class="string"> </span>
<span class="line"></span>|<span class="string"> updated </span>|<span class="string"> 2016-12-07T03:00:46 </span>|</span>
<span class="line">|<span class="string"> volume </span>|<span class="string"> 1 </span>|</span>
<span class="line">+-------------------|<span class="string">--------------------------------------+</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u67E5_u770B_u5DF2_u5B58_u5728_u7684_configuration_group_2C__u5E76_attach__u5230_instance "查看已存在的 configuration group, 并 attach 到 instance")查看已存在的 configuration group, 并 attach 到 instance
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
<span class="line">22</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># trove configuration-list </span></span>
<span class="line">+--------------------------------------|<span class="string">----------</span>|<span class="string">-------------</span>|<span class="string">----------------</span>|<span class="string">------------------------+ </span>
<span class="line"></span>|<span class="string"> ID </span>|<span class="string"> Name </span>|<span class="string"> Description </span>|<span class="string"> Datastore Name </span>|<span class="string"> Datastore Version Name </span>|</span>
<span class="line">+--------------------------------------|<span class="string">----------</span>|<span class="string">-------------</span>|<span class="string">----------------</span>|<span class="string">------------------------+ </span>
<span class="line"></span>|<span class="string"> bd81ea8e-e94d-4d49-a83b-ec78ffc6f15d </span>|<span class="string"> charset </span>|<span class="string"> charset </span>|<span class="string"> mysql </span>|<span class="string"> 5.5 </span>|<span class="string"> </span>
<span class="line">+--------------------------------------</span>|<span class="string">----------</span>|<span class="string">-------------</span>|<span class="string">----------------</span>|<span class="string">------------------------+ </span>
<span class="line"># trove configuration-show bd81ea8e-e94d-4d49-a83b-ec78ffc6f15d </span>
<span class="line">+------------------------</span>|<span class="string">--------------------------------------------------------------------------------------------------------------------------------------------------------------------$----------------------------------------------------------------------------------------------------------------------------------+</span>
<span class="line"></span>|<span class="string"> Property </span>|<span class="string"> Value </span>|</span>
<span class="line">+------------------------|<span class="string">--------------------------------------------------------------------------------------------------------------------------------------------------------------------$----------------------------------------------------------------------------------------------------------------------------------+</span>
<span class="line"></span>|<span class="string"> created </span>|<span class="string"> 2016-11-25T03:11:28 </span>|</span>
<span class="line">|<span class="string"> datastore_name </span>|<span class="string"> mysql </span>|</span>
<span class="line">|<span class="string"> datastore_version_name </span>|<span class="string"> 5.5 </span>|</span>
<span class="line">|<span class="string"> description </span>|<span class="string"> charset </span>|</span>
<span class="line">|<span class="string"> id </span>|<span class="string"> bd81ea8e-e94d-4d49-a83b-ec78ffc6f15d </span>|</span>
<span class="line">|<span class="string"> instance_count </span>|<span class="string"> 0 </span>|</span>
<span class="line">|<span class="string"> name </span>|<span class="string"> charset </span>|</span>
<span class="line">|<span class="string"> updated </span>|<span class="string"> 2016-11-25T03:11:28 </span>|</span>
<span class="line">|<span class="string"> values </span>|<span class="string"> &#123;"collation_database": "utf8", "character_set_connection": "utf8", "character_set_client": "utf8", "collation_server": "utf8", "character_set_database": "utf8", "c$llation_connection": "utf8", "character_set_results": "utf8", "character_set_server": "utf8", "character_set_filesystem": "utf8"&#125; </span>|</span>
<span class="line">+------------------------|<span class="string">--------------------------------------------------------------------------------------------------------------------------------------------------------------------$</span>
<span class="line">----------------------------------------------------------------------------------------------------------------------------------+ </span>
<span class="line"># trove configuration-attach 9788b066-a071-4aa8-bfb3-dd592596b3c6 bd81ea8e-e94d-4d49-a83b-ec78ffc6f15d</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u521B_u5EFA_u4E00_u4E2A_database__u548C_database_user "创建一个 database 和 database user")创建一个 database 和 database user
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># trove database-create <span class="number">9788</span>b066-a071-<span class="number">4</span>aa8-bfb3-dd592596b3c6 mydb </span></span>
<span class="line"><span class="preprocessor"># trove user-create <span class="number">9788</span>b066-a071-<span class="number">4</span>aa8-bfb3-dd592596b3c6 admin admin --host % --databases mydb</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u521B_u5EFA_u4E00_u4E2A_backup "创建一个 backup")创建一个 backup
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># trove backup-create 9788b066-a071-4aa8-bfb3-dd592596b3c6 backup.001 </span></span>
<span class="line">+-------------|-------------------------------------------------------------------------------------------------+ </span>
<span class="line">| Property | Value |</span>
<span class="line">+-------------|-------------------------------------------------------------------------------------------------+ </span>
<span class="line">| created | <span class="number">2016</span>-<span class="number">12</span>-<span class="number">07</span>T03:<span class="number">17</span>:<span class="number">22</span> |</span>
<span class="line">| datastore | &#123;<span class="string">u'version'</span>: <span class="string">u'5.5'</span>, <span class="string">u'type'</span>: <span class="string">u'mysql'</span>, <span class="string">u'version_id'</span>: <span class="string">u'42bce448-ec76-4c80-bea0-128c4eee3471'</span>&#125; | </span>
<span class="line">| description | <span class="keyword">None</span> |</span>
<span class="line">| id | <span class="number">538</span>afc35-<span class="number">1112</span>-<span class="number">4</span>d44-bd32-eb1ac39a5f97 |</span>
<span class="line">| instance_id | <span class="number">9788</span>b066-a071-<span class="number">4</span>aa8-bfb3-dd592596b3c6 |</span>
<span class="line">| locationRef | <span class="keyword">None</span> |</span>
<span class="line">| name | backup<span class="number">.001</span> |</span>
<span class="line">| parent_id | <span class="keyword">None</span> |</span>
<span class="line">| size | <span class="keyword">None</span> |</span>
<span class="line">| status | NEW |</span>
<span class="line">| updated | <span class="number">2016</span>-<span class="number">12</span>-<span class="number">07</span>T03:<span class="number">17</span>:<span class="number">22</span> |</span>
<span class="line">+-------------|-------------------------------------------------------------------------------------------------+</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/09/trove-introduction-and-practice/#u521B_u5EFA_u4E00_u4E2A_replica "创建一个 replica")创建一个 replica
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># trove create slave_1 6 --size=1 --datastore_version 5.5 --datastore mysql --replica_of 9788b066-a071-4aa8-bfb3-dd592596b3c6 </span></span>
<span class="line">+-------------------|<span class="string">--------------------------------------+ </span>
<span class="line"></span>|<span class="string"> Property </span>|<span class="string"> Value </span>|<span class="string"> </span>
<span class="line">+-------------------</span>|<span class="string">--------------------------------------+ </span>
<span class="line"></span>|<span class="string"> created </span>|<span class="string"> 2016-12-07T03:18:50 </span>|</span>
<span class="line">|<span class="string"> datastore </span>|<span class="string"> mysql </span>|</span>
<span class="line">|<span class="string"> datastore_version </span>|<span class="string"> 5.5 </span>|</span>
<span class="line">|<span class="string"> flavor </span>|<span class="string"> 6 </span>|<span class="string"> </span>
<span class="line"></span>|<span class="string"> id </span>|<span class="string"> 819b1d3c-638a-4db2-a193-6066c54b1676 </span>|</span>
<span class="line">|<span class="string"> name </span>|<span class="string"> slave_1 </span>|</span>
<span class="line">|<span class="string"> replica_of </span>|<span class="string"> 9788b066-a071-4aa8-bfb3-dd592596b3c6 </span>|</span>
<span class="line">|<span class="string"> status </span>|<span class="string"> BUILD </span>|</span>
<span class="line">|<span class="string"> updated </span>|<span class="string"> 2016-12-07T03:18:50 </span>|</span>
<span class="line">|<span class="string"> volume </span>|<span class="string"> 1 </span>|</span>
<span class="line">+-------------------|<span class="string">--------------------------------------+</span></span>
</pre></td></tr></table></figure>
