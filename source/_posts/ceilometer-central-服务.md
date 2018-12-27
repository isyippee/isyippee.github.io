---
title: ceilometer-central 服务
tags: [openstack, ceilometer]
date: 2015-09-14 10:41:27
---

## [](https://ly798.github.io/2015/09/14/ceilometer-central-%E6%9C%8D%E5%8A%A1/#u5206_u6790 "分析")分析
<!-- more --> <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">main</span><span class="params">()</span>:</span></span>
<span class="line"> service.prepare_service()</span>
<span class="line"> os_service.launch(manager.AgentManager()).wait()</span>
</pre></td></tr></table></figure> 

第一步准备服务`service.prepare_service()`
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
</pre></td><td class="code"><pre><span class="line">def <span class="function"><span class="title">prepare_service</span><span class="params">(argv=None)</span></span>:</span>
<span class="line"> gettextutils.<span class="function"><span class="title">install</span><span class="params">(<span class="string">'ceilometer'</span>)</span></span></span>
<span class="line"> gettextutils.<span class="function"><span class="title">enable_lazy</span><span class="params">()</span></span> #定义国际化</span>
<span class="line"> log_levels = (cfg<span class="class">.CONF</span><span class="class">.default_log_levels</span> +</span>
<span class="line"> [<span class="string">'stevedore=INFO'</span>, <span class="string">'keystoneclient=INFO'</span>])</span>
<span class="line"> cfg.set_defaults(log<span class="class">.log_opts</span>,</span>
<span class="line"> default_log_levels=log_levels) #设置日志等级</span>
<span class="line"> <span class="keyword">if</span> argv is None:</span>
<span class="line"> argv = sys<span class="class">.argv</span></span>
<span class="line"> cfg.<span class="function"><span class="title">CONF</span><span class="params">(argv[<span class="number">1</span>:], project=<span class="string">'ceilometer'</span>)</span></span></span>
<span class="line"> log.<span class="function"><span class="title">setup</span><span class="params">(<span class="string">'ceilometer'</span>)</span></span></span>
<span class="line"> messaging.<span class="function"><span class="title">setup</span><span class="params">()</span></span> #设置默认消息队列</span>
</pre></td></tr></table></figure> 

第二步启动服务`os_service.launch(manager.AgentManager()).wait()`
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">launch</span><span class="params">(service, workers=<span class="number">1</span>)</span>:</span></span>
<span class="line"> <span class="keyword">if</span> workers <span class="keyword">is</span> <span class="keyword">None</span> <span class="keyword">or</span> workers == <span class="number">1</span>:</span>
<span class="line"> launcher = ServiceLauncher()</span>
<span class="line"> launcher.launch_service(service)</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> launcher = ProcessLauncher()</span>
<span class="line"> launcher.launch_service(service, workers=workers)</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> launcher</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">ServiceLauncher</span><span class="params">(Launcher)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">wait</span><span class="params">(self, ready_callback=None)</span>:</span></span>
<span class="line"> systemd.notify_once()</span>
<span class="line"> <span class="keyword">while</span> <span class="keyword">True</span>:</span>
<span class="line"> self.handle_signal()</span>
<span class="line"> status, signo = self._wait_for_exit_or_signal(ready_callback)</span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> _is_sighup_and_daemon(signo):</span>
<span class="line"> <span class="keyword">return</span> status</span>
<span class="line"> self.restart()</span>
</pre></td></tr></table></figure> 

最终是调用这里的wait()

`launcher.launch_service(service)`
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">launch_service</span><span class="params">(self, service)</span>:</span></span>
<span class="line"> <span class="string">"""Load and start the given service.</span>
<span class="line"></span>
<span class="line"> :param service: The service you would like to start.</span>
<span class="line"> :returns: None</span>
<span class="line"></span>
<span class="line"> """</span></span>
<span class="line"> service.backdoor_port = self.backdoor_port</span>
<span class="line"> self.services.add(service)</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Services</span><span class="params">(object)</span>:</span></span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self)</span>:</span></span>
<span class="line"> self.services = []</span>
<span class="line"> self.tg = threadgroup.ThreadGroup()</span>
<span class="line"> self.done = event.Event()</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">add</span><span class="params">(self, service)</span>:</span></span>
<span class="line"> self.services.append(service)</span>
<span class="line"> self.tg.add_thread(self.run_service, service, self.done)</span>
</pre></td></tr></table></figure> 

这里是将各个服务放到了线程组中。

ceilometer/ceilometer/agent.py
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
<span class="line">17</span>
<span class="line">18</span>
<span class="line">19</span>
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line">class AgentManager(os_service.Service):</span>
<span class="line"> ...</span>
<span class="line"> def <span class="operator"><span class="keyword">start</span>(<span class="keyword">self</span>):</span>
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

这里定时去之执行task，注意`self.interval_task`。

### [](https://ly798.github.io/2015/09/14/ceilometer-central-%E6%9C%8D%E5%8A%A1/#u5177_u4F53_u67D0_u9879_u7684_u91C7_u96C6 "具体某项的采集")具体某项的采集

举例
`image = ceilometer.image.glance:ImagePollster`

对应到ceilometer/image/glance.py
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
<span class="line">14</span>
<span class="line">15</span>
</pre></td><td class="code"><pre><span class="line">class <span class="function"><span class="title">ImagePollster</span><span class="params">(_Base)</span></span>:</span>
<span class="line"> def <span class="function"><span class="title">get_samples</span><span class="params">(self, manager, cache, resources)</span></span>:</span>
<span class="line"> <span class="keyword">for</span> endpoint <span class="keyword">in</span> resources:</span>
<span class="line"> <span class="keyword">for</span> image <span class="keyword">in</span> self._iter_images(manager<span class="class">.keystone</span>, cache, endpoint):</span>
<span class="line"> yield sample.Sample(</span>
<span class="line"> name=<span class="string">'image'</span>,</span>
<span class="line"> type=sample<span class="class">.TYPE_GAUGE</span>,</span>
<span class="line"> unit=<span class="string">'image'</span>,</span>
<span class="line"> volume=<span class="number">1</span>,</span>
<span class="line"> user_id=None,</span>
<span class="line"> project_id=image<span class="class">.owner</span>,</span>
<span class="line"> resource_id=image<span class="class">.id</span>,</span>
<span class="line"> timestamp=timeutils.<span class="function"><span class="title">isotime</span><span class="params">()</span></span>,</span>
<span class="line"> resource_metadata=self.<span class="function"><span class="title">extract_image_metadata</span><span class="params">(image)</span></span>,</span>
<span class="line"> )</span>
</pre></td></tr></table></figure> 

self._iter_images(manager.keystone, cache, endpoint)
这是一个生成器，其中根据endpoint建立glanceclient去获取image信息。
