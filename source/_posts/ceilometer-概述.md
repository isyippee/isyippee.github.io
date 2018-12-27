---
title: ceilometer 概述
tags: [openstack, ceilometer]
date: 2015-07-07 10:40:38
---


## [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#ceilmeter_u4E3B_u8981_u6982_u5FF5 "ceilmeter主要概念")ceilmeter主要概念

<!-- more -->
*   meter:资源的可计量值，如一个虚拟机，有运行时间、cpu使用率、内存使用率等；<!-- more --> 有三种类型：
    1.  cumulative:累计的值，eg:虚拟机运行时间
    2.  gauge:离散或波动的值，eg:浮动ip、磁盘瞬间读写
    3.  delta:变化的值，eg:带宽
*   sample:采样数据，是每个采集点上的meter对应的值；
*   statistics:某个周期内的统计数据，如平均值、峰值等；
*   resource:是被监控的资源对象，虚拟机、卷等；
*   alarm:报警系统。有两种类型：
    1.  单个阈值报警
    2.  多个组合条件报警

## [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#ceilometer_u4E2D_u7684_u670D_u52A1 "ceilometer中的服务")ceilometer中的服务

### [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#u670D_u52A1_u5217_u8868 "服务列表")服务列表

项目中的setup.cfg
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">console_scripts =</span>
<span class="line"> ceilometer-api = ceilometer<span class="class">.cmd</span><span class="class">.api</span>:main</span>
<span class="line"> ceilometer-agent-central = ceilometer<span class="class">.cmd</span><span class="class">.agent_central</span>:main</span>
<span class="line"> ceilometer-agent-compute = ceilometer<span class="class">.cmd</span><span class="class">.agent_compute</span>:main</span>
<span class="line"> ceilometer-agent-notification = ceilometer<span class="class">.cmd</span><span class="class">.agent_notification</span>:main</span>
<span class="line"> ceilometer-agent-ipmi = ceilometer<span class="class">.cmd</span><span class="class">.agent_ipmi</span>:main</span>
<span class="line"> ceilometer-send-sample = ceilometer<span class="class">.cli</span>:send_sample</span>
<span class="line"> ceilometer-dbsync = ceilometer<span class="class">.cmd</span><span class="class">.storage</span>:dbsync</span>
<span class="line"> ceilometer-expirer = ceilometer<span class="class">.cmd</span><span class="class">.storage</span>:expirer</span>
<span class="line"> ceilometer-rootwrap = oslo<span class="class">.rootwrap</span><span class="class">.cmd</span>:main</span>
<span class="line"> ceilometer-collector = ceilometer<span class="class">.cmd</span><span class="class">.collector</span>:main</span>
<span class="line"> ceilometer-alarm-evaluator = ceilometer<span class="class">.cmd</span><span class="class">.alarm</span>:evaluator</span>
<span class="line"> ceilometer-alarm-notifier = ceilometer<span class="class">.cmd</span><span class="class">.alarm</span>:notifier</span>
</pre></td></tr></table></figure>

对应的就是服务。

### [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#u76D1_u63A7_u6570_u636E_u7684_u6536_u96C6 "监控数据的收集")监控数据的收集

官网上的一张图：
![](ceilometer-collect.jpg)

对应上图来看，监控数据有两种方式收集：

*   通过调用各个模块的API去主动获取数据；
*   消费各个模块推送到notification队列（oslo-messaging）的通知，被动的采集监控数据。

### [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#ceilometer-agent-compute "ceilometer-agent-compute")ceilometer-agent-compute

*   运行在每个compute节点；
*   主要用来收集计算节点上虚拟机的监控数据；
*   agent管理一组插件，分别来获取虚拟机的cpu、disk等信息，大部分通过调用Hypervisor的API，需要定期轮询来获取监控数据。

### [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#ceilometer-agent-central "ceilometer-agent-central")ceilometer-agent-central

*   运行在controller节点；
*   通过调用其他模块的API，来主动的收集模块的相关数据（eg:image、volume），需要定期轮询来获取监控数据。

### [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#ceilometer-agent-notification "ceilometer-agent-notification")ceilometer-agent-notification

*   openstack各个组件都会推送通知（notification）信息到oslo-messaging消息框架；
*   ceilometer-agent-notification实现访问消息队列，获取相关通知并做一定的转换成采样数据的格式；
*   需要一直监听oslo-messaging消息队列，可以看成是被动的监控数据；

### [](https://ly798.github.io/2015/07/07/ceilometer-%E6%A6%82%E8%BF%B0/#u6D88_u606F_u53D1_u5E03 "消息发布")消息发布

网络上的一张图：
![](ceilometer-publish.jpg)

对应上图来看

1.  当agent监控完成之后，会进行信息的发布，有三种方式：
    *   RPC 会将信息发布到消息队列；
    *   UDP 会建立一个socket信息通道，发布信息；
    *   FILE 直接将信息保存到文件；
2.  ceilometer-collector通过对应的方式去取得信息，并保存到数据存储中；
3.  数据存储系统有 mongodb/mysql/hbase/db2等；
