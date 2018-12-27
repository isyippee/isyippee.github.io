---
title: Ceilometer 中 meter 的流程（二）
tags: [openstack, ceilometer]
date: 2015-12-30  10:38:00
---

## [](https://ly798.github.io/2015/12/30/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/#u4ECB_u7ECD "介绍")介绍

接着上一篇 [meter 的采集](https://ly798.github.io/2015/12/26/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%B8%80%EF%BC%89/)，这一篇写的是 meter 的存储。
 <!-- more --> 

## [](https://ly798.github.io/2015/12/30/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/#meter__u7684_u5B58_u50A8 "meter 的存储")meter 的存储

存储 meter 整个流程大概如下：

*   agent 收集到 meter 的 samples，进行 push
*   push时，对 samples 进行 transform
*   获得 pipeline 的 publisher，进行 publish_samples 

## [](https://ly798.github.io/2015/12/30/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/#u6D41_u7A0B "流程")流程

### [](https://ly798.github.io/2015/12/30/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/#publisher "publisher")publisher

接上一节，在方法 poll_and_publish() 中，使用 with as 来调用 publisher()
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
</pre></td><td class="code"><pre><span class="line"><span class="keyword">with</span> self.publishers[source.name] <span class="keyword">as</span> publisher:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> samples = list(pollster.obj.get_samples(</span>
<span class="line"> manager=self.manager,</span>
<span class="line"> cache=cache,</span>
<span class="line"> resources=(source_resources <span class="keyword">or</span></span>
<span class="line"> pollster_resources <span class="keyword">or</span></span>
<span class="line"> agent_resources)</span>
<span class="line"> ))</span>
<span class="line"> publisher(samples)</span>
</pre></td></tr></table></figure>

这里的publisher就是类PublishContext
ceilometer/pipeline.py:PublishContext
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">PublishContext</span><span class="params">(object)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__enter__</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">p</span><span class="params">(samples)</span>:</span></span>
<span class="line"> <span class="keyword">for</span> p <span class="keyword">in</span> self.pipelines:</span>
<span class="line"> p.publish_samples(self.context,</span>
<span class="line"> samples)</span>
<span class="line"> <span class="keyword">return</span> p</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__exit__</span><span class="params">(self, exc_type, exc_value, traceback)</span>:</span></span>
<span class="line"> <span class="keyword">for</span> p <span class="keyword">in</span> self.pipelines:</span>
<span class="line"> p.flush(self.context)</span>
</pre></td></tr></table></figure>

在方法 __enter__() 中 self.context 维护 admin 用户的认证信息；p 是 ceilometer/pipeline.py:Pipeline 的实例
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Pipeline</span><span class="params">(object)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">publish_samples</span><span class="params">(self, ctxt, samples)</span>:</span></span>
<span class="line"> supported = [s <span class="keyword">for</span> s <span class="keyword">in</span> samples <span class="keyword">if</span> self.source.support_meter(s.name)]</span>
<span class="line"> self.sink.publish_samples(ctxt, supported)</span>
<span class="line"> ...</span>
</pre></td></tr></table></figure> 

sink 维护 source 及其对应的 publishers
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Sink</span><span class="params">(object)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_publish_samples</span><span class="params">(self, start, ctxt, samples)</span>:</span></span>
<span class="line"> transformed_samples = []</span>
<span class="line"> <span class="keyword">for</span> sample <span class="keyword">in</span> samples:</span>
<span class="line"> sample = self._transform_sample(start, ctxt, sample)</span>
<span class="line"> <span class="keyword">if</span> sample:</span>
<span class="line"> transformed_samples.append(sample)</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> transformed_samples:</span>
<span class="line"> <span class="keyword">for</span> p <span class="keyword">in</span> self.publishers:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> p.publish_samples(ctxt, transformed_samples)</span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">publish_samples</span><span class="params">(self, ctxt, samples)</span>:</span></span>
<span class="line"> <span class="keyword">for</span> meter_name, samples <span class="keyword">in</span> itertools.groupby(</span>
<span class="line"> sorted(samples, key=operator.attrgetter(<span class="string">'name'</span>)),</span>
<span class="line"> operator.attrgetter(<span class="string">'name'</span>)):</span>
<span class="line"> self._publish_samples(<span class="number">0</span>, ctxt, samples)</span>
</pre></td></tr></table></figure> 

*   先对 samples 根据 meter_name 进行升序排列，再根据 meter_name 进行分组
*   方法 _publish_samples() 中将 samples 进行 transform，然后遍历 publishers 再分别进行 publish
*   publishers 的初始化<figure class="highlight ruby"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">self</span>.publishers = []</span>
<span class="line"> <span class="keyword">for</span> p <span class="keyword">in</span> cfg[<span class="string">'publishers'</span>]<span class="symbol">:</span></span>
<span class="line"> <span class="keyword">if</span> <span class="string">'://'</span> <span class="keyword">not</span> <span class="keyword">in</span> <span class="symbol">p:</span></span>
<span class="line"> <span class="comment"># Support old format without URL</span></span>
<span class="line"> p = p + <span class="string">"://"</span></span>
<span class="line"> <span class="symbol">try:</span></span>
<span class="line"> <span class="keyword">self</span>.publishers.append(publisher.get_publisher(p))</span>
<span class="line"> except <span class="constant">Exception</span><span class="symbol">:</span></span>
<span class="line"> <span class="constant">LOG</span>.exception(<span class="number">_</span>(<span class="string">"Unable to load publisher %s"</span>), p)</span>
</pre></td></tr></table></figure> 

publisher的加载使用的是 stevedore 的 driver 方式，根据 pipeline.yarm 文件中的 publisher 的配置，使用的是 notifier，对应的是
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">ceilometer<span class="class">.publisher</span> =</span>
<span class="line">...</span>
<span class="line">notifier = ceilometer<span class="class">.publisher</span><span class="class">.messaging</span>:NotifierPublisher</span>
</pre></td></tr></table></figure>

使用 rabbitmq 的 topic 方式 push

## [](https://ly798.github.io/2015/12/30/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/#u603B_u7ED3 "总结")总结

## [](https://ly798.github.io/2015/12/30/Ceilometer-%E4%B8%AD-meter-%E7%9A%84%E6%B5%81%E7%A8%8B%EF%BC%88%E4%BA%8C%EF%BC%89/#u53C2_u8003 "参考")参考
