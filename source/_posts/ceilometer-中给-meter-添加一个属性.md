---
title: ceilometer 中给 meter 添加一个属性
tags: [openstack, ceilometer]
date: 2015-09-14 10:41:43
---

## [](https://ly798.github.io/2015/09/14/ceilometer-%E4%B8%AD%E7%BB%99-meter-%E6%B7%BB%E5%8A%A0%E4%B8%80%E4%B8%AA%E5%B1%9E%E6%80%A7/#u6DFB_u52A0meter_u5C5E_u6027 "添加meter属性")添加meter属性

meter的volume最终会保存到数据库中，有时候希望meter能够有其他的属性。
例如给meter添加一个other的属性
<!-- more -->

ceilometer/sample.py
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
<span class="line">16</span>
<span class="line">17</span>
<span class="line">18</span>
<span class="line">19</span>
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line">class <span class="function"><span class="title">Sample</span><span class="params">(object)</span></span>:</span>
<span class="line"></span>
<span class="line"> def __init__(self, name, type, unit, volume, user_id, project_id,</span>
<span class="line"> resource_id, timestamp, resource_metadata, source=None,</span>
<span class="line"> other=<span class="string">'default'</span>):</span>
<span class="line"> self<span class="class">.name</span> = name</span>
<span class="line"> self<span class="class">.type</span> = type</span>
<span class="line"> self<span class="class">.unit</span> = unit</span>
<span class="line"> self<span class="class">.volume</span> = volume</span>
<span class="line"> self<span class="class">.user_id</span> = user_id</span>
<span class="line"> self<span class="class">.project_id</span> = project_id</span>
<span class="line"> self<span class="class">.resource_id</span> = resource_id</span>
<span class="line"> self<span class="class">.timestamp</span> = timestamp</span>
<span class="line"> self<span class="class">.resource_metadata</span> = resource_metadata</span>
<span class="line"> self<span class="class">.source</span> = source or cfg<span class="class">.CONF</span><span class="class">.sample_source</span></span>
<span class="line"> self<span class="class">.id</span> = <span class="function"><span class="title">str</span><span class="params">(uuid.uuid1()</span></span>)</span>
<span class="line"> self<span class="class">.other</span> = other</span>
<span class="line">...</span>
</pre></td></tr></table></figure> 

ceilometer/publisher/utils.py
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
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">meter_message_from_counter</span><span class="params">(sample, secret)</span>:</span></span>
<span class="line"> <span class="string">"""Make a metering message ready to be published or stored.</span>
<span class="line"></span>
<span class="line"> Returns a dictionary containing a metering message</span>
<span class="line"> for a notification message and a Sample instance.</span>
<span class="line"> """</span></span>
<span class="line"> msg = &#123;<span class="string">'source'</span>: sample.source,</span>
<span class="line"> <span class="string">'counter_name'</span>: sample.name,</span>
<span class="line"> <span class="string">'counter_type'</span>: sample.type,</span>
<span class="line"> <span class="string">'counter_unit'</span>: sample.unit,</span>
<span class="line"> <span class="string">'counter_volume'</span>: sample.volume,</span>
<span class="line"> <span class="string">'user_id'</span>: sample.user_id,</span>
<span class="line"> <span class="string">'project_id'</span>: sample.project_id,</span>
<span class="line"> <span class="string">'resource_id'</span>: sample.resource_id,</span>
<span class="line"> <span class="string">'timestamp'</span>: sample.timestamp,</span>
<span class="line"> <span class="string">'resource_metadata'</span>: sample.resource_metadata,</span>
<span class="line"> <span class="string">'message_id'</span>: sample.id,</span>
<span class="line"> <span class="string">'other'</span>: sample.other,</span>
<span class="line"> &#125;</span>
<span class="line"> msg[<span class="string">'message_signature'</span>] = compute_signature(msg, secret)</span>
<span class="line"> <span class="keyword">return</span> msg</span>
<span class="line">...</span>
</pre></td></tr></table></figure> 

这样收集的meter中就会有一个other的属性

但ceilometer api还会出问题，需要作如下修改
ceilometer/storage/models.py
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
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Sample</span><span class="params">(base.Model)</span>:</span></span>
<span class="line"> <span class="string">"""One collected data point."""</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self,</span>
<span class="line"> source,</span>
<span class="line"> counter_name, counter_type, counter_unit, counter_volume,</span>
<span class="line"> user_id, project_id, resource_id,</span>
<span class="line"> timestamp, resource_metadata,</span>
<span class="line"> message_id,</span>
<span class="line"> message_signature,</span>
<span class="line"> recorded_at,</span>
<span class="line"> other,</span>
<span class="line"> )</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> base.Model.__init__(self,</span>
<span class="line"> source=source,</span>
<span class="line"> counter_name=counter_name,</span>
<span class="line"> counter_type=counter_type,</span>
<span class="line"> counter_unit=counter_unit,</span>
<span class="line"> counter_volume=counter_volume,</span>
<span class="line"> user_id=user_id,</span>
<span class="line"> project_id=project_id,</span>
<span class="line"> resource_id=resource_id,</span>
<span class="line"> timestamp=timestamp,</span>
<span class="line"> resource_metadata=resource_metadata,</span>
<span class="line"> message_id=message_id,</span>
<span class="line"> message_signature=message_signature,</span>
<span class="line"> recorded_at=recorded_at,</span>
<span class="line"> other=other)</span>
</pre></td></tr></table></figure> 

ceilometer/api/controller/v2.py
 <figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line">class <span class="function"><span class="title">OldSample</span><span class="params">(_Base)</span></span>:</span>
<span class="line"> other = wtype<span class="class">.text</span></span>
<span class="line">...</span>
</pre></td></tr></table></figure> 

若需要定义other的值，举例ceilometer/compute
修改

ceilometer/compute/pollsters/utils.py
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
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">make_sample_from_instance</span><span class="params">(instance, name, type, unit, volume,</span>
<span class="line"> resource_id=None, additional_metadata=None,</span>
<span class="line"> other)</span>:</span></span>
<span class="line"> additional_metadata = additional_metadata <span class="keyword">or</span> &#123;&#125;</span>
<span class="line"> resource_metadata = _get_metadata_from_object(instance)</span>
<span class="line"> resource_metadata.update(additional_metadata)</span>
<span class="line"> <span class="keyword">return</span> sample.Sample(</span>
<span class="line"> name=name,</span>
<span class="line"> type=type,</span>
<span class="line"> unit=unit,</span>
<span class="line"> volume=volume,</span>
<span class="line"> user_id=instance.user_id,</span>
<span class="line"> project_id=instance.tenant_id,</span>
<span class="line"> resource_id=resource_id <span class="keyword">or</span> instance.id,</span>
<span class="line"> timestamp=timeutils.isotime(),</span>
<span class="line"> resource_metadata=resource_metadata,</span>
<span class="line"> other=other,</span>
<span class="line"> )</span>
<span class="line">...</span>
</pre></td></tr></table></figure>

ceilometer/compute/pollsters/memory.py
<figure class="highlight lasso"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line"><span class="attribute">...</span></span>
<span class="line"><span class="keyword">yield</span> util<span class="built_in">.</span>make_sample_from_instance(</span>
<span class="line"> instance,</span>
<span class="line"> name=<span class="string">'memory.usage'</span>,</span>
<span class="line"> <span class="keyword">type</span>=sample<span class="built_in">.</span>TYPE_GAUGE,</span>
<span class="line"> unit=<span class="string">'MB'</span>,</span>
<span class="line"> volume=memory_info<span class="built_in">.</span>usage,</span>
<span class="line"> other=xxxxxx,</span>
<span class="line"> )</span>
<span class="line"><span class="attribute">...</span></span>
</pre></td></tr></table></figure>
