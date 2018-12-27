---
title: Radosgw process_request
tags: [ceph]
date: 2016-02-26 10:26:47
---

## [](https://ly798.github.io/2016/02/26/Radosgw-prcess_request/#u4ECB_u7ECD "介绍")介绍

rgw_main.cc:512，方法 process_request 用于处理请求。
 <!-- more --> 

## [](https://ly798.github.io/2016/02/26/Radosgw-prcess_request/#u5206_u6790 "分析")分析
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">static</span> <span class="keyword">int</span> <span class="title">process_request</span><span class="params">(RGWRados *store, RGWREST *rest, RGWRequest *req, RGWClientIO *client_io, OpsLogSocket *olog)</span></span>
<span class="line"></span>&#123;</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> dout(<span class="number">1</span>) &lt;&lt; <span class="string">"====== starting new request req="</span> &lt;&lt; hex &lt;&lt; req &lt;&lt; dec &lt;&lt; <span class="string">" ====="</span> &lt;&lt; dendl;</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"initializing"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> RGWHandler *handler = rest-&gt;get_handler(store, s, client_io, &amp;mgr, &amp;init_error);</span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"getting op"</span>);</span>
<span class="line"> op = handler-&gt;get_op(store);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;op = op;</span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"authorizing"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> ret = handler-&gt;authorize();</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"reading permissions"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"init op"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"verifying op mask"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"verifying op permissions"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"verifying op params"</span>);</span>
<span class="line"> <span class="comment">//...</span></span>
<span class="line"> req-&gt;<span class="built_in">log</span>(s, <span class="string">"executing"</span>);</span>
<span class="line"> op-&gt;pre_exec();</span>
<span class="line"> op-&gt;execute();</span>
<span class="line"> op-&gt;complete();</span>
<span class="line">done:</span>
<span class="line"> <span class="comment">//...错误处理</span></span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure> 

`RGWHandler *handler = rest-&gt;get_handler(store, s, client_io, &amp;mgr, &amp;init_error);`对应的是
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
</pre></td><td class="code"><pre><span class="line">RGWHandler *RGWRESTMgr_S3::get_handler(<span class="keyword">struct</span> req_state *s)</span>
<span class="line">&#123;</span>
<span class="line"> <span class="keyword">int</span> ret = RGWHandler_ObjStore_S3::init_from_header(s, RGW_FORMAT_XML, <span class="literal">false</span>);</span>
<span class="line"> <span class="keyword">if</span> (ret &lt; <span class="number">0</span>)</span>
<span class="line"> <span class="keyword">return</span> <span class="literal">NULL</span>;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (s-&gt;bucket_name_str.empty())</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">new</span> RGWHandler_ObjStore_Service_S3;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (!s-&gt;object)</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">new</span> RGWHandler_ObjStore_Bucket_S3;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">new</span> RGWHandler_ObjStore_Obj_S3;</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>

`op = handler-&gt;get_op(store);`对应的是 `rgw_op.cc:3134`
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
<span class="line">20</span>
<span class="line">21</span>
<span class="line">22</span>
<span class="line">23</span>
<span class="line">24</span>
<span class="line">25</span>
<span class="line">26</span>
<span class="line">27</span>
<span class="line">28</span>
</pre></td><td class="code"><pre><span class="line">RGWOp *RGWHandler::get_op(RGWRados *store)</span>
<span class="line">&#123;</span>
<span class="line"> RGWOp *op;</span>
<span class="line"> <span class="keyword">switch</span> (s-&gt;op) &#123;</span>
<span class="line"> <span class="keyword">case</span> OP_GET:</span>
<span class="line"> op = op_get();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">case</span> OP_PUT:</span>
<span class="line"> op = op_put();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">case</span> OP_DELETE:</span>
<span class="line"> op = op_delete();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">case</span> OP_HEAD:</span>
<span class="line"> op = op_head();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">case</span> OP_POST:</span>
<span class="line"> op = op_post();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">case</span> OP_COPY:</span>
<span class="line"> op = op_copy();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">case</span> OP_OPTIONS:</span>
<span class="line"> op = op_options();</span>
<span class="line"> <span class="keyword">break</span>;</span>
<span class="line"> <span class="keyword">default</span>:</span>
<span class="line"> <span class="keyword">return</span> <span class="literal">NULL</span>;</span>
<span class="line"> &#125;</span>
</pre></td></tr></table></figure>

这里的`op_xxx()`在`rgw_rest_s3.cc`里对应的 handler 的 `op_xxx()`,
如当使用 s3 的借口请求`s3cmd ls`时，op 对应 `RGWListBuckets_ObjStore_S3`…，这些类定义在 `rgw_rest_s3.h`
在真正进行处理的的一些方法
<figure class="highlight c++"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">op-&gt;pre_exec();</span>
<span class="line">op-&gt;execute();</span>
<span class="line">op-&gt;complete();</span>
</pre></td></tr></table></figure>

均是在`rgw_op.cc`中实现。
