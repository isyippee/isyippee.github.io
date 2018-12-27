---
title: ceilometer-监控项扩展
tags: [openstack, ceilometer]
date: 2015-07-23 10:40:50
---

## [](https://ly798.github.io/2015/07/23/ceilometer-%E7%9B%91%E6%8E%A7%E9%A1%B9%E6%89%A9%E5%B1%95/#u7B80_u4ECB "简介")简介

![](ceilometer-extend.png)
 <!-- more -->
*   在compute节点通过polister轮询获取instance的监控数据，并通过publisher调用了RPC发送监控数据到消息队列;
*   collect从消息队列中匹配接受数据，并调用storage接口进行存储;

## [](https://ly798.github.io/2015/07/23/ceilometer-%E7%9B%91%E6%8E%A7%E9%A1%B9%E6%89%A9%E5%B1%95/#u6269_u5C55 "扩展")扩展

添加一个ceilometer-agent-compute监控项`mem.max`

/usr/lib/python2.7/site-packages/ceilometer-2014.2-py2.7.egg-info/entry_points.txt
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">[ceilometer<span class="class">.poll</span><span class="class">.compute</span>]</span>
<span class="line">...</span>
<span class="line">mem<span class="class">.max</span> = ceilometer<span class="class">.compute</span><span class="class">.pollsters</span><span class="class">.mem</span>:MaxMemPollster</span>
</pre></td></tr></table></figure>

ceilometer/compute/pollsters/mem.py
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
</pre></td><td class="code"><pre><span class="line"><span class="keyword">import</span> ceilometer</span>
<span class="line"><span class="keyword">from</span> ceilometer.compute <span class="keyword">import</span> plugin</span>
<span class="line"><span class="keyword">from</span> ceilometer.compute.pollsters <span class="keyword">import</span> util</span>
<span class="line"><span class="keyword">from</span> ceilometer.compute.virt <span class="keyword">import</span> inspector <span class="keyword">as</span> virt_inspector</span>
<span class="line"><span class="keyword">from</span> ceilometer.openstack.common.gettextutils <span class="keyword">import</span> _</span>
<span class="line"><span class="keyword">from</span> ceilometer.openstack.common <span class="keyword">import</span> log</span>
<span class="line"><span class="keyword">from</span> ceilometer <span class="keyword">import</span> sample</span>
<span class="line"></span>
<span class="line">LOG = log.getLogger(__name__)</span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">MaxMemPollster</span><span class="params">(plugin.ComputePollster)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">get_samples</span><span class="params">(self, manager, cache, resources)</span>:</span></span>
<span class="line"> <span class="keyword">for</span> instance <span class="keyword">in</span> resources:</span>
<span class="line"> LOG.debug(_(<span class="string">'checking instance %s'</span>), instance.id)</span>
<span class="line"> instance_name = util.instance_name(instance)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> mem_info = manager.inspector.inspect_mems(instance_name)</span>
<span class="line"> LOG.debug(_(<span class="string">"mem.max: %(instance)s %(max)d"</span>),</span>
<span class="line"> &#123;<span class="string">'instance'</span>: instance.__dict__,</span>
<span class="line"> <span class="string">'max'</span>: mem_info.max&#125;)</span>
<span class="line"> <span class="keyword">yield</span> util.make_sample_from_instance(</span>
<span class="line"> instance,</span>
<span class="line"> name=<span class="string">'mem.max'</span>,</span>
<span class="line"> type=sample.TYPE_GAUGE,</span>
<span class="line"> unit=<span class="string">'MB'</span>,</span>
<span class="line"> volume=mem_info.max,</span>
<span class="line"> )</span>
<span class="line"> <span class="keyword">except</span> virt_inspector.InstanceNotFoundException <span class="keyword">as</span> err:</span>
<span class="line"> <span class="comment"># Instance was deleted while getting samples. Ignore it.</span></span>
<span class="line"> LOG.debug(_(<span class="string">'Exception while getting samples %s'</span>), err)</span>
<span class="line"> <span class="keyword">except</span> ceilometer.NotImplementedError:</span>
<span class="line"> <span class="comment"># Selected inspector does not implement this pollster.</span></span>
<span class="line"> LOG.debug(_(<span class="string">'Obtaining mem is not implemented for %s'</span></span>
<span class="line"> ), manager.inspector.__class__.__name__)</span>
<span class="line"> <span class="keyword">except</span> Exception <span class="keyword">as</span> err:</span>
<span class="line"> LOG.exception(_(<span class="string">'could not get mem for %(id)s: %(e)s'</span>),</span>
<span class="line"> &#123;<span class="string">'id'</span>: instance.id, <span class="string">'e'</span>: err&#125;)</span>
</pre></td></tr></table></figure> 

ceilometer/compute/virt/inspector.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Inspector</span><span class="params">(object)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">inspect_mems</span><span class="params">(self,instance,duration=None)</span>:</span></span>
<span class="line"> <span class="keyword">raise</span> ceilometer.NotImplementedError</span>
<span class="line">MaxMem = collections.namedtuple(<span class="string">'MaxMem'</span>, [<span class="string">'max'</span>])</span>
</pre></td></tr></table></figure> 

ceilometer/compute/virt/libvirt/inspector.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">LibvirtInspector</span><span class="params">(virt_inspector.Inspector)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">inspect_mems</span><span class="params">(self, instance_name)</span>:</span></span>
<span class="line"> domain = self._lookup_by_name(instance_name)</span>
<span class="line"> maxMem = domain.maxMemory()</span>
<span class="line"> <span class="keyword">return</span> virt_inspector.MaxMem(max=maxMem,)</span>
</pre></td></tr></table></figure> 

重启服务即有新的收集数据了
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">ceilometer sample-<span class="built_in">list</span> -m maxMem -q resource_id=<span class="number">9</span>af11e66-<span class="number">30</span>ef-<span class="number">42</span>cf-<span class="number">8f</span>48-bc4bfb03cc03</span>
</pre></td></tr></table></figure>