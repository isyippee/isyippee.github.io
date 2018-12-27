---
title: crush
tags: [ceph]
date: 2016-09-07 10:21:40
---

## [](https://ly798.github.io/2016/09/07/crush/#u4ECB_u7ECD "介绍")介绍

在看ceph的时候，crush这部分算是刚接触ceph时最让人迷惑的地方吧。

在分布式存储中的数据分布问题：
 <!-- more -->

*   为了负载均衡，数据需要均匀的进行分布
*   数据分布是动态的，要能够便于存储介质的添加删除

ceph使用了CRUSH，swift使用了consistent hashing，CRUSH类似consistent hashing，与之最大的区别在于接受的参数多了cluster map和placement rules，更加详细的考虑到了机架等存储层次。

## [](https://ly798.github.io/2016/09/07/crush/#consistent_hashing "consistent hashing")consistent hashing

### [](https://ly798.github.io/2016/09/07/crush/#u666E_u901Ahash "普通hash")普通hash

N个节点、假设一个均匀的hash函数hash()，计算数据对象key的分布：

`n = hash(key) mod N`

但是当node增加或减少的时候，所有的数据对象都需要重新计算分布，大量的数据移动。

### [](https://ly798.github.io/2016/09/07/crush/#consistent_hashing-1 "consistent hashing")consistent hashing

#### [](https://ly798.github.io/2016/09/07/crush/#u666E_u901Aconsistent_hashing "普通consistent hashing")普通consistent hashing

*   构造值为 0 - 2^32 的ring
*   计算node的hash，定位到ring
*   计算数据对象的hash，定位到ring
*   针对一个数据对象，顺时针查找第一个node即为所属node 

![ring](ring.png)

在node较少的时候，增加或减少节点，有较大的数据移动，负载不均匀。

![add_node](add_node.png)
![reduce_node](reduce_node.png)

#### [](https://ly798.github.io/2016/09/07/crush/#u5206_u5E03_u5F0Fconsistent_hashing "分布式consistent hashing")分布式consistent hashing

引入虚拟节点，其数量在部署的时候预估且不能改变。

![vnode](vnode.png)

当node增加或减少的时候，数据对象 –&gt; vnode 的映射不会改变，只有 vnode –&gt; node 映射改变，少量数据移动。

![vnode_add_node](vnode_add_node.png)

## [](https://ly798.github.io/2016/09/07/crush/#CRUSH "CRUSH")CRUSH

Controlled Replication Under Scalable Hashing

![obj_osd](obj_osd.png)

计算pgid：
<figure class="highlight ocaml"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">hash(<span class="keyword">object</span> <span class="type">ID</span>) <span class="keyword">mod</span> <span class="type">PGs</span> --&gt; <span class="number">58</span></span>
<span class="line">pool <span class="type">ID</span> --&gt; <span class="number">4</span></span>
<span class="line">pgid --&gt; <span class="number">4.58</span></span>
</pre></td></tr></table></figure>

pg映射osd：
<figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="title">CRUSH</span><span class="params">(x)</span></span> ­‐&gt; (osd n1, osd n2,osd n2)</span>
</pre></td></tr></table></figure>

参数

*   x pgid
*   cluster map
*   placement rule 

输出一组osd

### [](https://ly798.github.io/2016/09/07/crush/#cluster_map "cluster map")cluster map

描述存储设备的逻辑分布。在cluster map中有2个概念，

*   bucket 父节点，如机架、host，可以用于隔离故障域；
*   device 叶子节点，硬盘。 

各自的权重与容量、吞吐量有关系。 

![map](map.jpg)

不同的bucket type，有不同的选择算法，计算复杂度和osd删除添加的数据移动是不一样的。 

![bucket_types](bucket_types.png)

*   uniform
所有的item权重都是相同的，不考虑权重
*   list
添加设备的时候，有最优的数据移动，适用集群拓展类型
*   tree
*   straw
考虑权重，有最优的数据移动 

### [](https://ly798.github.io/2016/09/07/crush/#placement_rule "placement rule")placement rule

the replica placement policy，放置策略，描述了选择设备的过程

eg
<figure class="highlight sqf"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line">rule replicated_ruleset &#123;</span>
<span class="line"> ruleset <span class="number">0</span> <span class="preprocessor"># 编号</span></span>
<span class="line"> <span class="built_in">type</span> replicated <span class="preprocessor"># replicated/esurecode</span></span>
<span class="line"> min_size <span class="number">1</span> <span class="preprocessor"># 最小副本数</span></span>
<span class="line"> max_size <span class="number">10</span> <span class="preprocessor"># 最大副本数</span></span>
<span class="line"> <span class="built_in">step</span> take <span class="keyword">default</span></span>
<span class="line"> <span class="built_in">step</span> chooseleaf firstn <span class="number">0</span> <span class="built_in">type</span> host</span>
<span class="line"> <span class="built_in">step</span> emit</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

### [](https://ly798.github.io/2016/09/07/crush/#map_example "map example")map example

获取集群的crush map信息
 <figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">ceph osd getcrushmap -o crush.<span class="built_in">map</span> <span class="preprocessor"># 获取map，已编译</span></span>
<span class="line">crushtool -d crush.<span class="built_in">map</span> &gt;&gt; crush.txt <span class="preprocessor"># 反编译</span></span>
</pre></td></tr></table></figure> 

cursh.txt 的内容
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
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># begin crush map</span></span>
<span class="line">tunable choose_local_tries <span class="number">0</span></span>
<span class="line">tunable choose_local_fallback_tries <span class="number">0</span></span>
<span class="line">tunable choose_total_tries <span class="number">50</span></span>
<span class="line">tunable chooseleaf_descend_once <span class="number">1</span></span>
<span class="line">tunable straw_calc_version <span class="number">1</span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># devices</span></span>
<span class="line">device <span class="number">0</span> osd<span class="number">.0</span></span>
<span class="line">device <span class="number">1</span> osd<span class="number">.1</span></span>
<span class="line">device <span class="number">2</span> osd<span class="number">.2</span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># types</span></span>
<span class="line">type <span class="number">0</span> osd</span>
<span class="line">type <span class="number">1</span> host</span>
<span class="line">type <span class="number">2</span> chassis</span>
<span class="line">type <span class="number">3</span> rack</span>
<span class="line">type <span class="number">4</span> row</span>
<span class="line">type <span class="number">5</span> pdu</span>
<span class="line">type <span class="number">6</span> pod</span>
<span class="line">type <span class="number">7</span> room</span>
<span class="line">type <span class="number">8</span> datacenter</span>
<span class="line">type <span class="number">9</span> region</span>
<span class="line">type <span class="number">10</span> root</span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># buckets</span></span>
<span class="line">host ceph1 &#123;</span>
<span class="line"> id -<span class="number">2</span> <span class="preprocessor"># do not change unnecessarily</span></span>
<span class="line"> <span class="preprocessor"># weight <span class="number">0.093</span></span></span>
<span class="line"> alg straw</span>
<span class="line"> hash <span class="number">0</span> <span class="preprocessor"># rjenkins1</span></span>
<span class="line"> item osd<span class="number">.0</span> weight <span class="number">0.093</span></span>
<span class="line">&#125;</span>
<span class="line">host ceph2 &#123;</span>
<span class="line"> id -<span class="number">3</span> <span class="preprocessor"># do not change unnecessarily</span></span>
<span class="line"> <span class="preprocessor"># weight <span class="number">0.093</span></span></span>
<span class="line"> alg straw</span>
<span class="line"> hash <span class="number">0</span> <span class="preprocessor"># rjenkins1</span></span>
<span class="line"> item osd<span class="number">.1</span> weight <span class="number">0.093</span></span>
<span class="line">&#125;</span>
<span class="line">host ceph3 &#123;</span>
<span class="line"> id -<span class="number">4</span> <span class="preprocessor"># do not change unnecessarily</span></span>
<span class="line"> <span class="preprocessor"># weight <span class="number">0.093</span></span></span>
<span class="line"> alg straw</span>
<span class="line"> hash <span class="number">0</span> <span class="preprocessor"># rjenkins1</span></span>
<span class="line"> item osd<span class="number">.2</span> weight <span class="number">0.093</span></span>
<span class="line">&#125;</span>
<span class="line">root <span class="keyword">default</span> &#123;</span>
<span class="line"> id -<span class="number">1</span> <span class="preprocessor"># do not change unnecessarily</span></span>
<span class="line"> <span class="preprocessor"># weight <span class="number">0.279</span></span></span>
<span class="line"> alg straw</span>
<span class="line"> hash <span class="number">0</span> <span class="preprocessor"># rjenkins1</span></span>
<span class="line"> item ceph1 weight <span class="number">0.093</span></span>
<span class="line"> item ceph2 weight <span class="number">0.093</span></span>
<span class="line"> item ceph3 weight <span class="number">0.093</span></span>
<span class="line">&#125;</span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># rules</span></span>
<span class="line">rule replicated_ruleset &#123;</span>
<span class="line"> ruleset <span class="number">0</span></span>
<span class="line"> type replicated</span>
<span class="line"> min_size <span class="number">1</span></span>
<span class="line"> max_size <span class="number">10</span></span>
<span class="line"> step take <span class="keyword">default</span></span>
<span class="line"> step chooseleaf firstn <span class="number">0</span> type host</span>
<span class="line"> step emit</span>
<span class="line">&#125;</span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># end crush map</span></span>
</pre></td></tr></table></figure> 

#### [](https://ly798.github.io/2016/09/07/crush/#pool_u5B9E_u73B0ssd_u4F5C_u4E3Aprimary "pool实现ssd作为primary")pool实现ssd作为primary

新添加的一个rule规则：
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
</pre></td><td class="code"><pre><span class="line">host ceph1-ssd &#123;</span>
<span class="line"> id -<span class="number">5</span> <span class="preprocessor"># do not change unnecessarily</span></span>
<span class="line"> <span class="preprocessor"># weight <span class="number">0.093</span></span></span>
<span class="line"> alg straw</span>
<span class="line"> hash <span class="number">0</span> <span class="preprocessor"># rjenkins1</span></span>
<span class="line"> item osd<span class="number">.3</span> weight <span class="number">0.093</span></span>
<span class="line">&#125;</span>
<span class="line">root ssd &#123;</span>
<span class="line"> id -<span class="number">6</span> <span class="preprocessor"># do not change unnecessarily</span></span>
<span class="line"> <span class="preprocessor"># weight <span class="number">0.093</span></span></span>
<span class="line"> alg straw</span>
<span class="line"> hash <span class="number">0</span> <span class="preprocessor"># rjenkins1</span></span>
<span class="line"> item ceph1-ssd weight <span class="number">0.093</span></span>
<span class="line">&#125;</span>
<span class="line"></span>
<span class="line">rule ssd_primary &#123;</span>
<span class="line"> ruleset <span class="number">1</span></span>
<span class="line"> type replicated</span>
<span class="line"> min_size <span class="number">1</span></span>
<span class="line"> max_size <span class="number">10</span></span>
<span class="line"> step take ssd</span>
<span class="line"> step chooseleaf firstn <span class="number">1</span> type host</span>
<span class="line"> step emit</span>
<span class="line"> step take <span class="keyword">default</span></span>
<span class="line"> step chooseleaf firstn -<span class="number">1</span> type host</span>
<span class="line"> step emit</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure> 

创建一个rule 是 ssd_primary的pool，并上传一个对象
 <figure class="highlight livescript"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># ceph osd crush rule ls</span></span>
<span class="line"></span>
<span class="line"><span class="comment"># ceph osd pool create test-ssd 8 8 replicated ssd_primary</span></span>
<span class="line"></span>
<span class="line"><span class="comment"># rados -p test-ssd put aaa aaa</span></span>
<span class="line"><span class="comment"># ceph osd map test-ssd aaa</span></span>
<span class="line">osdmap e836 pool <span class="string">'test-ssd'</span> <span class="function"><span class="params">(<span class="number">132</span>)</span> <span class="title">object</span> '<span class="title">aaa</span>' -&gt;</span> pg <span class="number">132.c</span>6f58be5 <span class="function"><span class="params">(<span class="number">132.5</span>)</span> -&gt;</span> up ([<span class="number">3</span>,<span class="number">2</span>,<span class="number">1</span>], p3) acting ([<span class="number">3</span>,<span class="number">2</span>,<span class="number">1</span>], p3)</span>
</pre></td></tr></table></figure> 

可以看出该pg的primary osd 是3，即新定义的ssd osd

### [](https://ly798.github.io/2016/09/07/crush/#u6E90_u7801 "源码")源码

调试选择的过程
 <figure class="highlight stylus"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">ceph osd getmap -o osdmap</span>
<span class="line">ceph osd lspools</span>
<span class="line">gdb --args /home/ceph/src/osdmaptool --test-map-<span class="tag">object</span> aaa --pool <span class="number">132</span> osdmap</span>
<span class="line"><span class="tag">b</span> crush_do_rule</span>
</pre></td></tr></table></figure> <figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">int</span> <span class="title">crush_do_rule</span><span class="params">(<span class="keyword">const</span> <span class="keyword">struct</span> crush_map *<span class="built_in">map</span>,</span>
<span class="line"> <span class="keyword">int</span> ruleno, <span class="keyword">int</span> x, <span class="keyword">int</span> *result, <span class="keyword">int</span> result_max,</span>
<span class="line"> <span class="keyword">const</span> __u32 *weight, <span class="keyword">int</span> weight_max,</span>
<span class="line"> <span class="keyword">int</span> *scratch)</span></span>
<span class="line"></span>&#123;</span>
<span class="line">...</span>
<span class="line"> <span class="keyword">for</span> (step = <span class="number">0</span>; step &lt; rule-&gt;len; step++) &#123;</span>
<span class="line"> <span class="keyword">case</span> CRUSH_RULE_TAKE:</span>
<span class="line"> ...</span>
</pre></td></tr></table></figure> <table> <thead> <tr> <th style="text-align:left">_category_</th> <th style="text-align:left">_name_</th> </tr> </thead> <tbody> <tr> <td style="text-align:left">Start</td> <td style="text-align:left">CRUSH_RULE_TAKE(1)</td> </tr> <tr> <td style="text-align:left">End</td> <td style="text-align:left">CRUSH_RULE_EMIT(4)</td> </tr> <tr> <td style="text-align:left">ChooseBucket</td> <td style="text-align:left">CRUSH_RULE_CHOOSE_FIRSTN(2)</td> </tr> <tr> <td style="text-align:left">ChooseBucket</td> <td style="text-align:left">CRUSH_RULE_CHOOSE_INDEP(3)</td> </tr> <tr> <td style="text-align:left">ChooseDevice</td> <td style="text-align:left">CRUSH_RULE_CHOOSELEAF_FIRSTN(6)</td> </tr> <tr> <td style="text-align:left">ChooseDevice</td> <td style="text-align:left">CRUSH_RULE_CHOOSELEAF_INDEP(7)</td> </tr> <tr> <td style="text-align:left">SetVariable</td> <td style="text-align:left">CRUSH_RULE_SET_CHOOSE_TRIES(8)</td> </tr> <tr> <td style="text-align:left">SetVariable</td> <td style="text-align:left">CRUSH_RULE_SET_CHOOSELEAF_TRIES(9)</td> </tr> <tr> <td style="text-align:left">SetVariable</td> <td style="text-align:left">CRUSH_RULE_SET_CHOOSE_LOCAL_TRIES(10)</td> </tr> <tr> <td style="text-align:left">SetVariable</td> <td style="text-align:left">CRUSH_RULE_SET_CHOOSE_LOCAL_FALLBACK_TRIES(11)</td> </tr> <tr> <td style="text-align:left">SetVariable</td> <td style="text-align:left">CRUSH_RULE_SET_CHOOSELEAF_VARY_R(12)</td> </tr> </tbody> </table> 

在参数map 中crush_map
<figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">struct</span> crush_rule &#123;</span>
<span class="line"> __u32 len;</span>
<span class="line"> <span class="keyword">struct</span> crush_rule_mask mask;</span>
<span class="line"> <span class="keyword">struct</span> crush_rule_step steps[<span class="number">0</span>];</span>
<span class="line">&#125;;</span>
</pre></td></tr></table></figure>
 <figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">struct</span> crush_rule_step &#123;</span>
<span class="line"> __u32 op;</span>
<span class="line"> __s32 arg1;</span>
<span class="line"> __s32 arg2;</span>
<span class="line">&#125;;</span>
</pre></td></tr></table></figure> 

选择过程
<figure class="highlight oxygene"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">step</span> <span class="keyword">take</span> ssd</span>
<span class="line"><span class="keyword">step</span> chooseleaf firstn <span class="number">1</span> <span class="keyword">type</span> host <span class="comment">// pools size 3; 0&lt;1&lt;3 ; select 1</span></span>
<span class="line"><span class="keyword">step</span> emit</span>
<span class="line"><span class="keyword">step</span> <span class="keyword">take</span> <span class="keyword">default</span></span>
<span class="line"><span class="keyword">step</span> chooseleaf firstn -<span class="number">1</span> <span class="keyword">type</span> host <span class="comment">// -1&lt;0 ; select 3-1</span></span>
<span class="line"><span class="keyword">step</span> emit</span>
</pre></td></tr></table></figure>

对应的crush_rule_step依次是
<figure class="highlight xquery"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line"><span class="variable">$19</span> = &#123;op = <span class="number">1</span>, arg1 = -<span class="number">6</span>, arg2 = <span class="number">0</span>&#125; // -<span class="number">6</span> ssd id</span>
<span class="line"><span class="variable">$20</span> = &#123;op = <span class="number">6</span>, arg1 = <span class="number">1</span>, arg2 = <span class="number">1</span>&#125; // <span class="number">1</span></span>
<span class="line"><span class="variable">$21</span> = &#123;op = <span class="number">4</span>, arg1 = <span class="number">0</span>, arg2 = <span class="number">0</span>&#125; //</span>
<span class="line"><span class="variable">$22</span> = &#123;op = <span class="number">1</span>, arg1 = -<span class="number">1</span>, arg2 = <span class="number">0</span>&#125; // -<span class="number">1</span> <span class="keyword">default</span> id</span>
<span class="line"><span class="variable">$23</span> = &#123;op = <span class="number">6</span>, arg1 = -<span class="number">1</span>, arg2 = <span class="number">1</span>&#125; // -<span class="number">1</span></span>
<span class="line"><span class="variable">$24</span> = &#123;op = <span class="number">4</span>, arg1 = <span class="number">0</span>, arg2 = <span class="number">0</span>&#125; //</span>
</pre></td></tr></table></figure>

## [](https://ly798.github.io/2016/09/07/crush/#u53C2_u8003 "参考")参考

*   [http://my.huhoo.net/archives/2010/06/post_55.html](http://my.huhoo.net/archives/2010/06/post_55.html)
*   [http://ceph.com/papers/weil-crush-sc06.pdf](http://ceph.com/papers/weil-crush-sc06.pdf)
*   [http://way4ever.com/?p=123](http://way4ever.com/?p=123)
*   [http://my.oschina.net/linuxhunter/blog/639016](http://my.oschina.net/linuxhunter/blog/639016)
*   [https://www.mirantis.com/blog/object-storage-openstack-cloud-swift-ceph/](https://www.mirantis.com/blog/object-storage-openstack-cloud-swift-ceph/)<!-- markdown-toc end -->
