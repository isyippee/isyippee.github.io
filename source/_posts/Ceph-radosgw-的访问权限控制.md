---
title: Ceph radosgw 的访问权限控制
tags: [ceph]
date: 2015-12-14 10:42:42
---

## [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u4ECB_u7ECD "介绍")介绍

ACL(Access Control List)，访问控制列表。用于控制用户或组对资源的访问权限，资源指的是桶、对象。任何情况下桶或对象的所有者都对其拥有所有的权限。 
 <!-- more --> 

可以通过两种方式来操作权限：

*   直接对桶、对象的ACL进行操作
*   在创建桶、对象的时候设置权限 

## [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u76F4_u63A5_u64CD_u4F5CACL "直接操作ACL")直接操作ACL

### [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u89D2_u8272 "角色")角色

角色是用来被赋予资源的权限。有组和用户之分。

*   用户，radosgw中的用户，通过ID和DisplayName来指定。
*   组，一系列的用户，通过一个URL来指定。 

有如下：
 <table> <thead> <tr> <th style="text-align:left">角色</th> <th style="text-align:left">描述</th> <th style="text-align:left">示例</th> </tr> </thead> <tbody> <tr> <td style="text-align:left">认证用户</td> <td style="text-align:left">radosgw中注册的用户。</td> <td style="text-align:left">`&lt;ID&gt;user&lt;/ID&gt;&lt;DisplayName&gt;user&lt;/DisplayName&gt;`</td> </tr> <tr> <td style="text-align:left">认证的用户组</td> <td style="text-align:left">即所有的radosgw的用户</td> <td style="text-align:left">`&lt;URI&gt;http://acs.amazonaws.com/groups/global/AuthenticatedUsers&lt;/URI&gt;`</td> </tr> <tr> <td style="text-align:left">所有的用户组</td> <td style="text-align:left">包括radosgw用户和匿名用户。</td> <td style="text-align:left">`&lt;URI&gt;http://acs.amazonaws.com/groups/global/AllUsers&lt;/URI&gt;`</td> </tr> </tbody> </table> 

### [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u6743_u9650 "权限")权限

对于资源的权限有5种：
 <table> <thead> <tr> <th style="text-align:left">权限</th> <th style="text-align:left">授权给桶</th> <th style="text-align:left">授权给对象</th> </tr> </thead> <tbody> <tr> <td style="text-align:left">READ</td> <td style="text-align:left">列出对象、元数据</td> <td style="text-align:left">获取对象内容和元数据</td> </tr> <tr> <td style="text-align:left">WRITE</td> <td style="text-align:left">上传、删除对象</td> <td style="text-align:left">N/A</td> </tr> <tr> <td style="text-align:left">READ_ACP</td> <td style="text-align:left">获取桶的ACL</td> <td style="text-align:left">获取对象的ACL</td> </tr> <tr> <td style="text-align:left">WRITE_ACP</td> <td style="text-align:left">创建、修改桶的ACL</td> <td style="text-align:left">创建、修改对象的ACL</td> </tr> <tr> <td style="text-align:left">FULL_CONTROL</td> <td style="text-align:left">拥有以上所有权限</td> <td style="text-align:left">拥有以上所有权限</td> </tr> </tbody> </table> 

### [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u6D88_u606F_u4F53 "消息体")消息体

请求的消息体的格式如下：
 <figure class="highlight xml"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="pi">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span></span>
<span class="line"><span class="tag">&lt;<span class="title">AccessControlPolicy</span> <span class="attribute">xmlns</span>=<span class="value">"http://s3.amazonaws.com/doc/2006-03-01/"</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">Owner</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">ID</span>&gt;</span>&#123;id&#125;<span class="tag">&lt;/<span class="title">ID</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">DisplayName</span>&gt;</span>&#123;displayname&#125;<span class="tag">&lt;/<span class="title">DisplayName</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;/<span class="title">Owner</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">AccessControlList</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">Grant</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">Grantee</span> <span class="attribute">xmlns:xsi</span>=<span class="value">"http://www.w3.org/2001/XMLSchema-instance"</span> <span class="attribute">xsi:type</span>=<span class="value">"Canonical User"</span>&gt;</span></span>
<span class="line"> &#123;grantee&#125;</span>
<span class="line"> <span class="tag">&lt;/<span class="title">Grantee</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;<span class="title">Permission</span>&gt;</span>&#123;permission&#125;<span class="tag">&lt;/<span class="title">Permission</span>&gt;</span></span>
<span class="line"> <span class="tag">&lt;/<span class="title">Grant</span>&gt;</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="tag">&lt;/<span class="title">AccessControlList</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;/<span class="title">AccessControlPolicy</span>&gt;</span></span>
</pre></td></tr></table></figure> 

其中`&lt;Owner&gt;`指其所有者；`&lt;Grant&gt;`对应一个ACL，可以有多个；`&lt;Grantee&gt;`对应用户或用户组，`&lt;Permission&gt;`对应权限。

`&lt;Grantee&gt;`的四种格式：

*   认证用户
 <figure class="highlight xml"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="tag">&lt;<span class="title">Grantee</span> <span class="attribute">xmlns:xsi</span>=<span class="value">"http://www.w3.org/2001/XMLSchema-instance"</span> <span class="attribute">xsi:type</span>=<span class="value">"CanonicalUser"</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;<span class="title">ID</span>&gt;</span>&#123;id&#125;<span class="tag">&lt;/<span class="title">ID</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;<span class="title">DisplayName</span>&gt;</span>&#123;displayname&#125;<span class="tag">&lt;/<span class="title">DisplayName</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;/<span class="title">Grantee</span>&gt;</span></span>
</pre></td></tr></table></figure> 

*   认证的用户组
 <figure class="highlight xml"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="tag">&lt;<span class="title">Grantee</span> <span class="attribute">xmlns:xsi</span>=<span class="value">"http://www.w3.org/2001/XMLSchema-instance"</span> <span class="attribute">xsi:type</span>=<span class="value">"Group"</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;<span class="title">URI</span>&gt;</span>http://acs.amazonaws.com/groups/global/AuthenticatedUsers<span class="tag">&lt;/<span class="title">URI</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;/<span class="title">Grantee</span>&gt;</span></span>
</pre></td></tr></table></figure> 

*   所有的用户组
 <figure class="highlight xml"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="tag">&lt;<span class="title">Grantee</span> <span class="attribute">xmlns:xsi</span>=<span class="value">"http://www.w3.org/2001/XMLSchema-instance"</span> <span class="attribute">xsi:type</span>=<span class="value">"Group"</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;<span class="title">URI</span>&gt;</span>http://acs.amazonaws.com/groups/global/AllUsers<span class="tag">&lt;/<span class="title">URI</span>&gt;</span></span>
<span class="line"><span class="tag">&lt;/<span class="title">Grantee</span>&gt;</span></span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u521B_u5EFA_u65F6_u8BBE_u7F6EACL "创建时设置ACL")创建时设置ACL

创建桶、对象的时候可以使用一组标准的ACL，每个标准的ACL都包含也线设定好的控制策略。通过请求头x-amz-acl来指定。

对应描述如下：
 <table> <thead> <tr> <th style="text-align:left">标准ACL</th> <th style="text-align:left">适用资源</th> <th style="text-align:left">描述</th> </tr> </thead> <tbody> <tr> <td style="text-align:left">private</td> <td style="text-align:left">桶、对象</td> <td style="text-align:left">除了所有者拥有所有权限，其他用户都没有权限</td> </tr> <tr> <td style="text-align:left">public-read</td> <td style="text-align:left">桶、对象</td> <td style="text-align:left">除了所有者拥有所有权限，所有的用户组拥有对其的READ权限</td> </tr> <tr> <td style="text-align:left">public-read-write</td> <td style="text-align:left">桶、对象</td> <td style="text-align:left">除了所有者拥有所有权限，所有的用户组拥有对其的READ、WRITE权限</td> </tr> <tr> <td style="text-align:left">authenticated-read</td> <td style="text-align:left">桶、对象</td> <td style="text-align:left">除了所有者拥有所有权限，认证的用户组拥有对其的READ权限</td> </tr> <tr> <td style="text-align:left">bucket-owner-read</td> <td style="text-align:left">对象</td> <td style="text-align:left">除了所有者拥有所有权限，该对象所在桶的所有者拥有READ权限</td> </tr> <tr> <td style="text-align:left">bucket-owner-full-control</td> <td style="text-align:left">对象</td> <td style="text-align:left">除了所有者拥有所有权限，该对象所在桶的所有者拥有所有权限</td> </tr> <tr> <td style="text-align:left">log-delivery-write</td> <td style="text-align:left">桶</td> <td style="text-align:left">日志传输组拥有对其WRITE、READ_ACP权限</td> </tr> </tbody> </table> 

## [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u603B_u7ED3 "总结")总结

*   桶、对象的访问控制是平级的，相互没有包含关系。
也就是说若限制了桶的权限，并不代表不能操作桶内的对象；
若放开了桶的权限，并不代表就能操作桶内的对象。
*   系统默认给桶、对象的权限是其所有者拥有所有权限，也就是标准ACL中的private。
*   标准ACL中bucket-owner-read的意义。
当其他用户对你开放了其某个桶的写权限，你在其创建的对象的所有者是你，
你对这个对象拥有所有权限，而桶的所有者对这个对象没有任何权限，这个时候就需要这个标准ACL。
bucket-owner-full-control类似。 

## [](https://ly798.github.io/2015/12/14/Ceph-radosgw-%E7%9A%84%E8%AE%BF%E9%97%AE%E6%9D%83%E9%99%90%E6%8E%A7%E5%88%B6/#u53C2_u8003 "参考")参考

AmazonS3 Doc：[http://docs.aws.amazon.com/zh_cn/AmazonS3/latest/dev/acl-overview.html](http://docs.aws.amazon.com/zh_cn/AmazonS3/latest/dev/acl-overview.html)
