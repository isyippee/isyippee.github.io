---
title: Ceph radosgw gc 的处理过程
tags: [ceph]
date: 2015-12-11 10:42:29
---

## [](https://ly798.github.io/2015/12/11/Ceph-radosgw-gc-%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B/#u4ECB_u7ECD "介绍")介绍

通过 radosgw 删除一个对象之后，默认情况下data还是存在pool中，需要通过gc来自动删除。 
 <!-- more --> 

## [](https://ly798.github.io/2015/12/11/Ceph-radosgw-gc-%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B/#u914D_u7F6E "配置")配置

有关 gc 的配置项有如下：

    //查看有关gc的配置项（默认） # ceph daemon /var/run/ceph/ceph-client.radosgw.cd-ceph2.asok config show | grep gc &quot;rgw_gc_max_objs&quot;: &quot;32&quot;, &quot;rgw_gc_obj_min_wait&quot;: &quot;7200&quot;, &quot;rgw_gc_processor_max_time&quot;: &quot;3600&quot;, &quot;rgw_gc_processor_period&quot;: &quot;3600&quot;, `</pre>

    对于配置项的解释：
     <table> <thead> <tr> <th style="text-align:left">配置项</th> <th style="text-align:left">描述</th> </tr> </thead> <tbody> <tr> <td style="text-align:left">rgw gc max objs</td> <td style="text-align:left">垃圾回收进程在一个处理周期内可处理的最大对象数。</td> </tr> <tr> <td style="text-align:left">rgw gc obj min wait</td> <td style="text-align:left">对象可被删除并由垃圾回收器处理前最少等待多长时间。</td> </tr> <tr> <td style="text-align:left">rgw gc processor max time</td> <td style="text-align:left">两个连续的垃圾回收周期起点的最大时间间隔。</td> </tr> <tr> <td style="text-align:left">rgw gc processor period</td> <td style="text-align:left">垃圾回收进程的运行周期。</td> </tr> </tbody> </table> 

    ## [](https://ly798.github.io/2015/12/11/Ceph-radosgw-gc-%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B/#u8FC7_u7A0B "过程")过程

1.  创建一个大文件，上传为一个对象
     <pre>`# dd if=/dev/zero of=./brockfile bs=1M count=100 s3cmd put ./brockfile s3://demo # s3cmd ls s3://demo 2015-12-11 04:32 104857600 s3://demo/brockfile `</pre> 

1.  查看pool中的对象数据
     <pre>`# rados ls -p .cd.rgw.buckets | grep brockfile cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.5_3 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1_1 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.6_2 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.5_2 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.2_2 ... cd.4397.6_brockfile cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.3_3 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.3_2 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.6_1 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.2_1 # rados ls -p .cd.rgw.buckets | grep brockfile | wc -l 28 `</pre> 

1.  删除这个对象，并查看记录pool中的对象数据以及时间
     <pre>`# s3cmd rm s3://demo/brockfile delete: &apos;s3://demo/brockfile&apos; # date 2015年 12月 11日 星期五 13:37:07 CST # rados ls -p .cd.rgw.buckets | grep brockfile cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.5_3 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1_1 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.6_2 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.5_2 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.2_2 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.7_1 cd.4397.6__shadow_brockfile.2/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1_3 ... # rados ls -p .cd.rgw.buckets | grep brockfile | wc -l 27 `</pre>
    对比删除之前pool中对象少了`cd.4397.6_brockfile`。 

1.  查看gc任务
     <pre>`# radosgw-admin --name client.radosgw.cd-ceph1 gc list [] # radosgw-admin --name client.radosgw.cd-ceph1 gc list --include-all [ { &quot;tag&quot;: &quot;cd.4518.9&quot;, &quot;time&quot;: &quot;2015-12-11 14:18:13.693320&quot;, &quot;objs&quot;: []}, { &quot;tag&quot;: &quot;cd.4751.18&quot;, &quot;time&quot;: &quot;2015-12-11 15:36:23.285988&quot;, &quot;objs&quot;: [ { &quot;pool&quot;: &quot;.cd.rgw.buckets&quot;, &quot;oid&quot;: &quot;cd.4397.6__multipart_brockfile.2\/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1&quot;, &quot;key&quot;: &quot;&quot;}, { &quot;pool&quot;: &quot;.cd.rgw.buckets&quot;, &quot;oid&quot;: &quot;cd.4397.6__shadow_brockfile.2\/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1_1&quot;, &quot;key&quot;: &quot;&quot;}, { &quot;pool&quot;: &quot;.cd.rgw.buckets&quot;, &quot;oid&quot;: &quot;cd.4397.6__shadow_brockfile.2\/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1_2&quot;, &quot;key&quot;: &quot;&quot;}, { &quot;pool&quot;: &quot;.cd.rgw.buckets&quot;, &quot;oid&quot;: &quot;cd.4397.6__shadow_brockfile.2\/WrriwdjujTUjOrTkJM61hvs9U3aWI1S.1_3&quot;, &quot;key&quot;: &quot;&quot;}, ... ]}, { &quot;tag&quot;: &quot;cd.4526.13&quot;, &quot;time&quot;: &quot;2015-12-11 14:15:49.654526&quot;, &quot;objs&quot;: []}] `</pre>
     其中出现的每一个列表元素都是一个gc任务，都是计划中的任务，时间点在`time`字段。但radosgw并不能保证在指定的时间就一定回收掉这个对象。如：
     <pre>`# date 2015年 12月 11日 星期五 15:49:12 CST # rados ls -p .cd.rgw.buckets | grep brockfile | wc -l 27 `</pre>
     时间已经超过，但对象仍然没有被清理，具体原因是处于定义的时间点之后的gc还未开始运行，或者已经排在了gc处理的最大对象数量之后。
     等待一段时间，清理成功：
     <pre>`# rados ls -p .cd.rgw.buckets | grep brockfile | wc -l 0 [root@ceph3 ~]# date 2015年 12月 11日 星期五 16:12:51 CST 

## [](https://ly798.github.io/2015/12/11/Ceph-radosgw-gc-%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B/#u603B_u7ED3 "总结")总结

*   通过radosgw删除一个对象，是将其对象标记为删除，实质上是占用了存储空间，需要等待gc来进行清理。*   当删除时，会根据配置项`rgw gc obj min wait`来计算出一个该对象的删除的时间点，也就是在这个时间点才会被gc发现，并放入清理队列。*   而gc有配置项`rgw_gc_max_objs`，指明了一次处理的对象数，也就是说排在后面的gc任务可能会搁置到下一次gc。
*   gc的运行频率是根据配置`rgw_gc_processor_max_time`、`rgw_gc_processor_period`来定义。 

## [](https://ly798.github.io/2015/12/11/Ceph-radosgw-gc-%E5%A4%84%E7%90%86%E8%BF%87%E7%A8%8B/#u53C2_u8003 "参考")参考
