---
title: Ceph user managent
tags: [ceph]
date: 2015-09-25 10:42:12
---

## [](https://ly798.github.io/2015/09/25/Ceph-user-managent/#u4ECB_u7ECD "介绍")介绍

ceph一个用户是一个人或者一个系统应用，创建用户可以用来控制谁能够来访问集群以及pool和pool中的数据。

ceph有用户类型的概念。为了达到管理用户的目的，用户组的类型常常是client，eg：client.admin、client.user1。
设置用户类型的原因是mon、osd、mds服务也会使用cephx，但是他们不是client。这就是client用户和other用户的区别。
 <!-- more --> 

ceph用户与ceph对象存储用户和ceph文件系统用户不同。
ceph object gw使用ceph对象存储用户来进行gw daemon和存储集群之间的通信。
Ceph文件系统使用POSIX语义。与这里的集群用户都不一样。

## [](https://ly798.github.io/2015/09/25/Ceph-user-managent/#u7528_u6237_u6743_u9650 "用户权限")用户权限

ceph使用CAPS来描述用户权限

    #查看所有用户 # ceph auth list installed auth entries: osd.0 key: AQAZCfxVTedyJhAA5zItpwd/UoO9jUjvM8+sMA== caps: [mon] allow profile osd caps: [osd] allow * key: AQBECPxV7Ny8GBAARhGuGVVZ1hpU2X0HCy/xQQ== caps: [mds] allow caps: [mon] allow * caps: [osd] allow * client.cinder key: AQDmnANW3me8NhAA3s1VEyEydosTZLUQ4exn2g== caps: [mon] allow r caps: [osd] allow class-read object_prefix rbd_children, allow rwx pool=volumes, allow rwx pool=vms, allow rx pool=images client.rgw.ceph4 key: AQBPUAFWoES9FxAA1TERsuQFIKTp0AZB7Cdowg== caps: [mon] allow rw caps: [osd] allow rwx `</pre>

    CAPS语法如下：
     <pre>`{daemon-type} &apos;allow {capability}&apos; [{daemon-type} &apos;allow {capability}&apos;] `</pre>

    关于权限：
     <table> <thead> <tr> <th style="text-align:left">item</th> <th style="text-align:left">描述</th> </tr> </thead> <tbody> <tr> <td style="text-align:left">profile osd</td> <td style="text-align:left">让一个用户可以作为允许作为osd连接其他的osd，osd之间的心跳和状态报告。</td> </tr> <tr> <td style="text-align:left">profile mds</td> <td style="text-align:left">让一个用户可以作为允许作为mds连接其他的mds</td> </tr> <tr> <td style="text-align:left">profile bootstrap-osd</td> <td style="text-align:left">让一个用户可以启动osd，一些部署工具，such as ceph-disk, ceph-deploy</td> </tr> <tr> <td style="text-align:left">profile bootstrap-mds</td> <td style="text-align:left">N/A</td> </tr> </tbody> </table> 

    ## [](https://ly798.github.io/2015/09/25/Ceph-user-managent/#u7528_u6237_u7BA1_u7406 "用户管理")用户管理

    admin用户可以直接在集群中创建、修改和删除用户。

*   查
     -o 可以输出到文件
     <pre>`#ceph auth list #ceph auth get {TYPE.ID} #ceph auth export {TYPE.ID} #ceph auth print-key {TYPE.ID} `</pre>
*   增
     创建一个用户包括用户名、密钥、CAPS
     密钥用于认证集群，CAPS用于控制权限
     <pre>`#ceph auth add #ceph auth get-or-create 返回用户名和密钥，-o 可以输出到文件 #ceph auth get-or-create-key 只返回密钥，-o 可以输出到文件 #ceph auth import -i {path} `</pre>
     一个典型的用户至少有多mon的读能力、对osd的读写能力针对于某个pool
     <pre>`#ceph auth get-or-create client.yippee #没有任何权限 #ceph auth del client.yippee #ceph auth get-or-create client.yippee mon &apos;allow r&apos; osd &apos;allow rw pool=images&apos; `</pre>
*   改
     修改caps会抹去之前的所有caps
     <pre>`ceph auth caps USERTYPE.USERID {daemon} &apos;allow [r|w|x|*|...] [pool={pool-name}] [namespace={namespace-name}]&apos; [{daemon} &apos;allow [r|w|x|*|...] [pool={pool-name}] [namespace={namespace-name}]&apos;] #ceph auth caps client.yippee mon &apos;allow r&apos; osd &apos;allow rw pool=vms&apos; `</pre>
*   删
     <pre>`ceph auth del {TYPE.ID} `</pre> 

    ## [](https://ly798.github.io/2015/09/25/Ceph-user-managent/#keyring_management "keyring management")keyring management

    当作为客户端访问ceph集群的时候，会到几个默认目录下寻找，所以不需要手动指定。
     <pre>`/etc/ceph/$cluster.$name.keyring /etc/ceph/$cluster.keyring /etc/ceph/keyring /etc/ceph/keyring.bin 

$cluster指你的cpeh集群名字，在配置文件中有
$name指用户名字，{TYPE.ID}

## [](https://ly798.github.io/2015/09/25/Ceph-user-managent/#u603B_u7ED3 "总结")总结

*   ceph 用户要区别于 ceph radosgw 的用户。*   mon、osd、rgw、mds以及使用rbd等都是以 Ceph 用户的身份去访问 Ceph 集群。 

## [](https://ly798.github.io/2015/09/25/Ceph-user-managent/#u53C2_u8003 "参考")参考

Ceph Doc：[http://docs.ceph.com/docs/master/rados/operations/user-management/](http://docs.ceph.com/docs/master/rados/operations/user-management/)
