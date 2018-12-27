---
title: webpy template
tags: [python]
date: 2016-08-17 10:33:14
---

## [](https://ly798.github.io/2016/08/17/webpy-template/#u4ECB_u7ECD "介绍")介绍

模板是为了将逻辑数据与界面模板文件进行分离。
 <!-- more --> 

## [](https://ly798.github.io/2016/08/17/webpy-template/#u8FC7_u7A0B "过程")过程

还是todo的例子。
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
</pre></td><td class="code"><pre><span class="line">render = web.template.render(<span class="string">'templates'</span>, base=<span class="string">'base'</span>)</span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Index</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> </span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">GET</span><span class="params">(self)</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="keyword">return</span> render.index(todos, form)</span>
</pre></td></tr></table></figure> 

`render = web.template.render(&#39;templates&#39;, base=&#39;base&#39;)`创建了一个Render实例

`render.index(todos, form)`对应
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Render</span>:</span></span>
<span class="line"> ...</span>
<span class="line"> </span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__getattr__</span><span class="params">(self, name)</span>:</span></span>
<span class="line"> t = self._template(name)</span>
<span class="line"> <span class="keyword">if</span> self._base <span class="keyword">and</span> isinstance(t, Template):</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">template</span><span class="params">(*a, **kw)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> self._base(t(*a, **kw))</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> template</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> <span class="keyword">return</span> self._template(name)</span>
</pre></td></tr></table></figure>

*   如果没有base，返回一个名为index的Template()，所以是`Template()__call__(todos, form)`；
*   有base的情况下，是`Template()__call__(Template()__call__(todos, form))`，外面的是base的，里面的是index。上述表达是错误的，这样好理解。 

下面具体介绍：

1.  在没有base的情况下，`Template`的初始化的地方是`Template(open(path).read(), filename=path, **self._keywords)`

        *   generate_code
找到 def with，read_suite，将def with替换成
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">__template__</span> <span class="params">(todos, form)</span>:</span></span>
<span class="line"> __lineoffset__ = -<span class="number">4</span></span>
<span class="line"> loop = ForLoop()</span>
<span class="line"> self = TemplateResult(); extend_ = self.extend</span>
<span class="line"> ...</span>
<span class="line"> <span class="keyword">return</span> self</span>
</pre></td></tr></table></figure>
    在头部加上`# coding: utf-8\n`，解析中间的python code
    *   compile
编译成Code对象，`Template()__call__(todos, form)`的时候将todo、form作为该方法`__template__`的参数，返回一个`TemplateResult`2.  有base的情况下，将index返回的self（即一个TemplateResult的实例）作为新的`__template__`的参数，返回一个`TemplateResult` 

最后，`TemplateResult`作为一个可迭代的对象，传递给client

## [](https://ly798.github.io/2016/08/17/webpy-template/#u53C2_u8003 "参考")参考

[http://webpy.org/src/todo-list/0.3](http://webpy.org/src/todo-list/0.3)
