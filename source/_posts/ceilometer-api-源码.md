---
title: ceilometer api 源码
tags: [openstack, ceilometer]
date: 2015-08-03 10:41:16
---

## [](https://ly798.github.io/2015/08/03/ceilometer-api-%E6%BA%90%E7%A0%81/#u4ECB_u7ECD "介绍")介绍

版本：juno
 <!-- more -->

## [](https://ly798.github.io/2015/08/03/ceilometer-api-%E6%BA%90%E7%A0%81/#u6D41_u7A0B "流程")流程

/usr/bin/ceilometer-api
 <figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">import sys</span>
<span class="line">from ceilometer<span class="class">.cmd</span><span class="class">.api</span> import main</span>
<span class="line"><span class="keyword">if</span> __name__ == <span class="string">"__main__"</span>:</span>
<span class="line"> sys.<span class="function"><span class="title">exit</span><span class="params">(main()</span></span>)</span>
</pre></td></tr></table></figure>

cmd/api.py
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">main</span><span class="params">()</span>:</span></span>
<span class="line"> service.prepare_service()</span>
<span class="line"> srv = app.build_server()</span>
<span class="line"> srv.serve_forever()</span>
</pre></td></tr></table></figure>

api/app.py
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">build_server</span><span class="params">()</span>:</span></span>
<span class="line"> app = load_app()</span>
<span class="line"> ...</span>
<span class="line"> srv = simple_server.make_server(host, port, app,</span>
<span class="line"> server_cls, get_handler_cls())</span>
<span class="line"> ...</span>
<span class="line"> <span class="keyword">return</span> srv</span>
</pre></td></tr></table></figure>
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">load_app</span><span class="params">()</span>:</span></span>
<span class="line"> <span class="comment"># Build the WSGI app</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="keyword">return</span> deploy.loadapp(<span class="string">"config:"</span> + cfg_file)</span>
</pre></td></tr></table></figure> 

api_paste.ini对应app_factory
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">app_factory</span><span class="params">(global_config, **local_conf)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> VersionSelectorApplication()</span>
</pre></td></tr></table></figure>
 <figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">class <span class="function"><span class="title">VersionSelectorApplication</span><span class="params">(object)</span></span>:</span>
<span class="line"> ...</span>
<span class="line"> self<span class="class">.v2</span> = <span class="function"><span class="title">setup_app</span><span class="params">(pecan_config=pc)</span></span></span>
<span class="line"> ...</span>
</pre></td></tr></table></figure> <figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">def <span class="function"><span class="title">setup_app</span><span class="params">(pecan_config=None, extra_hooks=None)</span></span>:</span>
<span class="line"> ...</span>
<span class="line"> app = pecan.make_app(</span>
<span class="line"> pecan_config<span class="class">.app</span><span class="class">.root</span>,</span>
<span class="line"> static_root=pecan_config<span class="class">.app</span><span class="class">.static_root</span>,</span>
<span class="line"> template_path=pecan_config<span class="class">.app</span><span class="class">.template_path</span>,</span>
<span class="line"> debug=CONF<span class="class">.api</span><span class="class">.pecan_debug</span>,</span>
<span class="line"> force_canonical=<span class="function"><span class="title">getattr</span><span class="params">(pecan_config.app, <span class="string">'force_canonical'</span>, True)</span></span>,</span>
<span class="line"> hooks=app_hooks,</span>
<span class="line"> wrap_app=middleware<span class="class">.ParsableErrorMiddleware</span>,</span>
<span class="line"> guess_content_type_from_ext=False</span>
<span class="line"> )</span>
<span class="line"> return app</span>
</pre></td></tr></table></figure> 

这里用到了pecan。
pecan，一个轻量的web框架，主页[http://pecan.readthedocs.org/](http://pecan.readthedocs.org/)
ceilometer中主要是用来构建restapi，主页中关于restapi部分的[介绍](http://pecan.readthedocs.org/en/latest/rest.html#writing-restful-web-services-with-restcontroller),需要继承rest.RestController；
其中重要的部分URL Mapping：
![](pecanapi.jpg)

pecan的配置文件api/config.py
<figure class="highlight prolog"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
</pre></td><td class="code"><pre><span class="line">...</span>
<span class="line"><span class="atom">app</span> = &#123;</span>
<span class="line"> <span class="string">'root'</span>: <span class="string">'ceilometer.api.controllers.root.RootController'</span>,</span>
<span class="line"> <span class="string">'modules'</span>: [<span class="string">'ceilometer.api'</span>],</span>
<span class="line"> <span class="string">'static_root'</span>: <span class="string">'%(confdir)s/public'</span>,</span>
<span class="line"> <span class="string">'template_path'</span>: <span class="string">'%(confdir)s/ceilometer/api/templates'</span>,</span>
<span class="line">&#125;</span>
<span class="line">...</span>
</pre></td></tr></table></figure>

ceilometer.api.controllers.root.RootController.py
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">RootController</span><span class="params">(object)</span>:</span></span>
<span class="line"></span>
<span class="line"> v2 = v2.V2Controller()</span>
<span class="line"></span>
<span class="line"><span class="decorator"> @pecan.expose(generic=True, template='index.html')</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">index</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="comment"># <span class="doctag">FIXME:</span> Return version information</span></span>
<span class="line"> <span class="keyword">return</span> dict()</span>
</pre></td></tr></table></figure>

这里注意`v2 = v2.V2Controller()`，其中就写了很多的restapi
api/controllers/v2.py
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
</pre></td><td class="code"><pre><span class="line">class <span class="function"><span class="title">V2Controller</span><span class="params">(object)</span></span>:</span>
<span class="line"> <span class="string">""</span><span class="string">"Version 2 API controller root."</span><span class="string">""</span></span>
<span class="line"> resources = <span class="function"><span class="title">ResourcesController</span><span class="params">()</span></span></span>
<span class="line"> meters = <span class="function"><span class="title">MetersController</span><span class="params">()</span></span></span>
<span class="line"> samples = <span class="function"><span class="title">SamplesController</span><span class="params">()</span></span></span>
<span class="line"> alarms = <span class="function"><span class="title">AlarmsController</span><span class="params">()</span></span></span>
<span class="line"> event_types = <span class="function"><span class="title">EventTypesController</span><span class="params">()</span></span></span>
<span class="line"> events = <span class="function"><span class="title">EventsController</span><span class="params">()</span></span></span>
<span class="line"> query = <span class="function"><span class="title">QueryController</span><span class="params">()</span></span></span>
<span class="line"> capabilities = <span class="function"><span class="title">CapabilitiesController</span><span class="params">()</span></span></span>
</pre></td></tr></table></figure>

这里又加载各种对象的restapi
接下来就只分析其中一个，其他类似
这里又用到了wsme和wsmeext.pecan。
wsme即[Web Services Made Easy](http://wsme.readthedocs.org/en/latest/)，简化了REST Web服务的编写。
可以运行在另一个框架的顶层，如（Cornice、Flask、Pecan）等，这里就是运行在了pecan之上来更加简单的构建restapi，用法:
`wsmeext.pecan.wsexpose(return_type, *arg_types, **options)`
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
<span class="line">38</span>
<span class="line">39</span>
<span class="line">40</span>
<span class="line">41</span>
<span class="line">42</span>
<span class="line">43</span>
<span class="line">44</span>
<span class="line">45</span>
<span class="line">46</span>
<span class="line">47</span>
<span class="line">48</span>
<span class="line">49</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">import</span> wsmeext.pecan <span class="keyword">as</span> wsme_pecan</span>
<span class="line">...</span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">ResourcesController</span><span class="params">(rest.RestController)</span>:</span></span>
<span class="line"> <span class="string">"""Works on resources."""</span></span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_resource_links</span><span class="params">(self, resource_id, meter_links=<span class="number">1</span>)</span>:</span></span>
<span class="line"> links = [_make_link(<span class="string">'self'</span>, pecan.request.host_url, <span class="string">'resources'</span>,</span>
<span class="line"> resource_id)]</span>
<span class="line"> <span class="keyword">if</span> meter_links:</span>
<span class="line"> <span class="keyword">for</span> meter <span class="keyword">in</span> pecan.request.storage_conn.get_meters(</span>
<span class="line"> resource=resource_id):</span>
<span class="line"> query = &#123;<span class="string">'field'</span>: <span class="string">'resource_id'</span>, <span class="string">'value'</span>: resource_id&#125;</span>
<span class="line"> links.append(_make_link(meter.name, pecan.request.host_url,</span>
<span class="line"> <span class="string">'meters'</span>, meter.name, query=query))</span>
<span class="line"> <span class="keyword">return</span> links</span>
<span class="line"></span>
<span class="line"> <span class="comment">#这里的get_one对应的就是GET http://&lt;your ip&gt;:8777/v2/resources/&lt;resource_id&gt;</span></span>
<span class="line"> <span class="comment">#返回的为一个Resource类型</span></span>
<span class="line"> <span class="comment">#参数为unicode类型</span></span>
<span class="line"><span class="decorator"> @wsme_pecan.wsexpose(Resource, unicode)</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">get_one</span><span class="params">(self, resource_id)</span>:</span></span>
<span class="line"> <span class="string">"""Retrieve details about one resource.</span>
<span class="line"> :param resource_id: The UUID of the resource.</span>
<span class="line"> """</span></span>
<span class="line"> authorized_project = acl.get_limited_to_project(pecan.request.headers)</span>
<span class="line"> resources = list(pecan.request.storage_conn.get_resources(</span>
<span class="line"> resource=resource_id, project=authorized_project))</span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> resources:</span>
<span class="line"> <span class="keyword">raise</span> EntityNotFound(_(<span class="string">'Resource'</span>), resource_id)</span>
<span class="line"> <span class="keyword">return</span> Resource.from_db_and_links(resources[<span class="number">0</span>],</span>
<span class="line"> self._resource_links(resource_id))</span>
<span class="line"> <span class="comment">#这里的get_all对应的就是GET http://&lt;your ip&gt;:8777/v2/resources[?meter_links=&#123;meter_links&#125;&amp;</span></span>
<span class="line"> <span class="comment">#q.field=&#123;field&#125;&amp;q.op=&#123;operator&#125;&amp;q.type=&#123;type&#125;&amp;q.value=&#123;value&#125;]</span></span>
<span class="line"> <span class="comment">#返回类型是包含Resource的序列</span></span>
<span class="line"> <span class="comment">#参数类型是包含Query的序列</span></span>
<span class="line"><span class="decorator"> @wsme_pecan.wsexpose([Resource], [Query], int)</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">get_all</span><span class="params">(self, q=None, meter_links=<span class="number">1</span>)</span>:</span></span>
<span class="line"> <span class="string">"""Retrieve definitions of all of the resources.</span>
<span class="line"> :param q: Filter rules for the resources to be returned.</span>
<span class="line"> :param meter_links: option to include related meter links</span>
<span class="line"> """</span></span>
<span class="line"> q = q <span class="keyword">or</span> []</span>
<span class="line"> kwargs = _query_to_kwargs(q, pecan.request.storage_conn.get_resources)</span>
<span class="line"> resources = [</span>
<span class="line"> Resource.from_db_and_links(r,</span>
<span class="line"> self._resource_links(r.resource_id,</span>
<span class="line"> meter_links))</span>
<span class="line"> <span class="keyword">for</span> r <span class="keyword">in</span> pecan.request.storage_conn.get_resources(**kwargs)]</span>
<span class="line"> <span class="keyword">return</span> resources</span>
</pre></td></tr></table></figure>