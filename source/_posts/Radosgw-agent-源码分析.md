---
title: Radosgw-agent 源码分析
tags: [ceph]
date: 2016-01-25 10:39:42
---

## [](https://ly798.github.io/2016/01/25/Radosgw-agenti-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#u4ECB_u7ECD "介绍")介绍

在同步数据的时候分别对 source、destination 进行抓包，对应源码进行分析。
 <!-- more --> 

## [](https://ly798.github.io/2016/01/25/Radosgw-agenti-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#u6E90_u7801_u5206_u6790 "源码分析")源码分析

### [](https://ly798.github.io/2016/01/25/Radosgw-agenti-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#radosgw-agent/cli-py_3Amain_28_29 "radosgw-agent/cli.py:main()")radosgw-agent/cli.py:main()
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">main</span><span class="params">()</span>:</span></span>
<span class="line"> <span class="comment"># 设置 logger、日志等级</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="comment"># 解析 args，获得配置项</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="comment"># 获得 regionmap，对应的 api 是 /admin/config </span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> region_map = client.get_region_map(dest_conn)</span>
<span class="line"> <span class="keyword">except</span> AgentError:</span>
<span class="line"> ...</span>
<span class="line"> <span class="comment"># 配置 endpoint，其中会进行检查 master 或 secondary</span></span>
<span class="line"> client.configure_endpoints(region_map, dest, src, args.metadata_only)</span>
<span class="line"> <span class="comment"># test_server</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="comment"># 数据同步</span></span>
<span class="line"> <span class="comment">## 初始化 meta_syncer、data_syncer</span></span>
<span class="line"> ...</span>
<span class="line"> <span class="comment">## prepare_sync</span></span>
<span class="line"> sync.prepare_sync(meta_syncer, args.prepare_error_delay)</span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> args.metadata_only:</span>
<span class="line"> sync.prepare_sync(data_syncer, args.prepare_error_delay)</span>
<span class="line"> <span class="comment">## sync</span></span>
<span class="line"> <span class="keyword">if</span> args.sync_scope == <span class="string">'full'</span>:</span>
<span class="line"> log.info(<span class="string">'syncing all metadata'</span>)</span>
<span class="line"> meta_syncer.sync(args.num_workers, args.lock_timeout)</span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> args.metadata_only:</span>
<span class="line"> log.info(<span class="string">'syncing all data'</span>)</span>
<span class="line"> data_syncer.sync(args.num_workers, args.lock_timeout)</span>
<span class="line"> log.info(<span class="string">'Finished full sync. Check logs to see any issues that '</span></span>
<span class="line"> <span class="string">'incremental sync will retry.'</span>)</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> sync.incremental_sync(meta_syncer, data_syncer,</span>
<span class="line"> args.num_workers,</span>
<span class="line"> args.lock_timeout,</span>
<span class="line"> args.incremental_sync_delay,</span>
<span class="line"> args.metadata_only,</span>
<span class="line"> args.prepare_error_delay)</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/01/25/Radosgw-agenti-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#prepare_sync "prepare_sync")prepare_sync

以 meta_syncer 为例
/radosgw-agent/sync.py:class IncrementalSyncer(Syncer)
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">prepare</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="comment"># src 的 shards ，对应的方法是 self.num_shards = client.num_log_shards(self.src_conn, self.type)</span></span>
<span class="line"> <span class="comment"># 该方法使用的 api 是 /admin/log?type=metadata</span></span>
<span class="line"> <span class="comment"># 得到的结果类似 &#123;"num_objects":64&#125;</span></span>
<span class="line"> self.init_num_shards()</span>
<span class="line"> self.shard_info = &#123;&#125;</span>
<span class="line"> self.shard_work = &#123;&#125;</span>
<span class="line"> <span class="keyword">for</span> shard_num <span class="keyword">in</span> xrange(self.num_shards):</span>
<span class="line"> marker, retries = self.get_worker_bound(shard_num)</span>
<span class="line"> last_marker, log_entries = self.get_log_entries(shard_num, marker)</span>
<span class="line"> self.shard_work[shard_num] = log_entries, retries</span>
<span class="line"> self.shard_info[shard_num] = last_marker</span>
<span class="line"> self.prepared_at = time.time()</span>
</pre></td></tr></table></figure>

对于上面代码

1.  获取 dest 的 bound <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">marker, retries = self.get_worker_bound(shard_num)</span>
</pre></td></tr></table></figure>
     获取 dest 的 bound，对应的 api 是 /admin/replica_log?bounds&amp;type=data&amp;id=0
得到的结果是
 <figure class="highlight json"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">&#123;</span>
<span class="line"> "<span class="attribute">marker</span>": <span class="value"><span class="string">"1_1453184408.808048_5.1"</span></span>, </span>
<span class="line"> "<span class="attribute">oldest_time</span>": <span class="value"><span class="string">"0.000000"</span></span>, </span>
<span class="line"> "<span class="attribute">markers</span>": <span class="value">[</span>
<span class="line"> &#123;</span>
<span class="line"> "<span class="attribute">entity</span>": <span class="value"><span class="string">"radosgw-agent"</span></span>, </span>
<span class="line"> "<span class="attribute">position_marker</span>": <span class="value"><span class="string">"1_1453184408.808048_5.1"</span></span>, </span>
<span class="line"> "<span class="attribute">position_time</span>": <span class="value"><span class="string">"0.000000"</span></span>, </span>
<span class="line"> "<span class="attribute">items_in_progress</span>": <span class="value">[ ]</span>
<span class="line"> </span>&#125;</span>
<span class="line"> ]</span>
<span class="line"></span>&#125;</span>
</pre></td></tr></table></figure>
     或
 <figure class="highlight json"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">&#123;"<span class="attribute">Code</span>":<span class="value"><span class="string">"NoSuchKey"</span></span>&#125;</span>
</pre></td></tr></table></figure>
     marker, retries 都为空
2.  获取 src 的 log_entries
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">last_marker, log_entries = self.get_log_entries(shard_num, marker)</span>
</pre></td></tr></table></figure>
     获取 src 的 log_entries，对应的 api 是 /admin/log?marker=%20&amp;type=metadata&amp;id=0&amp;max-entries=1000
得到的结果一个 marker，包含许多的 entries
 <figure class="highlight json"><table><tr><td class="gutter"><pre><span class="line">1</span>
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
</pre></td><td class="code"><pre><span class="line">&#123;</span>
<span class="line"> "<span class="attribute">marker</span>": <span class="value"><span class="string">"1_1453461773.098122_674.1"</span></span>, </span>
<span class="line"> "<span class="attribute">truncated</span>": <span class="value"><span class="literal">false</span></span>, </span>
<span class="line"> "<span class="attribute">entries</span>": <span class="value">[</span>
<span class="line"> &#123;</span>
<span class="line"> "<span class="attribute">id</span>": <span class="value"><span class="string">"1_1448948065.089679_11.1"</span></span>, </span>
<span class="line"> "<span class="attribute">section</span>": <span class="value"><span class="string">"bucket.instance"</span></span>, </span>
<span class="line"> "<span class="attribute">name</span>": <span class="value"><span class="string">"pengdake_test:center-1.5447.2"</span></span>, </span>
<span class="line"> "<span class="attribute">timestamp</span>": <span class="value"><span class="string">"2015-12-01 05:34:25.089679Z"</span></span>, </span>
<span class="line"> "<span class="attribute">data</span>": <span class="value">&#123;</span>
<span class="line"> "<span class="attribute">read_version</span>": <span class="value">&#123;</span>
<span class="line"> "<span class="attribute">tag</span>": <span class="value"><span class="string">""</span></span>, </span>
<span class="line"> "<span class="attribute">ver</span>": <span class="value"><span class="number">0</span></span>
<span class="line"> </span>&#125;</span>, </span>
<span class="line"> "<span class="attribute">write_version</span>": <span class="value">&#123;</span>
<span class="line"> "<span class="attribute">tag</span>": <span class="value"><span class="string">"_yl8VxMaeEnZZpEAZE-M9vwX"</span></span>, </span>
<span class="line"> "<span class="attribute">ver</span>": <span class="value"><span class="number">1</span></span>
<span class="line"> </span>&#125;</span>, </span>
<span class="line"> "<span class="attribute">status</span>": <span class="value">&#123;</span>
<span class="line"> "<span class="attribute">status</span>": <span class="value"><span class="string">"write"</span></span>
<span class="line"> </span>&#125;</span>
<span class="line"> </span>&#125;</span>
<span class="line"> </span>&#125;, ...</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/01/25/Radosgw-agenti-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#sync "sync")sync

考虑 incremental 方式，radosgw-agent/sync.py
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">incremental_sync</span><span class="params">(meta_syncer, data_syncer, num_workers, lock_timeout,</span>
<span class="line"> incremental_sync_delay, metadata_only, error_delay)</span>:</span></span>
<span class="line"> <span class="string">"""Run a continuous incremental sync.</span>
<span class="line"></span>
<span class="line"> This will run forever, pausing between syncs by a</span>
<span class="line"> incremental_sync_delay seconds.</span>
<span class="line"> """</span></span>
<span class="line"> <span class="keyword">while</span> <span class="keyword">True</span>:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> meta_syncer.sync(num_workers, lock_timeout)</span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> metadata_only:</span>
<span class="line"> data_syncer.sync(num_workers, lock_timeout)</span>
<span class="line"> <span class="keyword">except</span> Exception:</span>
<span class="line"> log.warn(<span class="string">'error doing incremental sync, will try again. Traceback:'</span>,</span>
<span class="line"> exc_info=<span class="keyword">True</span>)</span>
<span class="line"></span>
<span class="line"> <span class="comment"># prepare data before sleeping due to rgw_log_bucket_window</span></span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> metadata_only:</span>
<span class="line"> prepare_sync(data_syncer, error_delay)</span>
<span class="line"> log.info(<span class="string">'waiting %d seconds until next sync'</span>,</span>
<span class="line"> incremental_sync_delay)</span>
<span class="line"> time.sleep(incremental_sync_delay)</span>
<span class="line"> prepare_sync(meta_syncer, error_delay)</span>
</pre></td></tr></table></figure>

1.  初始化
其中`meta_syncer.sync(num_workers, lock_timeout)`，最终方法是radosgw-agent/sync.py:class Syncer()
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">sync</span><span class="params">(self, num_workers, log_lock_time)</span>:</span></span>
<span class="line"> <span class="comment"># 初始化 multiprocessing，并启动</span></span>
<span class="line"> workQueue = multiprocessing.Queue()</span>
<span class="line"> resultQueue = multiprocessing.Queue()</span>
<span class="line"> processes = [self.worker_cls(workQueue,</span>
<span class="line"> resultQueue,</span>
<span class="line"> log_lock_time,</span>
<span class="line"> self.src,</span>
<span class="line"> self.dest,</span>
<span class="line"> daemon_id=self.daemon_id,</span>
<span class="line"> max_entries=self.max_entries,</span>
<span class="line"> object_sync_timeout=self.object_sync_timeout,</span>
<span class="line"> )</span>
<span class="line"> <span class="keyword">for</span> i <span class="keyword">in</span> xrange(num_workers)]</span>
<span class="line"> <span class="keyword">for</span> process <span class="keyword">in</span> processes:</span>
<span class="line"> process.daemon = <span class="keyword">True</span></span>
<span class="line"> process.start()</span>
<span class="line"> self.wait_until_ready()</span>
<span class="line"> log.info(<span class="string">'Starting sync'</span>)</span>
<span class="line"> <span class="comment"># enqueue the shards to be synced</span></span>
<span class="line"> <span class="comment"># 将 shards 放入队列</span></span>
<span class="line"> num_items = <span class="number">0</span></span>
<span class="line"> <span class="keyword">for</span> item <span class="keyword">in</span> self.generate_work():</span>
<span class="line"> num_items += <span class="number">1</span></span>
<span class="line"> workQueue.put(item)</span>
<span class="line"> <span class="comment"># add a poison pill for each worker</span></span>
<span class="line"> <span class="keyword">for</span> i <span class="keyword">in</span> xrange(num_workers):</span>
<span class="line"> workQueue.put(<span class="keyword">None</span>)</span>
<span class="line"> <span class="comment"># pull the results out as they are produced</span></span>
<span class="line"> <span class="comment"># 获取处理的结果</span></span>
<span class="line"> retries = &#123;&#125;</span>
<span class="line"> <span class="keyword">for</span> i <span class="keyword">in</span> xrange(num_items):</span>
<span class="line"> result, item = resultQueue.get()</span>
<span class="line"> shard_num, retries = item</span>
<span class="line"> <span class="keyword">if</span> result == worker.RESULT_SUCCESS:</span>
<span class="line"> log.debug(<span class="string">'synced item %r successfully'</span>, item)</span>
<span class="line"> self.complete_item(shard_num, retries)</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> log.error(<span class="string">'error syncing shard %d'</span>, shard_num)</span>
<span class="line"> retries.append(shard_num)</span>
<span class="line"> log.info(<span class="string">'%d/%d items processed'</span>, i + <span class="number">1</span>, num_items)</span>
<span class="line"> <span class="keyword">if</span> retries:</span>
<span class="line"> log.error(<span class="string">'Encountered errors syncing these %d shards: %r'</span>,</span>
<span class="line"> len(retries), retries)</span>
</pre></td></tr></table></figure>2.  同步
对于上 1 中的代码，`self.generate_work()`对应的是：
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">generate_work</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> self.shard_work.iteritems()</span>
</pre></td></tr></table></figure>
     这里的 shard_work 就是上述 prepare 中的 shard_work，其对应的格式是
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">shard_num, (<span class="built_in">log</span>_entries, retries)</span>
</pre></td></tr></table></figure>
     将其放入队列后，process 就的得到一个 work，进行 process.start()，对应的方法是
radosgw-agent/worker.py:class IncrementalMixin()
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">run</span><span class="params">(self)</span>:</span></span>
<span class="line"> self.prepare_lock()</span>
<span class="line"> <span class="keyword">while</span> <span class="keyword">True</span>:</span>
<span class="line"> <span class="comment"># 获得 shard_work 中的一个</span></span>
<span class="line"> item = self.work_queue.get()</span>
<span class="line"> <span class="keyword">if</span> item <span class="keyword">is</span> <span class="keyword">None</span>:</span>
<span class="line"> dev_log.info(<span class="string">'process %s is done. Exiting'</span>, self.ident)</span>
<span class="line"> <span class="keyword">break</span></span>
<span class="line"> shard_num, (log_entries, retries) = item</span>
<span class="line"> log.info(<span class="string">'%s is processing shard number %d'</span>,</span>
<span class="line"> self.ident, shard_num)</span>
<span class="line"> <span class="comment"># first, lock the log</span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> self.lock_shard(shard_num)</span>
<span class="line"> <span class="keyword">except</span> SkipShard:</span>
<span class="line"> <span class="keyword">continue</span></span>
<span class="line"> result = RESULT_SUCCESS</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="comment"># 同步</span></span>
<span class="line"> new_retries = self.sync_entries(log_entries, retries)</span>
<span class="line"> <span class="keyword">except</span> Exception:</span>
<span class="line"> log.exception(<span class="string">'syncing entries for shard %d failed'</span>,</span>
<span class="line"> shard_num)</span>
<span class="line"> result = RESULT_ERROR</span>
<span class="line"> new_retries = []</span>
<span class="line"> <span class="comment"># finally, unlock the log</span></span>
<span class="line"> self.unlock_shard()</span>
<span class="line"> self.result_queue.put((result, (shard_num, new_retries)))</span>
<span class="line"> log.info(<span class="string">'finished processing shard %d'</span>, shard_num)</span>
</pre></td></tr></table></figure>
     如上代码中`new_retries = self.sync_entries(log_entries, retries)`对应的是
radosgw-agent/worker.py:class IncrementalMixin()
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">sync_entries</span><span class="params">(self, log_entries, retries)</span>:</span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="comment"># 整理 entries 中的每一个 entry</span></span>
<span class="line"> entries = [_meta_entry_from_json(entry) <span class="keyword">for</span> entry <span class="keyword">in</span> log_entries]</span>
<span class="line"> <span class="keyword">except</span> KeyError:</span>
<span class="line"> log.error(<span class="string">'log containing bad key is: %s'</span>, log_entries)</span>
<span class="line"> <span class="keyword">raise</span></span>
<span class="line"> new_retries = []</span>
<span class="line"> <span class="comment"># 取 entry 的 [section, name] 并去重</span></span>
<span class="line"> mentioned = set([(entry.section, entry.name) <span class="keyword">for</span> entry <span class="keyword">in</span> entries])</span>
<span class="line"> <span class="comment"># 整理 bound</span></span>
<span class="line"> split_retries = [tuple(entry.split(<span class="string">'/'</span>, <span class="number">1</span>)) <span class="keyword">for</span> entry <span class="keyword">in</span> retries]</span>
<span class="line"> <span class="comment"># 组合并同步</span></span>
<span class="line"> <span class="keyword">for</span> section, name <span class="keyword">in</span> mentioned.union(split_retries):</span>
<span class="line"> sync_result = self.sync_meta(section, name)</span>
<span class="line"> <span class="comment"># 同步失败的将返回，下一次将重试</span></span>
<span class="line"> <span class="keyword">if</span> sync_result == RESULT_ERROR:</span>
<span class="line"> new_retries.append(section + <span class="string">'/'</span> + name)</span>
<span class="line"> <span class="keyword">return</span> new_retries</span>
</pre></td></tr></table></figure>
     如上代码中`sync_result = self.sync_meta(section, name)`对应的是
radosgw-agent/worker.py:class MetadataWorker(Worker)
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">sync_meta</span><span class="params">(self, section, name)</span>:</span></span>
<span class="line"> log.debug(<span class="string">'syncing metadata type %s key "%s"'</span>, section, name)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="comment"># 获取 source 的 某个 metadata</span></span>
<span class="line"> metadata = client.get_metadata(self.src_conn, section, name)</span>
<span class="line"> <span class="keyword">except</span> NotFound:</span>
<span class="line"> log.debug(<span class="string">'%s "%s" not found on master, deleting from secondary'</span>,</span>
<span class="line"> section, name)</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="comment"># 若没有找到，则在 dest 上删除</span></span>
<span class="line"> client.delete_metadata(self.dest_conn, section, name)</span>
<span class="line"> <span class="keyword">except</span> NotFound:</span>
<span class="line"> <span class="comment"># Since this error is handled appropriately, return success</span></span>
<span class="line"> <span class="keyword">return</span> RESULT_SUCCESS</span>
<span class="line"> <span class="keyword">except</span> Exception <span class="keyword">as</span> e:</span>
<span class="line"> log.warn(<span class="string">'error getting metadata for %s "%s": %s'</span>,</span>
<span class="line"> section, name, e, exc_info=<span class="keyword">True</span>)</span>
<span class="line"> <span class="keyword">return</span> RESULT_ERROR</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="comment"># 更新 metadata</span></span>
<span class="line"> client.update_metadata(self.dest_conn, section, name, metadata)</span>
<span class="line"> <span class="keyword">return</span> RESULT_SUCCESS</span>
<span class="line"> <span class="keyword">except</span> Exception <span class="keyword">as</span> e:</span>
<span class="line"> log.warn(<span class="string">'error updating metadata for %s "%s": %s'</span>,</span>
<span class="line"> section, name, e, exc_info=<span class="keyword">True</span>)</span>
<span class="line"> <span class="keyword">return</span> RESULT_ERROR</span>
</pre></td></tr></table></figure>3.  获取处理结果
其代码是
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
</pre></td><td class="code"><pre><span class="line">retries = &#123;&#125;</span>
<span class="line"> <span class="keyword">for</span> i <span class="keyword">in</span> xrange(num_items):</span>
<span class="line"> result, item = resultQueue.get()</span>
<span class="line"> shard_num, retries = item</span>
<span class="line"> <span class="keyword">if</span> result == worker.RESULT_SUCCESS:</span>
<span class="line"> log.debug(<span class="string">'synced item %r successfully'</span>, item)</span>
<span class="line"> self.complete_item(shard_num, retries)</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> log.error(<span class="string">'error syncing shard %d'</span>, shard_num)</span>
<span class="line"> retries.append(shard_num)</span>
<span class="line"> log.info(<span class="string">'%d/%d items processed'</span>, i + <span class="number">1</span>, num_items)</span>
<span class="line"> <span class="keyword">if</span> retries:</span>
<span class="line"> log.error(<span class="string">'Encountered errors syncing these %d shards: %r'</span>,</span>
<span class="line"> len(retries), retries)</span>
</pre></td></tr></table></figure>
     从 queue 中去的一个处理结果，
若失败，报错,
若成功，执行`self.complete_item(shard_num, retries)`，
radosgw-agent/sync.py:class Syncer(object)

        <span class="function"><span class="keyword">def</span> <span class="title">complete_item</span><span class="params">(self, shard_num, retries)</span>:</span> <span class="string">"""Called when syncing a single item completes successfully"""</span> marker = self.shard_info.get(shard_num) <span class="comment"># 获取该 shard_num 的 marker，若没有 shard_num，则返回空</span> <span class="keyword">if</span> <span class="keyword">not</span> marker: <span class="keyword">return</span> <span class="keyword">try</span>: <span class="comment"># post 该 shard 的 bound</span> data = [dict(name=retry, time=worker.DEFAULT_TIME) <span class="keyword">for</span> retry <span class="keyword">in</span> retries] client.set_worker_bound(self.dest_conn, self.type, marker, worker.DEFAULT_TIME, self.daemon_id, shard_num, data) <span class="keyword">except</span> Exception: log.warn(<span class="string">'could not set worker bounds, may repeat some work.'</span> <span class="string">'Traceback:'</span>, exc_info=<span class="keyword">True</span>) 

## [](https://ly798.github.io/2016/01/25/Radosgw-agenti-%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90/#u603B_u7ED3 "总结")总结

*   在 sync 中 2\. 同步 中`return new_retries`，同步失败的将返回，若没有失败的则返回空；
当获取处理结果的时候，从 queue 中获得该 retries，根据 shard_num 来 post admin/replica_log。
