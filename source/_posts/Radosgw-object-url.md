---
title: Radosgw object url
tags: [ceph]
date: 2016-02-22 10:25:50
---

## [](https://ly798.github.io/2016/02/22/Radosgw-object-url/#u4ECB_u7ECD "介绍")介绍

ceph 对象存储中的对象的 url 是通过本地计算得来的，无论是权限为所有人可读还是私有的。
 <!-- more --> 

## [](https://ly798.github.io/2016/02/22/Radosgw-object-url/#u516C_u6709 "公有")公有

如对象存储 url 为 `obs.eayun.com`
对象 `bucket/object` 的 url 就为 `http://bucket.obs.eayun.com/object`

## [](https://ly798.github.io/2016/02/22/Radosgw-object-url/#u79C1_u6709 "私有")私有

计算似有对象的 url 需要一些变量：

*   access_key:用户的 access_key
*   secret_key:用户的 secret_key
*   timeStamp:该对象过期时间点的unix时间戳
*   signature:radosgw 中的认证 signature 

如对象存储 url 为 `obs.eayun.com`
以 shell 脚本为例
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line"><span class="shebang">#!/usr/bin/env bash</span></span>
<span class="line">KEY_ACCESS=<span class="string">"xxx"</span></span>
<span class="line">KEY_SECRET=<span class="string">"xxx"</span></span>
<span class="line"></span>
<span class="line">FILE=<span class="string">"object"</span></span>
<span class="line">BUCKET=<span class="string">"bucket"</span></span>
<span class="line">relativePath=<span class="string">"/<span class="variable">$&#123;BUCKET&#125;</span>/<span class="variable">$&#123;FILE&#125;</span>"</span></span>
<span class="line">contentType=<span class="string">""</span></span>
<span class="line">ContentMD5=<span class="string">""</span></span>
<span class="line"></span>
<span class="line">current=`date <span class="string">"+%Y-%m-%d %H:%M:%S"</span>`</span>
<span class="line">timeStamp=`date <span class="operator">-d</span> <span class="string">"<span class="variable">$current</span>"</span> +%s` <span class="comment">#将current转换为时间戳，精确到秒</span></span>
<span class="line"></span>
<span class="line">stringToSign=<span class="string">"GET\n<span class="variable">$&#123;ContentMD5&#125;</span>\n<span class="variable">$&#123;contentType&#125;</span>\n<span class="variable">$&#123;timeStamp&#125;</span>\n<span class="variable">$&#123;relativePath&#125;</span>"</span></span>
<span class="line"></span>
<span class="line">signature=`<span class="built_in">echo</span> -en <span class="variable">$&#123;stringToSign&#125;</span> | openssl sha1 -hmac <span class="variable">$&#123;KEY_SECRET&#125;</span> -binary | base64`</span>
<span class="line"></span>
<span class="line">HOST=<span class="string">"<span class="variable">$&#123;BUCKET&#125;</span>.obs.eayun.com/<span class="variable">$&#123;FILE&#125;</span>"</span></span>
<span class="line"><span class="built_in">echo</span> <span class="string">"http://<span class="variable">$&#123;HOST&#125;</span>?AWSAccessKeyId=<span class="variable">$&#123;KEY_ACCESS&#125;</span>&amp;Expires=<span class="variable">$&#123;timeStamp&#125;</span>&amp;Signature=<span class="variable">$&#123;signature&#125;</span>"</span></span>
</pre></td></tr></table></figure>

对象 `bucket/object` 的 url 就为
`http://${HOST}?AWSAccessKeyId=${KEY_ID}&amp;Expires=${timeStamp}&amp;Signature=${signature}`
