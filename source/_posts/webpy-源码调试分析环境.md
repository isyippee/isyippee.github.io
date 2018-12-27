---
title: webpy 源码调试分析环境
tags: [python]
date: 2016-06-20 10:32:28
---

## [](https://ly798.github.io/2016/06/20/webpy-%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%95%E5%88%86%E6%9E%90%E7%8E%AF%E5%A2%83/#u4ECB_u7ECD "介绍")介绍

用过不少的python web框架，然而仅仅是使用并不知其中原理，所以萌生阅读框架源码的想法。
对于框架的选择，django使用最多最全面，然而并不适合；
twisted&amp;tornado异步网络框架，包括web，不合适；
还是webpy吧 - -，代码少，目的能够达到。
 <!-- more --> 

### [](https://ly798.github.io/2016/06/20/webpy-%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%95%E5%88%86%E6%9E%90%E7%8E%AF%E5%A2%83/#config "config")config

fork [webpy/webpy](https://github.com/webpy/webpy) 到自己的github，并克隆到本地，建立新分支,
在项目根目录创建一个demo的目录用于记录笔记或是写例子。

使用调试工具，虽然pdb很好用了，但是有时候print还是可以的，比如多线程的时候 - -，
也就多了修改源码的需求，去修改系统的web模块肯定不妥，那就使用当前分支的web模块吧，
然而要得到这样的效果，就在写demo的时候import之前：
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">import</span> sys</span>
<span class="line">sys.path.insert(<span class="number">0</span>, <span class="string">'/home/xxx/webpy/'</span>)</span>
</pre></td></tr></table></figure>

蛋疼，但是，使用pycharm会优先搜索当前工程目录，完全符合我的目录结构和需求！

### [](https://ly798.github.io/2016/06/20/webpy-%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%95%E5%88%86%E6%9E%90%E7%8E%AF%E5%A2%83/#example "example")example

我从一个简单的例子入手看代码，例子来自[http://webpy.org/src/todo-list/0.3](http://webpy.org/src/todo-list/0.3)。
将其放到我的demo目录中，大概是这样子：
![webpy](webpy_strut.png)
下面无论是用print，还是pdb，还是pycharm自带的debug都直接操作当前工程的web模块了。

## [](https://ly798.github.io/2016/06/20/webpy-%E6%BA%90%E7%A0%81%E8%B0%83%E8%AF%95%E5%88%86%E6%9E%90%E7%8E%AF%E5%A2%83/#u53C2_u8003 "参考")参考

`http://webpy.org/src/todo-list/0.3`
