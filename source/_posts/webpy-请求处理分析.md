---
title: webpy 请求处理分析
tags: [python]
date: 2016-07-01 10:32:59
---

## [](https://ly798.github.io/2016/07/01/webpy-%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E5%88%86%E6%9E%90/#u4ECB_u7ECD "介绍")介绍

通过一个todolist的例子来分析请求处理的大概的流程。
 <!-- more --> 

## [](https://ly798.github.io/2016/07/01/webpy-%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E5%88%86%E6%9E%90/#u8FC7_u7A0B "过程")过程

从`app.run()`服务开始运行
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
<span class="line">50</span>
<span class="line">51</span>
<span class="line">52</span>
<span class="line">53</span>
<span class="line">54</span>
<span class="line">55</span>
<span class="line">56</span>
<span class="line">57</span>
<span class="line">58</span>
<span class="line">59</span>
<span class="line">60</span>
<span class="line">61</span>
<span class="line">62</span>
<span class="line">63</span>
<span class="line">64</span>
<span class="line">65</span>
<span class="line">66</span>
<span class="line">67</span>
<span class="line">68</span>
<span class="line">69</span>
<span class="line">70</span>
<span class="line">71</span>
<span class="line">72</span>
<span class="line">73</span>
<span class="line">74</span>
<span class="line">75</span>
<span class="line">76</span>
<span class="line">77</span>
<span class="line">78</span>
<span class="line">79</span>
<span class="line">80</span>
<span class="line">81</span>
<span class="line">82</span>
<span class="line">83</span>
<span class="line">84</span>
<span class="line">85</span>
<span class="line">86</span>
<span class="line">87</span>
<span class="line">88</span>
<span class="line">89</span>
<span class="line">90</span>
<span class="line">91</span>
<span class="line">92</span>
<span class="line">93</span>
<span class="line">94</span>
<span class="line">95</span>
<span class="line">96</span>
<span class="line">97</span>
<span class="line">98</span>
<span class="line">99</span>
<span class="line">100</span>
<span class="line">101</span>
<span class="line">102</span>
<span class="line">103</span>
<span class="line">104</span>
<span class="line">105</span>
<span class="line">106</span>
<span class="line">107</span>
<span class="line">108</span>
<span class="line">109</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">application</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">run</span><span class="params">(self, *middleware)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> wsgi.runwsgi(self.wsgifunc(*middleware))</span>
<span class="line"></span>
<span class="line"><span class="comment">#这里的传递fun是wsgi中的app</span></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">runwsgi</span><span class="params">(func)</span>:</span></span>
<span class="line"> <span class="comment">#判断fcgi、scgi、simple server，使用simple server</span></span>
<span class="line"> httpserver.runsimple(func, server_addr)</span>
<span class="line"></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">runsimple</span><span class="params">(func, server_address=<span class="params">(<span class="string">"0.0.0.0"</span>, <span class="number">8080</span>)</span>)</span>:</span></span>
<span class="line"> server = WSGIServer(server_address, func)</span>
<span class="line"> server.start()</span>
<span class="line"></span>
<span class="line"><span class="comment">#这个WSGIServer是一个方法，不是类</span></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">WSGIServer</span><span class="params">(server_address, wsgi_app)</span>:</span></span>
<span class="line"> <span class="keyword">import</span> wsgiserver</span>
<span class="line"> server = wsgiserver.CherryPyWSGIServer(server_address, wsgi_app, server_name=<span class="string">"localhost"</span>)</span>
<span class="line"> <span class="keyword">return</span> server</span>
<span class="line"> </span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">CherryPyWSGIServer</span><span class="params">(HTTPServer)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, bind_addr, wsgi_app, numthreads=<span class="number">10</span>, server_name=None,</span>
<span class="line"> max=-<span class="number">1</span>, request_queue_size=<span class="number">5</span>, timeout=<span class="number">10</span>, shutdown_timeout=<span class="number">5</span>)</span>:</span></span>
<span class="line"> <span class="comment">#初始化线程池</span></span>
<span class="line"> self.requests = ThreadPool(self, min=numthreads <span class="keyword">or</span> <span class="number">1</span>, max=max)</span>
<span class="line"> self.wsgi_app = wsgi_app</span>
<span class="line"> <span class="comment">#用于调用wsgi处理请求并回写给客户端</span></span>
<span class="line"> self.gateway = wsgi_gateways[self.wsgi_version]</span>
<span class="line"> </span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">HTTPServer</span><span class="params">(object)</span>:</span></span>
<span class="line"> <span class="comment">#start实在runsimple方法中调用了</span></span>
<span class="line"> <span class="comment">#获取socket信息，绑定监听地址和端口</span></span>
<span class="line"> <span class="comment">#启动self.requests线程</span></span>
<span class="line"> <span class="comment">#获取来自可断断的请求</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">start</span><span class="params">(self)</span>:</span></span>
<span class="line"> self.requests.start()</span>
<span class="line"> self.ready = <span class="keyword">True</span></span>
<span class="line"> <span class="keyword">while</span> self.ready:</span>
<span class="line"> self.tick()</span>
<span class="line"> </span>
<span class="line"> <span class="comment">#接受客户端的请求，封装为ConnectionClass，放到线程池的队列中</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">tick</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="string">"""Accept a new connection and put it on the Queue."""</span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> s, addr = self.socket.accept()</span>
<span class="line"> ...</span>
<span class="line"> conn = self.ConnectionClass(self, s, makefile)</span>
<span class="line"> self.requests.put(conn)</span>
<span class="line"> </span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">ThreadPool</span><span class="params">(object)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, server, min=<span class="number">10</span>, max=-<span class="number">1</span>)</span>:</span></span>
<span class="line"> self.server = server</span>
<span class="line"> self.min = min</span>
<span class="line"> self.max = max</span>
<span class="line"> self._threads = []</span>
<span class="line"> self._queue = Queue.Queue()</span>
<span class="line"> self.get = self._queue.get</span>
<span class="line"> </span>
<span class="line"> <span class="comment">#创建固定数量的线程，并启动</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">start</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="string">"""Start the pool of threads."""</span></span>
<span class="line"> <span class="keyword">for</span> i <span class="keyword">in</span> range(self.min):</span>
<span class="line"> self._threads.append(WorkerThread(self.server))</span>
<span class="line"> <span class="keyword">for</span> worker <span class="keyword">in</span> self._threads:</span>
<span class="line"> worker.setName(<span class="string">"CP Server "</span> + worker.getName())</span>
<span class="line"> worker.start()</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">put</span><span class="params">(self, obj)</span>:</span></span>
<span class="line"> self._queue.put(obj)</span>
<span class="line"> <span class="keyword">if</span> obj <span class="keyword">is</span> _SHUTDOWNREQUEST:</span>
<span class="line"> <span class="keyword">return</span></span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">WorkerThread</span><span class="params">(threading.Thread)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">run</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">while</span> <span class="keyword">True</span>:</span>
<span class="line"> conn = self.server.requests.get() <span class="comment">#self.get = self._queue.get</span></span>
<span class="line"> conn.communicate()</span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">HTTPConnection</span><span class="params">(object)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">communicate</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">while</span> <span class="keyword">True</span>:</span>
<span class="line"> <span class="comment">#验证</span></span>
<span class="line"> req = self.RequestHandlerClass(self.server, self)</span>
<span class="line"> req.parse_request()</span>
<span class="line"> req.respond()</span>
<span class="line"> </span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">HTTPRequest</span><span class="params">(object)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">respond</span><span class="params">(self)</span>:</span></span>
<span class="line"> self.server.gateway(self).respond()</span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">WSGIGateway_10</span><span class="params">(WSGIGateway)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, req)</span>:</span></span>
<span class="line"> self.req = req</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">respond</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="comment"># 这里的wsgi_app就是CherryPyWSGIServer的__init__中传递的wsgi_app</span></span>
<span class="line"> response = self.req.server.wsgi_app(self.env, self.start_response)</span>
<span class="line"> <span class="keyword">for</span> chunk <span class="keyword">in</span> response:</span>
<span class="line"> <span class="keyword">if</span> chunk:</span>
<span class="line"> <span class="keyword">if</span> isinstance(chunk, unicode):</span>
<span class="line"> chunk = chunk.encode(<span class="string">'ISO-8859-1'</span>)</span>
<span class="line"> self.write(chunk)</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">application</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">wsgifunc</span><span class="params">(self, *middleware)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">wsgi</span><span class="params">(env, start_resp)</span>:</span></span>
<span class="line"> self.load(env)</span>
<span class="line"> result = web.safestr(iter(result))</span>
<span class="line"> start_resp(status, headers)</span>
<span class="line"> <span class="keyword">return</span> itertools.chain(result, cleanup())</span>
<span class="line"> <span class="keyword">return</span> wsgi</span>
</pre></td></tr></table></figure>

一一对应WSGI中的角色，豁然开朗。

大概总结wsgi规则:
application要是一个可调用的对象，接受两个参数，返回一个可迭代的值
server调用application，传递给application两个参数，迭代application的返回值，传递给client

下面具体看application的实现
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
<span class="line">50</span>
<span class="line">51</span>
<span class="line">52</span>
<span class="line">53</span>
<span class="line">54</span>
<span class="line">55</span>
<span class="line">56</span>
<span class="line">57</span>
<span class="line">58</span>
<span class="line">59</span>
<span class="line">60</span>
<span class="line">61</span>
<span class="line">62</span>
<span class="line">63</span>
<span class="line">64</span>
<span class="line">65</span>
<span class="line">66</span>
<span class="line">67</span>
<span class="line">68</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">application</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, mapping=<span class="params">()</span>, fvars=&#123;&#125;, autoreload=None)</span>:</span></span>
<span class="line"> <span class="keyword">if</span> autoreload <span class="keyword">is</span> <span class="keyword">None</span>:</span>
<span class="line"> autoreload = web.config.get(<span class="string">'debug'</span>, <span class="keyword">False</span>)</span>
<span class="line"> self.init_mapping(mapping)</span>
<span class="line"> self.fvars = fvars</span>
<span class="line"> self.processors = []</span>
<span class="line"> self.add_processor(loadhook(self._load))</span>
<span class="line"> self.add_processor(unloadhook(self._unload))</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">wsgifunc</span><span class="params">(self, *middleware)</span>:</span></span>
<span class="line"> <span class="string">"""Returns a WSGI-compatible function for this application."""</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">wsgi</span><span class="params">(env, start_resp)</span>:</span></span>
<span class="line"> <span class="comment"># clear threadlocal to avoid inteference of previous requests</span></span>
<span class="line"> self._cleanup()</span>
<span class="line"> self.load(env)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="comment"># allow uppercase methods only</span></span>
<span class="line"> <span class="keyword">if</span> web.ctx.method.upper() != web.ctx.method:</span>
<span class="line"> <span class="keyword">raise</span> web.nomethod()</span>
<span class="line"> result = self.handle_with_processors()</span>
<span class="line"> <span class="keyword">if</span> is_generator(result):</span>
<span class="line"> result = peep(result)</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> result = [result]</span>
<span class="line"> <span class="keyword">except</span> web.HTTPError, e:</span>
<span class="line"> result = [e.data]</span>
<span class="line"> result = web.safestr(iter(result))</span>
<span class="line"> status, headers = web.ctx.status, web.ctx.headers</span>
<span class="line"> start_resp(status, headers)</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">cleanup</span><span class="params">()</span>:</span></span>
<span class="line"> self._cleanup()</span>
<span class="line"> <span class="keyword">yield</span> <span class="string">''</span> <span class="comment"># force this function to be a generator</span></span>
<span class="line"> <span class="keyword">return</span> itertools.chain(result, cleanup())</span>
<span class="line"> <span class="keyword">for</span> m <span class="keyword">in</span> middleware: </span>
<span class="line"> wsgi = m(wsgi)</span>
<span class="line"> <span class="keyword">return</span> wsgi</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">init_mapping</span><span class="params">(self, mapping)</span>:</span></span>
<span class="line"> self.mapping = list(utils.group(mapping, <span class="number">2</span>))</span>
<span class="line"> <span class="comment">#将元组转换为类似[['/', 'Index'], ['/del/(\\d+)', 'Delete']]</span></span>
<span class="line"> </span>
<span class="line"> <span class="comment"># 执行self.handle()</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">handle_with_processors</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="comment"># return self.processors[0](self.processors[1](self.handle()))</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">process</span><span class="params">(processors)</span>:</span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="keyword">if</span> processors:</span>
<span class="line"> p, processors = processors[<span class="number">0</span>], processors[<span class="number">1</span>:]</span>
<span class="line"> <span class="keyword">return</span> p(<span class="keyword">lambda</span>: process(processors))</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> <span class="keyword">return</span> self.handle()</span>
<span class="line"> <span class="keyword">except</span> web.HTTPError:</span>
<span class="line"> <span class="keyword">raise</span></span>
<span class="line"> <span class="keyword">except</span> (KeyboardInterrupt, SystemExit):</span>
<span class="line"> <span class="keyword">raise</span></span>
<span class="line"> <span class="keyword">except</span>:</span>
<span class="line"> <span class="keyword">print</span> &gt;&gt; web.debug, traceback.format_exc()</span>
<span class="line"> <span class="keyword">raise</span> self.internalerror()</span>
<span class="line"> </span>
<span class="line"> <span class="comment"># processors must be applied in the resvere order. (??)</span></span>
<span class="line"> <span class="keyword">return</span> process(self.processors)</span>
<span class="line"> </span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">handle</span><span class="params">(self)</span>:</span></span>
<span class="line"> fn, args = self._match(self.mapping, web.ctx.path)</span>
<span class="line"> <span class="keyword">return</span> self._delegate(fn, self.fvars, args)</span>
<span class="line"> <span class="comment"># 加入请求路径是 /</span></span>
<span class="line"> <span class="comment"># _match 获得 Index 和 args</span></span>
<span class="line"> <span class="comment"># _delegate 执行 globals()中的Index的实例中meth对应方法的值返回</span></span>
</pre></td></tr></table></figure>

## [](https://ly798.github.io/2016/07/01/webpy-%E8%AF%B7%E6%B1%82%E5%A4%84%E7%90%86%E5%88%86%E6%9E%90/#u53C2_u8003 "参考")参考

[https://www.python.org/dev/peps/pep-3333/](https://www.python.org/dev/peps/pep-3333/)
[http://webpy.org/src/todo-list/0.3](http://webpy.org/src/todo-list/0.3)
