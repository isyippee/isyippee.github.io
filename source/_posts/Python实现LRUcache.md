---
title: Python实现LRUcache
tags: [python]
date: 2016-08-18 10:33:30
---

## [](https://ly798.github.io/2016/08/18/Python-%E5%AE%9E%E7%8E%B0-LRUcache/#u4ECB_u7ECD "介绍")介绍

设计一个简单通用的cache，具体思路：
 <!-- more --> 

*   遵循lru算法，lru：近期最少使用算法
*   使用python里的OrderDict来存储kv
*   cache 有大小限制和过期限制
*   外部通过decorator方式来使用 

## [](https://ly798.github.io/2016/08/18/Python-%E5%AE%9E%E7%8E%B0-LRUcache/#u5B9E_u73B0 "实现")实现
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># cache.py</span></span>
<span class="line"><span class="comment"># coding: utf8</span></span>
<span class="line"><span class="comment"># LRUcache decorator</span></span>
<span class="line"><span class="keyword">import</span> time</span>
<span class="line"><span class="keyword">from</span> collections <span class="keyword">import</span> OrderedDict</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Cache</span><span class="params">(object)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self, expiration_time=<span class="number">10</span>, cache_size=<span class="number">100</span>)</span>:</span></span>
<span class="line"> self.cache = OrderedDict()</span>
<span class="line"> self.expiration_time = expiration_time</span>
<span class="line"> self.cache_size = cache_size</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__call__</span><span class="params">(self, func)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">_decorate</span><span class="params">(key)</span>:</span></span>
<span class="line"> <span class="keyword">if</span> self.cache.has_key(key):</span>
<span class="line"> tm_point = self.cache[key].get(<span class="string">'time'</span>)</span>
<span class="line"> <span class="keyword">if</span> time.time() - tm_point &gt; self.expiration_time:</span>
<span class="line"> add_new = <span class="keyword">True</span> <span class="comment"># cache包含该key但是已经过期，需重新添加</span></span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> add_new = <span class="keyword">False</span> <span class="comment"># cache 包含该key且未过期，不需要重新添加</span></span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> add_new = <span class="keyword">True</span> <span class="comment"># cache没有该key，需要重新添加</span></span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> add_new:</span>
<span class="line"> value = func(key)</span>
<span class="line"> <span class="comment"># 判断cache是否满，满了则从头部弹出一个</span></span>
<span class="line"> <span class="keyword">if</span> len(self.cache) == self.cache_size:</span>
<span class="line"> self.cache.popitem(last=<span class="keyword">False</span>)</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> <span class="comment"># 弹出使用现成的</span></span>
<span class="line"> value = self.cache.pop(key).get(<span class="string">'value'</span>)</span>
<span class="line"> cur = time.time()</span>
<span class="line"> <span class="comment"># 添加一个新的或者更新旧的到尾部，并添加time</span></span>
<span class="line"> self.cache[key] = &#123;<span class="string">'time'</span>: cur, <span class="string">'value'</span>: value&#125;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> value</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> _decorate</span>
</pre></td></tr></table></figure> 

测试一下
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
</pre></td><td class="code"><pre><span class="line"><span class="comment"># main.py</span></span>
<span class="line"><span class="keyword">from</span> cache <span class="keyword">import</span> Cache <span class="keyword">as</span> LRUcache</span>
<span class="line"></span>
<span class="line">expiration_time=<span class="number">5</span></span>
<span class="line">cache_size=<span class="number">10</span></span>
<span class="line"></span>
<span class="line"><span class="decorator">@LRUcache(expiration_time, cache_size))</span></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">get_auth</span><span class="params">(key)</span>:</span></span>
<span class="line"> <span class="keyword">print</span> <span class="string">'get a new'</span></span>
<span class="line"> value = key</span>
<span class="line"> <span class="keyword">return</span> value</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">main</span><span class="params">()</span>:</span></span>
<span class="line"> <span class="keyword">for</span> i <span class="keyword">in</span> <span class="string">'a'</span> * <span class="number">5</span>:</span>
<span class="line"> <span class="keyword">print</span> get_auth(i)</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="keyword">if</span> __name__ == <span class="string">'__main__'</span>:</span>
<span class="line"> main()</span>
</pre></td></tr></table></figure>

输出
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line">get <span class="tag">a</span> new</span>
<span class="line"><span class="tag">a</span></span>
<span class="line"><span class="tag">a</span></span>
<span class="line"><span class="tag">a</span></span>
<span class="line"><span class="tag">a</span></span>
<span class="line">a</span>
</pre></td></tr></table></figure>

说明只是第一次去调用了`get_auth`，后面都是从cache取的。
