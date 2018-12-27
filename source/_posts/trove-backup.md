---
title: trove backup
tags: [openstack trove]
date: 2016-12-19 10:19:27
---

## [](https://ly798.github.io/2016/12/19/trove-backup/#introduction "introduction")introduction

Code Reading
 <!-- more --> 

## [](https://ly798.github.io/2016/12/19/trove-backup/#backup "backup")backup

api:create_backup —-&gt; taskmanager:create_backup —-&gt; self.guest.create_backup —-&gt; rabbitmq —-&gt; guestagent:mysql_manager:create_backup —-&gt; innobackupex —-&gt; swift put

### [](https://ly798.github.io/2016/12/19/trove-backup/#taskmanager__u4E2D_u7684_create_backup "taskmanager 中的 create_backup")taskmanager 中的 create_backup

trove/taskmanager/manager.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Manager</span><span class="params">(periodic_task.PeriodicTasks)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">create_backup</span><span class="params">(self, context, backup_info, instance_id)</span>:</span></span>
<span class="line"> instance_tasks = models.BuiltInstanceTasks.load(context, instance_id)</span>
<span class="line"> instance_tasks.create_backup(backup_info)</span>
</pre></td></tr></table></figure> 

trove/taskmanager/models.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">BuiltInstanceTasks</span><span class="params">(BuiltInstance, NotifyMixin, ConfigurationMixin)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">create_backup</span><span class="params">(self, backup_info)</span>:</span></span>
<span class="line"> LOG.info(_(<span class="string">"Initiating backup for instance %s."</span>) % self.id)</span>
<span class="line"> self.guest.create_backup(backup_info)</span>
</pre></td></tr></table></figure> 

这里的 self.guest trove/instance/models.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">BaseInstance</span><span class="params">(SimpleInstance)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">get_guest</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> create_guest_client(self.context, self.db_info.id)</span>
</pre></td></tr></table></figure> 

trove/common/remote.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line">create_guest_client = import_class(CONF.remote_guest_client)</span>
<span class="line"></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">guest_client</span><span class="params">(context, id, manager=None)</span>:</span></span>
<span class="line"> <span class="keyword">from</span> trove.guestagent.api <span class="keyword">import</span> API</span>
<span class="line"> <span class="keyword">if</span> manager:</span>
<span class="line"> clazz = strategy.load_guestagent_strategy(manager).guest_client_class</span>
<span class="line"> <span class="keyword">else</span>:</span>
<span class="line"> clazz = API</span>
<span class="line"> <span class="keyword">return</span> clazz(context, id)</span>
</pre></td></tr></table></figure> 

所以 guest 对应的是 API 实例, cast()发送消息。guestagent 接受到消息

### [](https://ly798.github.io/2016/12/19/trove-backup/#guestagent__u4E2D_u7684_createbackup "guestagent 中的 createbackup")guestagent 中的 create<sub>backup</sub>

trove/guestagent/datastore/mysql/manager.py
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">Manager</span><span class="params">(periodic_task.PeriodicTasks)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">create_backup</span><span class="params">(self, context, backup_info)</span>:</span></span>
<span class="line"> backup.backup(context, backup_info)</span>
</pre></td></tr></table></figure> 

trove/guestagent/backup/backupagent.py
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
<span class="line">70</span>
<span class="line">71</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment"># innobackupex 或 mysqldump</span></span>
<span class="line">RUNNER = get_backup_strategy(STRATEGY, BACKUP_NAMESPACE)</span>
<span class="line"><span class="comment"># 额外的参数，如使用 innobackupex，可以有--user os_admin --socket /var/lib/mysql/mysql.sock</span></span>
<span class="line">EXTRA_OPTS = CONF.backup_runner_options.get(STRATEGY, <span class="string">''</span>)</span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">BackupAgent</span><span class="params">(object)</span>:</span></span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">stream_backup_to_storage</span><span class="params">(self, backup_info, runner, storage,</span>
<span class="line"> parent_metadata=&#123;&#125;, extra_opts=EXTRA_OPTS)</span>:</span></span>
<span class="line"> backup_id = backup_info[<span class="string">'id'</span>]</span>
<span class="line"> ctxt = trove_context.TroveContext(</span>
<span class="line"> user=CONF.nova_proxy_admin_user,</span>
<span class="line"> auth_token=CONF.nova_proxy_admin_pass)</span>
<span class="line"> conductor = conductor_api.API(ctxt)</span>
<span class="line"></span>
<span class="line"> <span class="comment"># Store the size of the filesystem before the backup.</span></span>
<span class="line"> mount_point = CONFIG_MANAGER.mount_point</span>
<span class="line"> stats = get_filesystem_volume_stats(mount_point)</span>
<span class="line"> backup_state = &#123;</span>
<span class="line"> <span class="string">'backup_id'</span>: backup_id,</span>
<span class="line"> <span class="string">'size'</span>: stats.get(<span class="string">'used'</span>, <span class="number">0.0</span>),</span>
<span class="line"> <span class="string">'state'</span>: BackupState.BUILDING,</span>
<span class="line"> &#125;</span>
<span class="line"> conductor.update_backup(CONF.guest_id,</span>
<span class="line"> sent=timeutils.float_utcnow(),</span>
<span class="line"> **backup_state)</span>
<span class="line"> LOG.debug(<span class="string">"Updated state for %s to %s."</span>, backup_id, backup_state)</span>
<span class="line"></span>
<span class="line"> <span class="keyword">with</span> runner(filename=backup_id, extra_opts=extra_opts,</span>
<span class="line"> **parent_metadata) <span class="keyword">as</span> bkup:</span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> LOG.debug(<span class="string">"Starting backup %s."</span>, backup_id)</span>
<span class="line"> success, note, checksum, location = storage.save(</span>
<span class="line"> bkup.manifest,</span>
<span class="line"> bkup)</span>
<span class="line"></span>
<span class="line"> backup_state.update(&#123;</span>
<span class="line"> <span class="string">'checksum'</span>: checksum,</span>
<span class="line"> <span class="string">'location'</span>: location,</span>
<span class="line"> <span class="string">'note'</span>: note,</span>
<span class="line"> <span class="string">'success'</span>: success,</span>
<span class="line"> <span class="string">'backup_type'</span>: bkup.backup_type,</span>
<span class="line"> &#125;)</span>
<span class="line"> . . . </span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> success:</span>
<span class="line"> <span class="keyword">raise</span> BackupError(note)</span>
<span class="line"></span>
<span class="line"> meta = bkup.metadata()</span>
<span class="line"> meta[<span class="string">'datastore'</span>] = backup_info[<span class="string">'datastore'</span>]</span>
<span class="line"> meta[<span class="string">'datastore_version'</span>] = backup_info[</span>
<span class="line"> <span class="string">'datastore_version'</span>]</span>
<span class="line"> storage.save_metadata(location, meta)</span>
<span class="line"></span>
<span class="line"> backup_state.update(&#123;<span class="string">'state'</span>: BackupState.COMPLETED&#125;)</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> meta</span>
<span class="line"> . . .</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">execute_backup</span><span class="params">(self, context, backup_info,</span>
<span class="line"> runner=RUNNER, extra_opts=EXTRA_OPTS)</span>:</span></span>
<span class="line"></span>
<span class="line"> LOG.debug(<span class="string">"Running backup %(id)s."</span>, backup_info)</span>
<span class="line"> storage = get_storage_strategy(</span>
<span class="line"> CONF.storage_strategy,</span>
<span class="line"> CONF.storage_namespace)(context)</span>
<span class="line"></span>
<span class="line"> <span class="comment">#是否是 incremental backup</span></span>
<span class="line"> . . . </span>
<span class="line"> self.stream_backup_to_storage(backup_info, runner, storage,</span>
<span class="line"> parent_metadata, extra_opts)</span>
</pre></td></tr></table></figure> 

RUNNER 对应的是 `trove.guestagent.strategies.backup.mysql_impl:InnoBackupEx/InnoBackupExIncremental` , 使用 stream backup
 <figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">BackupRunner</span><span class="params">(Strategy)</span>:</span></span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__enter__</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="string">"""Start up the process."""</span></span>
<span class="line"> self._run_pre_backup()</span>
<span class="line"> self.run()</span>
<span class="line"> <span class="keyword">return</span> self</span>
</pre></td></tr></table></figure> 

storage 对应的是 `trove.guestagent.strategies.storage.swift:SwiftStorage` , 从 stream read 出来 使用 swift put 上传.

## [](https://ly798.github.io/2016/12/19/trove-backup/#radosgw_u2019s_swift_api "radosgw’s swift api")radosgw’s swift api

### [](https://ly798.github.io/2016/12/19/trove-backup/#rgw__u96C6_u6210_keystone__u90E8_u5206_u914D_u7F6E "rgw 集成 keystone 部分配置")rgw 集成 keystone 部分配置
<figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line">rgw keystone url = http://<span class="number">192.168</span>.<span class="number">6.125</span>:<span class="number">5000</span></span>
<span class="line"><span class="comment"># 若不配置该项，需要配置 rgw_keystone_admin_user rgw_keystone_admin_password rgw_keystone_admin_tenant 三项，去请求 url 得到 admin token (rgw_swift.cc:244)</span></span>
<span class="line">rgw keystone admin token = <span class="number">85</span>cc6b3914c042fe8be37032ff03fa49</span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/12/19/trove-backup/#rgw__u63A5_u53D7_u5230_swift__u8BF7_u6C42_u4E4B_u540E_u5904_u7406 "rgw 接受到 swift 请求之后处理")rgw 接受到 swift 请求之后处理
<figure class="highlight c"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">ret = handler-&gt;authorize();</span>
</pre></td></tr></table></figure> 

请求携带 token，对 token 进行验证
 <figure class="highlight c"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">bool</span> RGWSwift::verify_swift_token(RGWRados *store, req_state *s)</span>
</pre></td></tr></table></figure> 

validate<sub>token</sub> 返回详细信息和其 tenant、user、role 等信息, 对应请求的 keystone API `/v2.0/tokens/{tokenId}`
 <figure class="highlight c"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">validate_keystone_token(store, s-&gt;os_auth_token, &amp;info, s-&gt;user);</span>
</pre></td></tr></table></figure> <figure class="highlight c"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">parse_keystone_token_response(token, bl, info, t)</span>
</pre></td></tr></table></figure> 

parse 之后的 info
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">&#123;status = <span class="number">200</span>, auth_groups = <span class="string">""</span>, user = <span class="string">"a400d920a63642d4913e8f2f3afda408"</span>, display_name = <span class="string">"demo"</span>, ttl = <span class="number">0</span>&#125;</span>
</pre></td></tr></table></figure> <figure class="highlight c"><table><tr><td class="gutter"><pre><span class="line">1</span>
</pre></td><td class="code"><pre><span class="line">update_user_info(store, info, rgw_user);</span>
</pre></td></tr></table></figure> 

对应关系：
 <figure class="highlight bash"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
</pre></td><td class="code"><pre><span class="line">info user --&gt; keystone tenant_id --&gt; rgw uid</span>
<span class="line">info display_name --&gt; keystone tenant_name --&gt; rgw display_name</span>
</pre></td></tr></table></figure> 

那使用 rgw swift 来作 trove backup, 但是很遗憾。trove 配置项 `backup_swift_container = database_backups` , 创建 backup 时为 tenant 创建一个 container 用来存放 backup，但是 rgw 的 bucket_name 是全局的，swift 中 container<sub>name</sub> 是 tenant 下的。当我使用第二个 tenant 去创建 backups 时，会先创建`database<sub>backups</sub>`的 bucket，而这个 bucket 已被占用，`BuceketAlreadyExist`

或许可以使用 database_backups_$(tenant_id) 的 bucket 来存储，这种方式在 juno 版是可以(trove juno 使 swift DLO 方式上传)trove master 使用了 swift SLO 的方式，目前 rgw swift 不支持。so, openstack or implement s3 in trove.
