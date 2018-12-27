---
title: Radosgw object 命令以及伪文件夹
tags: [ceph]
date: 2016-03-07 10:27:06
---

## [](https://ly798.github.io/2016/03/07/Radosgw-object-%E5%91%BD%E5%90%8D%E4%BB%A5%E5%8F%8A%E4%BC%AA%E6%96%87%E4%BB%B6%E5%A4%B9/#u4ECB_u7ECD "介绍")介绍

在 rgw 中，对创建的对象的命名有一定的规则，长度限制。
 <!-- more --> 

## [](https://ly798.github.io/2016/03/07/Radosgw-object-%E5%91%BD%E5%90%8D%E4%BB%A5%E5%8F%8A%E4%BC%AA%E6%96%87%E4%BB%B6%E5%A4%B9/#u5206_u6790 "分析")分析

rgw_rest.cc:1070
<figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="comment">// "The name for a key is a sequence of Unicode characters whose UTF-8 encoding</span></span>
<span class="line"><span class="comment">// is at most 1024 bytes long."</span></span>
<span class="line"><span class="comment">// However, we can still have control characters and other nasties in there.</span></span>
<span class="line"><span class="comment">// Just as long as they're utf-8 nasties.</span></span>
<span class="line"><span class="keyword">int</span> RGWHandler_ObjStore::validate_object_name(<span class="keyword">const</span> <span class="built_in">string</span>&amp; object)</span>
<span class="line">&#123;</span>
<span class="line"> <span class="keyword">int</span> len = object.size();</span>
<span class="line"> <span class="keyword">if</span> (len &gt; <span class="number">1024</span>) &#123;</span>
<span class="line"> <span class="comment">// Name too long</span></span>
<span class="line"> <span class="keyword">return</span> -ERR_INVALID_OBJECT_NAME;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (check_utf8(object.c_str(), len)) &#123;</span>
<span class="line"> <span class="comment">// Object names must be valid UTF-8.</span></span>
<span class="line"> <span class="keyword">return</span> -ERR_INVALID_OBJECT_NAME;</span>
<span class="line"> &#125;</span>
<span class="line"> <span class="keyword">return</span> <span class="number">0</span>;</span>
<span class="line">&#125;</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

object 的名字通过 rest 传过来是需要编码成 utf8，所以这里的 `len`指的是 utf8 的字节数，字节数不能超过 1024。
 > 在 utf8 中每个字母数字1个字节，大多数汉字每个占3个字节。 

### [](https://ly798.github.io/2016/03/07/Radosgw-object-%E5%91%BD%E5%90%8D%E4%BB%A5%E5%8F%8A%E4%BC%AA%E6%96%87%E4%BB%B6%E5%A4%B9/#u4F2A_u6587_u4EF6_u5939 "伪文件夹")伪文件夹

在 rgw 中是没有文件夹的概念的，但是可以通过对对象的命名通过字符`/`来分割，然后对对象进行分类分层就可以视作文件夹；

若是仅创建一个文件夹，其实质也是一个对象，其命名如 `folder/`，且该对象 Content-Length: 0；

举个例子：
<figure class="highlight haml"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">folder1/</span>
<span class="line">-<span class="ruby">---object1</span>
<span class="line"></span>-<span class="ruby">---folder2/</span>
<span class="line"></span>-<span class="ruby">---folder3/object2</span></span>
</pre></td></tr></table></figure>

在上面的目录结构中存在的对象有如下4个：
<figure class="highlight ceylon"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line">folder<span class="number">1</span>/</span>
<span class="line">folder<span class="number">1</span>/<span class="keyword">object</span><span class="number">1</span></span>
<span class="line">folder<span class="number">1</span>/folder<span class="number">2</span>/</span>
<span class="line">folder<span class="number">1</span>/folder<span class="number">3</span>/<span class="keyword">object</span><span class="number">2</span></span>
</pre></td></tr></table></figure>

上级文件夹与下级文件没有实质性的关系，如文件夹 /folder1/ 与 文件夹/folder1/folder2/，两者是相互独立的对象，只是第二个文件夹名字前缀是第一个文件夹的名字。

这里牵涉到命名长度的问题了：

*   若全使用字母数字，则最大长度为 1024-1(字符’/‘) = 1023；
*   若全是用汉字，则最大长度为 (1024-1)/3 = 341； 

所以无论多少级文件夹（对象名字中有多少个’/‘），只要保证对象名字的字节数不大于 1024 即可。
