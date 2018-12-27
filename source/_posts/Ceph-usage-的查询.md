---
title: Ceph usage 的查询
tags: [ceph]
date: 2016-01-04 10:38:29
---

## [](https://ly798.github.io/2016/01/04/Ceph-usage-%E7%9A%84%E6%9F%A5%E8%AF%A2/#u4ECB_u7ECD "介绍")介绍

usage 用于统计用户的使用情况，操作记录。
 <!-- more --> 

## [](https://ly798.github.io/2016/01/04/Ceph-usage-%E7%9A%84%E6%9F%A5%E8%AF%A2/#admin_commands "admin commands")admin commands
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">radosgw-admin usage show [--uid=&#123;uid&#125;] [--start-date=&#123;date&#125;] [--end-date=&#123;date&#125;] [--categories=&lt;list&gt;] [--show-log-entries=&lt;flag&gt;] [--show-log-sum=&lt;flag&gt;]</span>
</pre></td></tr></table></figure> 

example:
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">radosgw-admin usage show --uid=demo --start-date=<span class="string">"2015-12-01 08:00:00"</span> --end-date=<span class="string">"2015-12-02 08:00:00"</span> --categories=<span class="string">"put_obj,put_acls"</span></span>
</pre></td></tr></table></figure>

*   --start-date、--categories 中含需转义字符，使用” “即可。
*   --show-log-entries=true/false 即usage详细条目，--show-log-sum=true/false 即usage概览
*   关于 –categories，有如下：<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">radosgw-admin usage show | grep category | awk -F <span class="string">'"'</span> <span class="string">'&#123;print $4&#125;'</span> | sort | uniq</span>
<span class="line">abort_multipart</span>
<span class="line">complete_multipart</span>
<span class="line">copy_obj</span>
<span class="line">create_bucket</span>
<span class="line">delete_bucket</span>
<span class="line">delete_obj</span>
<span class="line">get_acls</span>
<span class="line">get_bucket_logging</span>
<span class="line">get_cors</span>
<span class="line">get_obj</span>
<span class="line">init_multipart</span>
<span class="line">list_bucket</span>
<span class="line">list_bucket_multiparts</span>
<span class="line">list_buckets</span>
<span class="line">list_multipart</span>
<span class="line">multi_object_delete</span>
<span class="line">options_cors</span>
<span class="line">post_obj</span>
<span class="line">put_acls</span>
<span class="line">put_obj</span>
<span class="line"><span class="built_in">stat</span>_bucket</span>
</pre></td></tr></table></figure> 

## [](https://ly798.github.io/2016/01/04/Ceph-usage-%E7%9A%84%E6%9F%A5%E8%AF%A2/#admin_restapi "admin restapi")admin restapi

请求
<figure class="highlight http"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="request">GET <span class="string">/admin/usage</span> HTTP/1.1</span></span>
<span class="line"><span class="attribute">Host</span>: <span class="string">host</span></span>
<span class="line"><span class="attribute">date</span>: <span class="string">Date</span></span>
<span class="line"><span class="attribute">Authorization</span>: <span class="string">auth</span></span>
</pre></td></tr></table></figure>

请求参数
<figure class="highlight sql"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">uid,<span class="operator"><span class="keyword">start</span>,<span class="keyword">end</span>,<span class="keyword">show</span>-entries,<span class="keyword">show</span>-summary</span></span>
</pre></td></tr></table></figure>

example
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
</pre></td><td class="code"><pre><span class="line">relativePath=<span class="string">"/admin/usage"</span></span>
<span class="line">start=<span class="string">"2015-12-20%2020%3A00%3A00"</span> <span class="comment"># : 和空格转义</span></span>
<span class="line">curl <span class="operator">-s</span> -v -X GET <span class="string">"http://<span class="variable">$&#123;HOST&#125;</span><span class="variable">$&#123;relativePath&#125;</span>?start=<span class="variable">$&#123;start&#125;</span>&amp;show-entries=true&amp;show-summary=false"</span> \</span>
<span class="line">-H <span class="string">"Authorization: AWS <span class="variable">$&#123;KEY_ID&#125;</span>:<span class="variable">$&#123;signature&#125;</span>"</span> \</span>
<span class="line">-H <span class="string">"Date: <span class="variable">$&#123;DATE&#125;</span>"</span> \</span>
<span class="line">-H <span class="string">"Host: <span class="variable">$&#123;HOST&#125;</span>"</span></span>
</pre></td></tr></table></figure>

## [](https://ly798.github.io/2016/01/04/Ceph-usage-%E7%9A%84%E6%9F%A5%E8%AF%A2/#code "code")code

1.  rgw_admin.cc:main()
 <figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">else</span> <span class="keyword">if</span> (ceph_argparse_witharg(args, i, &amp;val, <span class="string">"--date"</span>, <span class="string">"--time"</span>, (<span class="keyword">char</span>*)<span class="literal">NULL</span>)) &#123;</span>
<span class="line"> date = val;</span>
<span class="line"> <span class="keyword">if</span> (end_date.empty())</span>
<span class="line"> end_date = date;</span>
<span class="line">&#125; <span class="keyword">else</span> <span class="keyword">if</span> (ceph_argparse_witharg(args, i, &amp;val, <span class="string">"--start-date"</span>, <span class="string">"--start-time"</span>, (<span class="keyword">char</span>*)<span class="literal">NULL</span>)) &#123;</span>
<span class="line"> start_date = val;</span>
<span class="line">&#125; <span class="keyword">else</span> <span class="keyword">if</span> (ceph_argparse_witharg(args, i, &amp;val, <span class="string">"--end-date"</span>, <span class="string">"--end-time"</span>, (<span class="keyword">char</span>*)<span class="literal">NULL</span>)) &#123;</span>
<span class="line"> end_date = val;</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>
        *   可以用 --date 或 --time 来指定 end_date
    *   --*-date 也可以用 --*-time 来指定2.  utime_t::parse_date()
 <figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">const</span> <span class="keyword">char</span> *p = strptime(date.c_str(), <span class="string">"%Y-%m-%d"</span>, &amp;tm);</span>
</pre></td></tr></table></figure>
        *   strptime() 函数成功返回 *buff 的最后位置，失败返回空指针。
    *   当 date 是 “%Y-%m-%d %H:%M:%S” 时，*p 为字符 “ “，先转换 %Y-%m-%d，当 *p== “ “，说明还有 %H:%M:%S
    *   date 可以是 “%Y-%m-%d” 或者 “%Y-%m-%d %H:%M:%S”3.  rgw_log.cc:125
存储一个usage时调用的方法
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">void</span> <span class="title">insert</span><span class="params">(utime_t&amp; timestamp, rgw_usage_log_entry&amp; entry)</span> </span>&#123;</span>
<span class="line"> lock.Lock();</span>
<span class="line"> <span class="keyword">if</span> (timestamp.sec() &gt; round_timestamp + <span class="number">3600</span>)</span>
<span class="line"> recalc_round_timestamp(timestamp);</span>
<span class="line"> entry.epoch = round_timestamp.sec();</span>
<span class="line"> <span class="keyword">bool</span> account;</span>
<span class="line"> <span class="function">rgw_user_bucket <span class="title">ub</span><span class="params">(entry.owner, entry.bucket)</span></span>;</span>
<span class="line"> usage_map[ub].insert(round_timestamp, entry, &amp;account);</span>
<span class="line"> <span class="keyword">if</span> (account)</span>
<span class="line"> num_entries++;</span>
<span class="line"> <span class="keyword">bool</span> need_flush = (num_entries &gt; cct-&gt;_conf-&gt;rgw_usage_log_flush_threshold);</span>
<span class="line"> lock.Unlock();</span>
<span class="line"> <span class="keyword">if</span> (need_flush) &#123;</span>
<span class="line"> Mutex::<span class="function">Locker <span class="title">l</span><span class="params">(timer_lock)</span></span>;</span>
<span class="line"> flush();</span>
<span class="line"> &#125;</span>
<span class="line"> &#125;</span>
</pre></td></tr></table></figure> 

对应的recalc_round_timestamp()
<figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">void</span> <span class="title">recalc_round_timestamp</span><span class="params">(utime_t&amp; ts)</span> </span>&#123;</span>
<span class="line"> round_timestamp = ts.round_to_hour();</span>
<span class="line"> &#125;</span>
</pre></td></tr></table></figure>

最终的 usage_map[ub].insert(round_timestamp, entry, &amp;account); 中的 round_timestamp 是以小时为单位。

## [](https://ly798.github.io/2016/01/04/Ceph-usage-%E7%9A%84%E6%9F%A5%E8%AF%A2/#u603B_u7ED3 "总结")总结

usage 的查询可以时间可以指定到时分秒，但事实上 usage 的存储是以小时为单位的，也就是说可以精确到小时。
time 的 start、end 遵循左闭右开的规则。
该时间是 UTC 时间。
