---
title: ceilometer-alarm 源码
tags: [openstack, ceilometer]
date: 2015-07-25 10:41:01
---

## [](https://ly798.github.io/2015/07/25/ceilometer-alarm-%E6%BA%90%E7%A0%81/#u4ECB_u7ECD "介绍")介绍

以一个alarm为例
<!-- more -->
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
<span class="line">23</span>
<span class="line">24</span>
</pre></td><td class="code"><pre><span class="line">+---------------------------|<span class="string">-----------------------------------------------------+</span>
<span class="line"></span>|<span class="string"> Property </span>|<span class="string"> Value </span>|</span>
<span class="line">+---------------------------|<span class="string">-----------------------------------------------------+</span>
<span class="line"></span>|<span class="string"> alarm_actions </span>|<span class="string"> [u'log://'] </span>|</span>
<span class="line">|<span class="string"> alarm_id </span>|<span class="string"> a020c017-b2a9-422e-b1bc-c414c98091d1 </span>|</span>
<span class="line">|<span class="string"> comparison_operator </span>|<span class="string"> gt </span>|</span>
<span class="line">|<span class="string"> description </span>|<span class="string"> instance running hot </span>|</span>
<span class="line">|<span class="string"> enabled </span>|<span class="string"> True </span>|</span>
<span class="line">|<span class="string"> evaluation_periods </span>|<span class="string"> 3 </span>|</span>
<span class="line">|<span class="string"> exclude_outliers </span>|<span class="string"> False </span>|</span>
<span class="line">|<span class="string"> insufficient_data_actions </span>|<span class="string"> [] </span>|</span>
<span class="line">|<span class="string"> meter_name </span>|<span class="string"> cpu_util </span>|</span>
<span class="line">|<span class="string"> name </span>|<span class="string"> cpu_high </span>|</span>
<span class="line">|<span class="string"> ok_actions </span>|<span class="string"> [] </span>|</span>
<span class="line">|<span class="string"> period </span>|<span class="string"> 600 </span>|</span>
<span class="line">|<span class="string"> project_id </span>|<span class="string"> </span>|</span>
<span class="line">|<span class="string"> query </span>|<span class="string"> resource_id == 0da3f008-60d7-4b10-a7eb-b143a842a25b </span>|</span>
<span class="line">|<span class="string"> repeat_actions </span>|<span class="string"> False </span>|</span>
<span class="line">|<span class="string"> state </span>|<span class="string"> ok </span>|</span>
<span class="line">|<span class="string"> statistic </span>|<span class="string"> avg </span>|</span>
<span class="line">|<span class="string"> threshold </span>|<span class="string"> 70.0 </span>|</span>
<span class="line">|<span class="string"> type </span>|<span class="string"> threshold </span>|</span>
<span class="line">|<span class="string"> user_id </span>|<span class="string"> 3dbf0919d60d4025842e6ea149e4aeba </span>|</span>
<span class="line">+---------------------------|<span class="string">-----------------------------------------------------+</span></span>
</pre></td></tr></table></figure> 

此alarm为threshold类型，其方法调用在
ceilometer/alarm/evaluator/threshold.py
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">evaluate</span><span class="params">(self, alarm)</span>:</span></span>
<span class="line"> <span class="comment">#判断是否在包含的时间内</span></span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> self.within_time_constraint(alarm):</span>
<span class="line"> LOG.debug(_(<span class="string">'Attempted to evaluate alarm %s, but it is not '</span></span>
<span class="line"> <span class="string">'within its time constraint.'</span>) % alarm.alarm_id)</span>
<span class="line"> <span class="keyword">return</span></span>
<span class="line"></span>
<span class="line"> query = self._bound_duration(</span>
<span class="line"> alarm,</span>
<span class="line"> alarm.rule[<span class="string">'query'</span>]</span>
<span class="line"> )</span>
<span class="line"></span>
<span class="line"> statistics = self._sanitize(</span>
<span class="line"> alarm,</span>
<span class="line"> self._statistics(alarm, query)</span>
<span class="line"> )</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> self._sufficient(alarm, statistics):</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_compare</span><span class="params">(stat)</span>:</span></span>
<span class="line"> op = COMPARATORS[alarm.rule[<span class="string">'comparison_operator'</span>]]</span>
<span class="line"> value = getattr(stat, alarm.rule[<span class="string">'statistic'</span>])</span>
<span class="line"> limit = alarm.rule[<span class="string">'threshold'</span>]</span>
<span class="line"> LOG.debug(_(<span class="string">'comparing value %(value)s against threshold'</span></span>
<span class="line"> <span class="string">' %(limit)s'</span>) %</span>
<span class="line"> &#123;<span class="string">'value'</span>: value, <span class="string">'limit'</span>: limit&#125;)</span>
<span class="line"> <span class="keyword">return</span> op(value, limit)</span>
<span class="line"></span>
<span class="line"> self._transition(alarm,</span>
<span class="line"> statistics,</span>
<span class="line"> map(_compare, statistics))</span>
</pre></td></tr></table></figure>

先分析
<figure class="highlight perl"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">query = self._bound_duration(</span>
<span class="line"> <span class="keyword">alarm</span>,</span>
<span class="line"> <span class="keyword">alarm</span>.rule[<span class="string">'query'</span>]</span>
<span class="line"> )</span>
</pre></td></tr></table></figure>

这里的alarm.rule[‘query’]为resource_id == 0da3f008-60d7-4b10-a7eb-b143a842a25b

在查看函数_bound_duration
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
</pre></td><td class="code"><pre><span class="line"><span class="decorator">@classmethod</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_bound_duration</span><span class="params">(cls, alarm, constraints)</span>:</span></span>
<span class="line"> <span class="string">"""Bound the duration of the statistics query."""</span></span>
<span class="line"> now = timeutils.utcnow()</span>
<span class="line"> <span class="comment"># when exclusion of weak datapoints is enabled, we extend</span></span>
<span class="line"> <span class="comment"># the look-back period so as to allow a clearer sample count</span></span>
<span class="line"> <span class="comment"># trend to be established</span></span>
<span class="line"> look_back = (cls.look_back <span class="keyword">if</span> <span class="keyword">not</span> alarm.rule.get(<span class="string">'exclude_outliers'</span>)</span>
<span class="line"> <span class="keyword">else</span> alarm.rule[<span class="string">'evaluation_periods'</span>])</span>
<span class="line"></span>
<span class="line"> window = (alarm.rule[<span class="string">'period'</span>] *</span>
<span class="line"> (alarm.rule[<span class="string">'evaluation_periods'</span>] + look_back))</span>
<span class="line"> start = now - datetime.timedelta(seconds=window)</span>
<span class="line"> LOG.debug(_(<span class="string">'query stats from %(start)s to '</span></span>
<span class="line"> <span class="string">'%(now)s'</span>) % &#123;<span class="string">'start'</span>: start, <span class="string">'now'</span>: now&#125;)</span>
<span class="line"> after = dict(field=<span class="string">'timestamp'</span>, op=<span class="string">'ge'</span>, value=start.isoformat())</span>
<span class="line"> before = dict(field=<span class="string">'timestamp'</span>, op=<span class="string">'le'</span>, value=now.isoformat())</span>
<span class="line"> constraints.extend([before, after])</span>
<span class="line"> <span class="keyword">return</span> constraints</span>
</pre></td></tr></table></figure>

这个函数主要计算evaluate的时间段;

*   结束时间即before为now;
*   开始时间为after即start为now – period*(evaluation_periods + lookback)

    *   evaluation_periods
    *   look_back:这个参数是reporting/ingestion的延迟时间，可以忽略的细节;*   然后将时间段这个过滤条件加到query中返回; 

接下来函数_sanitize
<figure class="highlight armasm"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="label">statistics</span> = <span class="keyword">self._sanitize(</span>
<span class="line"></span> alarm,</span>
<span class="line"> <span class="keyword">self._statistics(alarm, </span>query)</span>
<span class="line"> )</span>
</pre></td></tr></table></figure>

这个函数就是判断是否清洗数据，用方差的方式;
其中数据的来源是函数_statistics;
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">_statistics</span><span class="params">(self, alarm, query)</span>:</span></span>
<span class="line"> LOG.debug(_(<span class="string">'stats query %s'</span>) % query)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="keyword">return</span> self._client.statistics.list(</span>
<span class="line"> meter_name=alarm.rule[<span class="string">'meter_name'</span>], q=query,</span>
<span class="line"> period=alarm.rule[<span class="string">'period'</span>])</span>
<span class="line"> <span class="keyword">except</span> Exception:</span>
<span class="line"> LOG.exception(_(<span class="string">'alarm stats retrieval failed'</span>))</span>
<span class="line"> <span class="keyword">return</span> []</span>
</pre></td></tr></table></figure> 

这里穿进来的query就包括resource_id == 0da3f008-60d7-4b10-a7eb-b143a842a25b、[after,before];
这里的_client是调用的ceilometerclient/v2/client.py，通过分析，最终定位到ceilometerclient/v2/statistics.py中的list方法;
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">list</span><span class="params">(self, meter_name, q=None, period=None, groupby=None,</span>
<span class="line"> aggregates=None)</span>:</span></span>
<span class="line"> groupby = groupby <span class="keyword">or</span> []</span>
<span class="line"> aggregates = aggregates <span class="keyword">or</span> []</span>
<span class="line"> p = [<span class="string">'period=%s'</span> % period] <span class="keyword">if</span> period <span class="keyword">else</span> []</span>
<span class="line"> <span class="keyword">if</span> isinstance(groupby, six.string_types):</span>
<span class="line"> groupby = [groupby]</span>
<span class="line"> p.extend([<span class="string">'groupby=%s'</span> % g <span class="keyword">for</span> g <span class="keyword">in</span> groupby] <span class="keyword">if</span> groupby <span class="keyword">else</span> [])</span>
<span class="line"> p.extend(self._build_aggregates(aggregates))</span>
<span class="line"> <span class="keyword">return</span> self._list(options.build_url(</span>
<span class="line"> <span class="string">'/v2/meters/'</span> + meter_name + <span class="string">'/statistics'</span>,</span>
<span class="line"> q, p))</span>
</pre></td></tr></table></figure>

其中根据meter_name、perioed、query等构建一个url返回;
然后回到了ceilometerclient/common/base.py中的base.Manager的方法_list;
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
<span class="line">20</span>
<span class="line">21</span>
<span class="line">22</span>
<span class="line">23</span>
</pre></td><td class="code"><pre><span class="line">def _list(self, url, response_key=None, obj_class=None, body=None,</span>
<span class="line"> expect_single=False):</span>
<span class="line"> try:</span>
<span class="line"> resp = self<span class="class">.api</span><span class="class">.get</span>(url)</span>
<span class="line"> except exceptions<span class="class">.NotFound</span>:</span>
<span class="line"> raise exc<span class="class">.HTTPNotFound</span></span>
<span class="line"> <span class="keyword">if</span> not resp<span class="class">.content</span>:</span>
<span class="line"> raise exc<span class="class">.HTTPNotFound</span></span>
<span class="line"> <span class="tag">body</span> = resp.<span class="function"><span class="title">json</span><span class="params">()</span></span></span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> obj_class is None:</span>
<span class="line"> obj_class = self<span class="class">.resource_class</span></span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> response_key:</span>
<span class="line"> try:</span>
<span class="line"> data = <span class="tag">body</span>[response_key]</span>
<span class="line"> except KeyError:</span>
<span class="line"> return []</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> data = <span class="tag">body</span></span>
<span class="line"> <span class="keyword">if</span> expect_single:</span>
<span class="line"> data = [data]</span>
<span class="line"> return [<span class="function"><span class="title">obj_class</span><span class="params">(self, res, loaded=True)</span></span> <span class="keyword">for</span> res <span class="keyword">in</span> data <span class="keyword">if</span> res]</span>
</pre></td></tr></table></figure>

这个使用GET去查到了数据返回;
最后回到evaluate方法_sufficient
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
</pre></td><td class="code"><pre><span class="line"><span class="keyword">if</span> self._sufficient(alarm, statistics):</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_compare</span><span class="params">(stat)</span>:</span></span>
<span class="line"> op = COMPARATORS[alarm.rule[<span class="string">'comparison_operator'</span>]]</span>
<span class="line"> value = getattr(stat, alarm.rule[<span class="string">'statistic'</span>])</span>
<span class="line"> limit = alarm.rule[<span class="string">'threshold'</span>]</span>
<span class="line"> LOG.debug(_(<span class="string">'comparing value %(value)s against threshold'</span></span>
<span class="line"> <span class="string">' %(limit)s'</span>) %</span>
<span class="line"> &#123;<span class="string">'value'</span>: value, <span class="string">'limit'</span>: limit&#125;)</span>
<span class="line"> <span class="keyword">return</span> op(value, limit)</span>
<span class="line"></span>
<span class="line"> self._transition(alarm,</span>
<span class="line"> statistics,</span>
<span class="line"> map(_compare, statistics))</span>
</pre></td></tr></table></figure>

这里先判断数据是否充足;
不充足刷新alarm状态为insufficient data;
充足则执行下面的代码方法_transition;
<figure class="highlight pf"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">def _transition(<span class="literal">self</span>, alarm, statistics, compared):</span>
<span class="line"> <span class="comment">#是否alarm定义的所有statistics均满足</span></span>
<span class="line"> distilled = <span class="literal">all</span>(compared)</span>
<span class="line"> <span class="comment">#是否alarm定义的所有statistics均满足或均不满足</span></span>
<span class="line"> unequivocal = distilled or not <span class="literal">any</span>(compared)</span>
<span class="line"> <span class="comment">#alarm的当前状态是否为UNKNOWN</span></span>
<span class="line"> <span class="literal">unknown</span> = alarm.<span class="keyword">state</span> == evaluator.UNKNOWN</span>
<span class="line"> <span class="comment">#是否连续</span></span>
<span class="line"> continuous = alarm.repeat_actions</span>
<span class="line"> <span class="comment">#如果都满足或都不满足的时候，若是都满足，转换为alarm;若都不满足，则转换ok，为正常状态;</span></span>
<span class="line"> if unequivocal:</span>
<span class="line"> <span class="keyword">state</span> = evaluator.ALARM if distilled else evaluator.OK</span>
<span class="line"> reason, reason_data = <span class="literal">self</span>._reason(alarm, statistics,</span>
<span class="line"> distilled, <span class="keyword">state</span>)</span>
<span class="line"> <span class="comment">#如果当前状态与目标状态不同，则需要转换；另外，如果alarm的repeat_actions被设置，则状态相同也要进行转换</span></span>
<span class="line"> if alarm.<span class="keyword">state</span> != <span class="keyword">state</span> or continuous:</span>
<span class="line"> <span class="literal">self</span>._refresh(alarm, <span class="keyword">state</span>, reason, reason_data)</span>
<span class="line"> elif <span class="literal">unknown</span> or continuous:</span>
<span class="line"> trending_state = evaluator.ALARM if compared[-<span class="number">1</span>] else evaluator.OK</span>
<span class="line"> <span class="keyword">state</span> = trending_state if <span class="literal">unknown</span> else alarm.<span class="keyword">state</span></span>
<span class="line"> reason, reason_data = <span class="literal">self</span>._reason(alarm, statistics,</span>
<span class="line"> distilled, <span class="keyword">state</span>)</span>
<span class="line"> <span class="literal">self</span>._refresh(alarm, <span class="keyword">state</span>, reason, reason_data)</span>
</pre></td></tr></table></figure>

接下来看函数_refresh（/ceilometer/alarm/evaluator/**init**.py）
<figure class="highlight pf"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">def _refresh(<span class="literal">self</span>, alarm, <span class="keyword">state</span>, reason, reason_data):</span>
<span class="line"> <span class="string">""</span><span class="string">"Refresh alarm state."</span><span class="string">""</span></span>
<span class="line"> try:</span>
<span class="line"> previous = alarm.<span class="keyword">state</span></span>
<span class="line"> if previous != <span class="keyword">state</span>:</span>
<span class="line"> LOG.info(_('alarm %(id)s transitioning <span class="keyword">to</span> %(<span class="keyword">state</span>)s because '</span>
<span class="line"> '%(reason)s') % &#123;'id': alarm.alarm_id,</span>
<span class="line"> '<span class="keyword">state</span>': <span class="keyword">state</span>,</span>
<span class="line"> 'reason': reason&#125;)</span>
<span class="line"></span>
<span class="line"> <span class="literal">self</span>._client.alarms.set_state(alarm.alarm_id, <span class="keyword">state</span>=<span class="keyword">state</span>)</span>
<span class="line"> alarm.<span class="keyword">state</span> = <span class="keyword">state</span></span>
<span class="line"> if <span class="literal">self</span>.notifier:</span>
<span class="line"> <span class="literal">self</span>.notifier.notify(alarm, previous, reason, reason_data)</span>
<span class="line"> except Exception:</span>
<span class="line"> <span class="comment"># retry will occur naturally on the next evaluation</span></span>
<span class="line"> <span class="comment"># cycle (unless alarm state reverts in the meantime)</span></span>
<span class="line"> LOG.exception(_('alarm <span class="keyword">state</span> update failed'))</span>
</pre></td></tr></table></figure>

这里才是真正设置状态的地方，对比之前和现在的状态，不一样则改之，反之;