---
title: rgw创建rpm源仓库
tags: [ceph]
date: 2016-09-05 10:19:51
---

## [](https://ly798.github.io/2016/09/05/rgw%E5%88%9B%E5%BB%BArpm%E6%BA%90%E4%BB%93%E5%BA%93/#u4ECB_u7ECD "介绍")介绍

对象存储以http的方式对外提供服务，那么是可以用来创建rpm源仓库。
 <!-- more --> 

## [](https://ly798.github.io/2016/09/05/rgw%E5%88%9B%E5%BB%BArpm%E6%BA%90%E4%BB%93%E5%BA%93/#u6B65_u9AA4 "步骤")步骤

### [](https://ly798.github.io/2016/09/05/rgw%E5%88%9B%E5%BB%BArpm%E6%BA%90%E4%BB%93%E5%BA%93/#u521B_u5EFA_u672C_u5730_u6E90 "创建本地源")创建本地源

在一个centos的系统上获取rpm包用来后面的测试，
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># yum install --downloadonly redis</span></span>
<span class="line"><span class="preprocessor"># ls /var/cache/yum/x86_64/<span class="number">7</span>/epel/packages/</span></span>
<span class="line">jemalloc-<span class="number">3.6</span><span class="number">.0</span>-<span class="number">1.</span>el7.x86_64.rpm redis-<span class="number">2.8</span><span class="number">.19</span>-<span class="number">2.</span>el7.x86_64.rpm</span>
</pre></td></tr></table></figure>

本地系统是ubuntu，使用`createrepo`在本地创建一个源仓库，
<figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># apt-get install createrepo</span></span>
<span class="line"><span class="preprocessor"># mkdir repo</span></span>
<span class="line"><span class="preprocessor"># cd repo</span></span>
<span class="line"><span class="preprocessor"># scp root@centos:/var/cache/yum/x86_64/7/epel/packages/* .</span></span>
<span class="line"><span class="preprocessor"># createrepo -v .</span></span>
</pre></td></tr></table></figure>

### [](https://ly798.github.io/2016/09/05/rgw%E5%88%9B%E5%BB%BArpm%E6%BA%90%E4%BB%93%E5%BA%93/#u540C_u6B65_u5230rgw "同步到rgw")同步到rgw

将该目录使用`s3cmd`同步到rgw
<figure class="highlight vala"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># cd ..</span></span>
<span class="line"><span class="preprocessor"># s3cmd -c myconf sync repo s3://mirrors</span></span>
</pre></td></tr></table></figure>

应该保证该bucket下的所有对象，包括repodata下的文件，acl为`public-read`
<figure class="highlight brainfuck"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment">#</span> <span class="comment">s3cmd</span> <span class="literal">-</span><span class="comment">c</span> <span class="comment">myconf</span> <span class="comment">setacl</span> <span class="literal">-</span><span class="literal">-</span><span class="comment">verbose</span> <span class="literal">-</span><span class="literal">-</span><span class="comment">acl</span><span class="literal">-</span><span class="comment">public</span> <span class="literal">-</span><span class="literal">-</span><span class="comment">recursive</span> <span class="comment">s3://mirrors/repo/</span></span>
</pre></td></tr></table></figure>

## [](https://ly798.github.io/2016/09/05/rgw%E5%88%9B%E5%BB%BArpm%E6%BA%90%E4%BB%93%E5%BA%93/#u9A8C_u8BC1 "验证")验证

给centos配置源：
<figure class="highlight ini"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># vim /etc/yum.repos.d/repotest.repo</span></span>
<span class="line"><span class="title">[repo]</span></span>
<span class="line"><span class="setting">name=<span class="value">repo</span></span></span>
<span class="line"><span class="setting">baseurl=<span class="value">http://mirrors.rgw.xxx.com/repo/</span></span></span>
</pre></td></tr></table></figure>

验证：
<figure class="highlight cpp"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="preprocessor"># yum repolist</span></span>
<span class="line">源标识 源名称 状态</span>
<span class="line">repo repo <span class="number">2</span></span>
<span class="line"></span>
<span class="line"><span class="preprocessor"># yum whatprovides redis</span></span>
<span class="line">redis-<span class="number">2.8</span><span class="number">.19</span>-<span class="number">2.</span>el7.x86_64 : A persistent key-value database</span>
<span class="line">源 ：epel</span>
<span class="line">redis-<span class="number">2.8</span><span class="number">.19</span>-<span class="number">2.</span>el7.x86_64 : A persistent key-value database</span>
<span class="line">源 ：repo</span>
</pre></td></tr></table></figure>
