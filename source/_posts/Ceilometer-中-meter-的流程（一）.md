---
title: Ceilometer 中 meter 的流程（一）
tags: [openstack, ceilometer]
date: 2015-12-26 10:37:41
---

## [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#u4ECB_u7ECD "介绍")介绍

在 ceilometer 中一个 meter 的生命周期可以大概归纳为：

1.  meter 的采集
2.  meter 的存储
3.  meter 的查询 <!-- more --> 

这一篇写的是 meter 的采集。

## [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#meter_u7684_u91C7_u96C6 "meter的采集")meter的采集

以 agent_compute 为例，采集 meter 整个流程大概如下：

*   ceilometer-agent-compute 服务初始化
*   初始化 pollster manager，即系统中实现的采集扩展（extensions）
*   初始化 discovery manager，用于获取资源，这里就指的是云主机
*   初始化 inspector manager，即 meter 数据的采集器（调用Hypervisor的API），其中实现了具体的数据收集
*   获取 pipeline manager，即配置文件 pipeline.yram 中定义的采集项，及其采集周期和数据发送方式
*   以 pipelines 和 extensions 的笛卡尔积产生 polling task，并放入线程池
*   每一个 task 中，调用 get_samples() 收集到数据，通过 publisher 发送出去
*   get_samples() 调用 inspector manager 中对应的采集器获取数据 

如图：
![ceilometer-agent-compute](ceilometer-agent-compute.png)

### [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#stevedore "stevedore")stevedore

借助 steuptools 的 entry points，可以方便的管理应用程序插件，动态的加载类，三种方式：

1.  Drivers – Single Name, Single Entry Point
2.  Hooks – Single Name, Many Entry Points
3.  Extensions – Many Names, Many Entry Points 

ceilometer 使用了 Extensions 的方式，包括 pollster manager、discovery manager、inspector manager，以`pollster manager`为例：

ceilometer/agent.py
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">AgentManager</span><span class="params">(os_service.Service)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, namespace, default_discovery=None, group_prefix=None)</span>:</span></span>
<span class="line"> self.pollster_manager = self._extensions(<span class="string">'poll'</span>, namespace)</span>
<span class="line"> ...</span>
<span class="line"><span class="decorator"> @staticmethod</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_extensions</span><span class="params">(category, agent_ns=None)</span>:</span></span>
<span class="line"> namespace = (<span class="string">'ceilometer.%s.%s'</span> % (category, agent_ns) <span class="keyword">if</span> agent_ns</span>
<span class="line"> <span class="keyword">else</span> <span class="string">'ceilometer.%s'</span> % category)</span>
<span class="line"> <span class="keyword">return</span> extension.ExtensionManager(</span>
<span class="line"> namespace=namespace,</span>
<span class="line"> invoke_on_load=<span class="keyword">True</span>,</span>
<span class="line"> )</span>
</pre></td></tr></table></figure>

这里的 namespace 是`ceilometer.poll.compute`，对应到 entry_point 文件中是
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line">ceilometer<span class="class">.poll</span><span class="class">.compute</span> =</span>
<span class="line"> disk<span class="class">.read</span><span class="class">.requests</span> = ceilometer<span class="class">.compute</span><span class="class">.pollsters</span><span class="class">.disk</span>:ReadRequestsPollster</span>
<span class="line"> disk<span class="class">.write</span><span class="class">.requests</span> = ceilometer<span class="class">.compute</span><span class="class">.pollsters</span><span class="class">.disk</span>:WriteRequestsPollster</span>
<span class="line"> disk<span class="class">.read</span><span class="class">.bytes</span> = ceilometer<span class="class">.compute</span><span class="class">.pollsters</span><span class="class">.disk</span>:ReadBytesPollster</span>
<span class="line"> ...</span>
</pre></td></tr></table></figure>

最后返回`self.pollster_manager`是一个`extension.ExtensionManager`，其中包含多个采集扩展的实例。

## [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#u6D41_u7A0B "流程")流程

### [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#u542F_u52A8_u670D_u52A1 "启动服务")启动服务

ceilometer/cmd/agent_compute.py 中的`main()`做最终调用 ceilometer/agent.py 中类 AgentManager 的`start()`
<figure class="highlight sql"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">def <span class="operator"><span class="keyword">start</span>(<span class="keyword">self</span>):</span>
<span class="line"> <span class="keyword">self</span>.pipeline_manager = publish_pipeline.setup_pipeline()</span>
<span class="line"></span>
<span class="line"> <span class="keyword">self</span>.partition_coordinator.<span class="keyword">start</span>()</span>
<span class="line"> <span class="keyword">self</span>.join_partitioning_groups()</span>
<span class="line"></span>
<span class="line"> # <span class="keyword">allow</span> <span class="keyword">time</span> <span class="keyword">for</span> coordination <span class="keyword">if</span> necessary</span>
<span class="line"> delay_start = <span class="keyword">self</span>.partition_coordinator.is_active()</span>
<span class="line"></span>
<span class="line"> <span class="keyword">for</span> <span class="built_in">interval</span>, task <span class="keyword">in</span> six.iteritems(<span class="keyword">self</span>.setup_polling_tasks()):</span>
<span class="line"> <span class="keyword">self</span>.tg.add_timer(<span class="built_in">interval</span>,</span>
<span class="line"> <span class="keyword">self</span>.interval_task,</span>
<span class="line"> initial_delay=<span class="built_in">interval</span> <span class="keyword">if</span> delay_start <span class="keyword">else</span> <span class="keyword">None</span>,</span>
<span class="line"> task=task)</span>
<span class="line"> <span class="keyword">self</span>.tg.add_timer(cfg.CONF.coordination.heartbeat,</span>
<span class="line"> <span class="keyword">self</span>.partition_coordinator.heartbeat)</span></span>
</pre></td></tr></table></figure>

*   publish_pipeline.setup_pipeline()，读取配置文件 pipeline.yaml，返回 PipelineManager 实例
*   self.setup_polling_tasks()
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">setup_polling_tasks</span><span class="params">(self)</span>:</span></span>
<span class="line"> polling_tasks = &#123;&#125;</span>
<span class="line"> <span class="keyword">for</span> pipeline, pollster <span class="keyword">in</span> itertools.product(</span>
<span class="line"> self.pipeline_manager.pipelines,</span>
<span class="line"> self.pollster_manager.extensions):</span>
<span class="line"> <span class="keyword">if</span> pipeline.support_meter(pollster.name):</span>
<span class="line"> polling_task = polling_tasks.get(pipeline.get_interval())</span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> polling_task:</span>
<span class="line"> polling_task = self.create_polling_task()</span>
<span class="line"> polling_tasks[pipeline.get_interval()] = polling_task</span>
<span class="line"> polling_task.add(pollster, pipeline)</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> polling_tasks</span>
</pre></td></tr></table></figure>
    pipelines 与 pollster 笛卡尔积，判断 pipelines 中是否包括对 pollster 中的 meter 的采集配置，若支持，添加到 polling_task，而 polling_task 在 polling_tasks 以轮询时间为 key 来形成字典，形如 <figure class="highlight css"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="rules">&#123;<span class="rule"><span class="attribute">interval</span>:<span class="value">polling_task.<span class="function">add</span>(pollster, pipeline)</span></span></span>&#125;</span>
</pre></td></tr></table></figure>*   遍历 polling_tasks，添加到线程池 

### [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#polling_task "polling_task")polling_task

具体的单个 polling_task 的运行
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="decorator">@staticmethod</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">interval_task</span><span class="params">(task)</span>:</span></span>
<span class="line"> task.poll_and_publish()</span>
</pre></td></tr></table></figure> <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">PollingTask</span><span class="params">(object)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">poll_and_publish</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="string">"""Polling sample and publish into pipeline."""</span></span>
<span class="line"> agent_resources = self.manager.discover()</span>
<span class="line"> cache = &#123;&#125;</span>
<span class="line"> discovery_cache = &#123;&#125;</span>
<span class="line"> <span class="keyword">for</span> source, pollster <span class="keyword">in</span> self.pollster_matches:</span>
<span class="line"> pollster_resources = <span class="keyword">None</span></span>
<span class="line"> <span class="keyword">if</span> pollster.obj.default_discovery:</span>
<span class="line"> pollster_resources = self.manager.discover(</span>
<span class="line"> [pollster.obj.default_discovery], discovery_cache)</span>
<span class="line"> key = Resources.key(source, pollster)</span>
<span class="line"> source_resources = list(self.resources[key].get(discovery_cache))</span>
<span class="line"> <span class="keyword">with</span> self.publishers[source.name] <span class="keyword">as</span> publisher:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> samples = list(pollster.obj.get_samples(</span>
<span class="line"> manager=self.manager,</span>
<span class="line"> cache=cache,</span>
<span class="line"> resources=(source_resources <span class="keyword">or</span></span>
<span class="line"> pollster_resources <span class="keyword">or</span></span>
<span class="line"> agent_resources)</span>
<span class="line"> ))</span>
<span class="line"> publisher(samples)</span>
<span class="line"> ...</span>
</pre></td></tr></table></figure> 

*   关于这里的`resources=(source_resources or pollster_resources or agent_resources)`，resources 是一个包含了 vm 及其信息的 list，第一次运行的时候等于 agent_resources。
*   这里使用提供的 discover（compute_agent 提供的是 local_instances）去匹配 discover manager 中的 discover，返回一个其 discover 实例，即 ceilometer/compute/discover.py:InstanceDiscovery 实例，返回其实例方法 discover() 得到的 vms。值得注意的是 ceilometer/compute/discover.py:InstanceDiscovery
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">discover</span><span class="params">(self, manager, param=None)</span>:</span></span>
<span class="line"> <span class="string">"""Discover resources to monitor."""</span></span>
<span class="line"> instances = self.nova_cli.instance_get_all_by_host(cfg.CONF.host)</span>
<span class="line"> <span class="keyword">return</span> [i <span class="keyword">for</span> i <span class="keyword">in</span> instances</span>
<span class="line"> <span class="keyword">if</span> getattr(i, <span class="string">'OS-EXT-STS:vm_state'</span>, <span class="keyword">None</span>) != <span class="string">'error'</span>]</span>
</pre></td></tr></table></figure> 

如果 vm 的 status 是 error 的不返回。

### [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#u5355_u4E2A_meter "单个 meter")单个 meter

以 cpu 为例，上节的 poll_and_publish() 中的 pollster.obj.get_samples()

ceilometer/compute/pollster/cpu.py:CPUPollster
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">get_samples</span><span class="params">(self, manager, cache, resources)</span>:</span></span>
<span class="line"> <span class="keyword">for</span> instance <span class="keyword">in</span> resources:</span>
<span class="line"> LOG.debug(_(<span class="string">'checking instance %s'</span>), instance.id)</span>
<span class="line"> instance_name = util.instance_name(instance)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> cpu_info = manager.inspector.inspect_cpus(instance_name)</span>
<span class="line"> LOG.debug(_(<span class="string">"CPUTIME USAGE: %(instance)s %(time)d"</span>),</span>
<span class="line"> &#123;<span class="string">'instance'</span>: instance.__dict__,</span>
<span class="line"> <span class="string">'time'</span>: cpu_info.time&#125;)</span>
<span class="line"> cpu_num = &#123;<span class="string">'cpu_number'</span>: cpu_info.number&#125;</span>
<span class="line"> <span class="keyword">yield</span> util.make_sample_from_instance(</span>
<span class="line"> instance,</span>
<span class="line"> name=<span class="string">'cpu'</span>,</span>
<span class="line"> type=sample.TYPE_CUMULATIVE,</span>
<span class="line"> unit=<span class="string">'ns'</span>,</span>
<span class="line"> volume=cpu_info.time,</span>
<span class="line"> additional_metadata=cpu_num,</span>
<span class="line"> )</span>
<span class="line"> ...</span>
</pre></td></tr></table></figure> 

*   循环 resources（即 vms），调用 inspector manager 中其对应的 inspector 实例的 inspect_cpus() 获取 cpu 时长，具体实现调用了 libvirt 的 API：
ceilometer/compute/virt/libvirt/inspector.py:LibvirtInspector
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">inspect_cpus</span><span class="params">(self, instance_name)</span>:</span></span>
<span class="line"> domain = self._lookup_by_name(instance_name)</span>
<span class="line"> dom_info = domain.info()</span>
<span class="line"> <span class="keyword">return</span> virt_inspector.CPUStats(number=dom_info[<span class="number">3</span>], time=dom_info[<span class="number">4</span>])</span>
</pre></td></tr></table></figure>*   得到所有 vms 的 cpu 使用时长数据，使用 publisher(samples) 

## [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#u603B_u7ED3 "总结")总结

## [](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/#u53C2_u8003 "参考")参考

stevedore：[http://docs.openstack.org/developer/stevedore/](http://docs.openstack.org/developer/stevedore/)
