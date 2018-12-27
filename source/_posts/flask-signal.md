---
title: flask signal
tags: [python]
date: 2016-09-20  10:22:27
---

## [](https://ly798.github.io/2016/09/20/flask-signal/#u4ECB_u7ECD "介绍")介绍

flask通过blinker实现信号机制。
 <!-- more --> 

## [](https://ly798.github.io/2016/09/20/flask-signal/#blinker "blinker")blinker

blinker是python里 的信号库，可用来对组建解耦。一个例子
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
</pre></td><td class="code"><pre><span class="line"><span class="comment">#coding: utf8</span></span>
<span class="line"><span class="keyword">from</span> blinker <span class="keyword">import</span> signal</span>
<span class="line"></span>
<span class="line"><span class="comment">#定义signal，单例模式，所以全局唯一</span></span>
<span class="line">print_signal = signal(<span class="string">'print'</span>)</span>
<span class="line"></span>
<span class="line"><span class="comment">#订阅signal，connect()或装饰器connect_via()</span></span>
<span class="line"><span class="decorator">@print_signal.connect_via()</span></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">printer</span><span class="params">(sender)</span>:</span></span>
<span class="line"> <span class="keyword">print</span> sender</span>
<span class="line"> </span>
<span class="line"><span class="comment"># print_signal.connect(printer)</span></span>
<span class="line"></span>
<span class="line"><span class="comment">#触发 signal</span></span>
<span class="line">print_signal.send(<span class="string">'obj'</span>)</span>
</pre></td></tr></table></figure>

## [](https://ly798.github.io/2016/09/20/flask-signal/#flask_signal "flask signal")flask signal

### [](https://ly798.github.io/2016/09/20/flask-signal/#flask_core_u4E2D_u7684signal "flask core中的signal")flask core中的signal

`flask/signals.py`
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
</pre></td><td class="code"><pre><span class="line"><span class="keyword">from</span> blinker <span class="keyword">import</span> Namespace</span>
<span class="line"></span>
<span class="line">_signals = Namespace()</span>
<span class="line"></span>
<span class="line">template_rendered = _signals.signal(<span class="string">'template-rendered'</span>)</span>
<span class="line">request_started = _signals.signal(<span class="string">'request-started'</span>)</span>
<span class="line">request_finished = _signals.signal(<span class="string">'request-finished'</span>) <span class="comment"># 发送响应前</span></span>
<span class="line">request_tearing_down = _signals.signal(<span class="string">'request-tearing-down'</span>)</span>
<span class="line">got_request_exception = _signals.signal(<span class="string">'got-request-exception'</span>)</span>
<span class="line">appcontext_tearing_down = _signals.signal(<span class="string">'appcontext-tearing-down'</span>)</span>
<span class="line">appcontext_pushed = _signals.signal(<span class="string">'appcontext-pushed'</span>)</span>
<span class="line">appcontext_popped = _signals.signal(<span class="string">'appcontext-popped'</span>)</span>
<span class="line">message_flashed = _signals.signal(<span class="string">'message-flashed'</span>)</span>
</pre></td></tr></table></figure>

### [](https://ly798.github.io/2016/09/20/flask-signal/#signal_send "signal send")signal send

以request_started为例。

先看flask wsgi app 的实现

`flask/app.py`
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Flask</span><span class="params">(_PackageBoundObject)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">wsgi_app</span><span class="params">(self, environ, start_response)</span>:</span></span>
<span class="line"> ctx = self.request_context(environ)</span>
<span class="line"> ctx.push()</span>
<span class="line"> error = <span class="keyword">None</span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> response = self.full_dispatch_request()</span>
<span class="line"> <span class="keyword">except</span> Exception <span class="keyword">as</span> e:</span>
<span class="line"> error = e</span>
<span class="line"> response = self.make_response(self.handle_exception(e))</span>
<span class="line"> <span class="keyword">return</span> response(environ, start_response)</span>
<span class="line"> <span class="keyword">finally</span>:</span>
<span class="line"> <span class="keyword">if</span> self.should_ignore_error(error):</span>
<span class="line"> error = <span class="keyword">None</span></span>
<span class="line"> ctx.auto_pop(error)</span>
</pre></td></tr></table></figure>

当调用`self.full_dispatch_request()`时
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">full_dispatch_request</span><span class="params">(self)</span>:</span></span>
<span class="line"> self.try_trigger_before_first_request_functions()</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> request_started.send(self)</span>
<span class="line"> rv = self.preprocess_request()</span>
<span class="line"> <span class="keyword">if</span> rv <span class="keyword">is</span> <span class="keyword">None</span>:</span>
<span class="line"> rv = self.dispatch_request()</span>
<span class="line"> <span class="keyword">except</span> Exception <span class="keyword">as</span> e:</span>
<span class="line"> rv = self.handle_user_exception(e)</span>
<span class="line"> response = self.make_response(rv)</span>
<span class="line"> response = self.process_response(response)</span>
<span class="line"> request_finished.send(self, response=response)</span>
<span class="line"> <span class="keyword">return</span> response</span>
</pre></td></tr></table></figure> 

该方法会先发送一个signal，然后才进行request的预处理。

在flask中用装饰器的方式使用
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="decorator">@request_started.connect_via(app)</span></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">start</span><span class="params">(sender, **extra)</span>:</span></span>
<span class="line"> <span class="keyword">print</span> sender.name</span>
</pre></td></tr></table></figure>
