---
title: Console font
tags: [linux]
date: 2016-03-18 10:27:25
---

## [](https://ly798.github.io/2016/03/18/Console-python-color/#u4ECB_u7ECD "介绍")介绍

控制台的字体样式、颜色是可控制的，具体的描述可以参考:
[ANSI escape code](https://en.wikipedia.org/wiki/ANSI_escape_code)
[ANSI Escape Sequences](http://www.isthe.com/chongo/tech/comp/ansi_escapes.html)
 <!-- more --> 

## [](https://ly798.github.io/2016/03/18/Console-python-color/#code "code")code

`ESC` 在ASCII字表里是 `\033`,其使用的格式一般是 `\033[显示方式;前景色;背景色m`

显示方式
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="number">00</span> 或 <span class="number">0</span> 正常显示 </span>
<span class="line"><span class="number">01</span> 或 <span class="number">1</span> 粗体 </span>
<span class="line"><span class="number">02</span> 或 <span class="number">2</span> 模糊 </span>
<span class="line"><span class="number">03</span> 或 <span class="number">3</span> 斜体 </span>
<span class="line"><span class="number">04</span> 或 <span class="number">4</span> 下划线 </span>
<span class="line"><span class="number">05</span> 或 <span class="number">5</span> 闪烁（慢） </span>
<span class="line"><span class="number">06</span> 或 <span class="number">6</span> 闪烁（快） </span>
<span class="line"><span class="number">07</span> 或 <span class="number">7</span> 反转显示（前景色与背景色调过来） </span>
<span class="line"><span class="number">08</span> 或 <span class="number">8</span> 隐藏 </span>
<span class="line"><span class="number">22</span> 正常 </span>
<span class="line"><span class="number">23</span> 不斜体 </span>
<span class="line"><span class="number">24</span> 无下划线 </span>
<span class="line"><span class="number">25</span> 不闪烁 </span>
<span class="line"><span class="number">27</span> 不反转 </span>
<span class="line"><span class="number">28</span> 不隐藏</span>
</pre></td></tr></table></figure>

前景色
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line"><span class="number">30</span> 黑色 </span>
<span class="line"><span class="number">31</span> 红色 </span>
<span class="line"><span class="number">32</span> 绿色 </span>
<span class="line"><span class="number">33</span> 黄色 </span>
<span class="line"><span class="number">34</span> 蓝色 </span>
<span class="line"><span class="number">35</span> 品红/紫红 </span>
<span class="line"><span class="number">36</span> 青色/蓝绿 </span>
<span class="line"><span class="number">37</span> 白色 </span>
<span class="line"><span class="number">38</span> xterm-<span class="number">256</span> 色 </span>
<span class="line"><span class="number">39</span> 默认色</span>
</pre></td></tr></table></figure>

背景色
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
<span class="line">10</span>
</pre></td><td class="code"><pre><span class="line"><span class="number">40</span> 黑色 </span>
<span class="line"><span class="number">41</span> 红色 </span>
<span class="line"><span class="number">42</span> 绿色 </span>
<span class="line"><span class="number">43</span> 黄色 </span>
<span class="line"><span class="number">44</span> 蓝色 </span>
<span class="line"><span class="number">45</span> 品红/紫红 </span>
<span class="line"><span class="number">46</span> 青色/蓝绿 </span>
<span class="line"><span class="number">47</span> 白色 </span>
<span class="line"><span class="number">48</span> xterm-<span class="number">256</span> 色 </span>
<span class="line"><span class="number">49</span> 默认色</span>
</pre></td></tr></table></figure>

`\033[00;39;49m` 常放到后面用于清除前面的设置

## [](https://ly798.github.io/2016/03/18/Console-python-color/#python__u4F7F_u7528 "python 使用")python 使用

在一些 python 脚本中常常会用到输出，写一个常用的类
printer.py
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Printer</span><span class="params">(object)</span>:</span></span>
<span class="line"> DM = &#123;<span class="string">'default'</span>: <span class="string">'00'</span>,</span>
<span class="line"> <span class="string">'bold'</span>: <span class="string">'01'</span>,</span>
<span class="line"> <span class="string">'italic'</span>: <span class="string">'03'</span>,</span>
<span class="line"> <span class="string">'underline'</span>: <span class="string">'04'</span>&#125;</span>
<span class="line"> FG = &#123;<span class="string">'default'</span>: <span class="string">'39'</span>,</span>
<span class="line"> <span class="string">'black'</span>: <span class="string">'30'</span>,</span>
<span class="line"> <span class="string">'red'</span>: <span class="string">'31'</span>,</span>
<span class="line"> <span class="string">'green'</span>: <span class="string">'32'</span>,</span>
<span class="line"> <span class="string">'yellow'</span>: <span class="string">'33'</span>,</span>
<span class="line"> <span class="string">'blue'</span>: <span class="string">'34'</span>,</span>
<span class="line"> <span class="string">'white'</span>: <span class="string">'37'</span>&#125;</span>
<span class="line"> BG = &#123;<span class="string">'default'</span>: <span class="string">'49'</span>,</span>
<span class="line"> <span class="string">'black'</span>: <span class="string">'40'</span>,</span>
<span class="line"> <span class="string">'red'</span>: <span class="string">'41'</span>,</span>
<span class="line"> <span class="string">'green'</span>: <span class="string">'42'</span>,</span>
<span class="line"> <span class="string">'yellow'</span>: <span class="string">'43'</span>,</span>
<span class="line"> <span class="string">'blue'</span>: <span class="string">'44'</span>,</span>
<span class="line"> <span class="string">'white'</span>: <span class="string">'47'</span>&#125;</span>
<span class="line"> DEFAULT = <span class="string">'\033[00;39;49m'</span></span>
<span class="line"> CUSTOM = <span class="string">'\033[&#123;0&#125;;&#123;1&#125;;&#123;2&#125;m'</span></span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self)</span>:</span></span>
<span class="line"> self.map = &#123;&#125;</span>
<span class="line"> <span class="keyword">for</span> dier <span class="keyword">in</span> dir(self):</span>
<span class="line"> <span class="keyword">if</span> <span class="string">'custom'</span> <span class="keyword">in</span> dier:</span>
<span class="line"> lev = getattr(self, dier)</span>
<span class="line"> self.map[lev[<span class="number">0</span>]] = lev[<span class="number">1</span>]</span>
<span class="line"></span>
<span class="line"> custom_info = (<span class="number">0</span>, CUSTOM.format(DM[<span class="string">'default'</span>], FG[<span class="string">'default'</span>], BG[<span class="string">'default'</span>]))</span>
<span class="line"> custom_success = (<span class="number">1</span>, CUSTOM.format(DM[<span class="string">'underline'</span>], FG[<span class="string">'default'</span>], BG[<span class="string">'green'</span>]))</span>
<span class="line"> custom_warn = (<span class="number">2</span>, CUSTOM.format(DM[<span class="string">'default'</span>], FG[<span class="string">'red'</span>], BG[<span class="string">'default'</span>]))</span>
<span class="line"> custom_error = (<span class="number">3</span>, CUSTOM.format(DM[<span class="string">'bold'</span>], FG[<span class="string">'red'</span>], BG[<span class="string">'default'</span>]))</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">printer</span><span class="params">(mode, chs)</span>:</span></span>
<span class="line"> p = Printer()</span>
<span class="line"> style = p.map[mode]</span>
<span class="line"> <span class="keyword">print</span> <span class="string">'%s%s%s'</span> % (style, chs, p.DEFAULT)</span>
<span class="line"></span>
<span class="line"></span>
<span class="line"><span class="keyword">if</span> __name__ == <span class="string">'__main__'</span>:</span>
<span class="line"> printer(<span class="number">0</span>, <span class="string">"info..."</span>)</span>
<span class="line"> printer(<span class="number">1</span>, <span class="string">"success..."</span>)</span>
<span class="line"> printer(<span class="number">2</span>, <span class="string">"warn..."</span>)</span>
<span class="line"> printer(<span class="number">3</span>, <span class="string">"error..."</span>)</span>
</pre></td></tr></table></figure>

效果如下：
![result](2016-03-29-094522_323x90_scrot.png)
