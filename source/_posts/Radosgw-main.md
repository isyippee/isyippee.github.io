---
title: Radosgw main()
tags: [ceph]
date: 2016-01-06 10:38:42
---

## [](https://ly798.github.io/2016/01/06/Radosgw-main/#u4ECB_u7ECD "介绍")介绍

main()作为入口
 <!-- more --> 

## [](https://ly798.github.io/2016/01/06/Radosgw-main/#u5206_u6790 "分析")分析
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
<span class="line">72</span>
<span class="line">73</span>
<span class="line">74</span>
<span class="line">75</span>
<span class="line">76</span>
<span class="line">77</span>
<span class="line">78</span>
<span class="line">79</span>
<span class="line">80</span>
<span class="line">81</span>
<span class="line">82</span>
<span class="line">83</span>
<span class="line">84</span>
<span class="line">85</span>
<span class="line">86</span>
<span class="line">87</span>
<span class="line">88</span>
<span class="line">89</span>
<span class="line">90</span>
<span class="line">91</span>
<span class="line">92</span>
<span class="line">93</span>
<span class="line">94</span>
<span class="line">95</span>
<span class="line">96</span>
<span class="line">97</span>
<span class="line">98</span>
<span class="line">99</span>
<span class="line">100</span>
<span class="line">101</span>
<span class="line">102</span>
<span class="line">103</span>
<span class="line">104</span>
<span class="line">105</span>
<span class="line">106</span>
<span class="line">107</span>
<span class="line">108</span>
<span class="line">109</span>
<span class="line">110</span>
<span class="line">111</span>
<span class="line">112</span>
<span class="line">113</span>
<span class="line">114</span>
<span class="line">115</span>
<span class="line">116</span>
<span class="line">117</span>
<span class="line">118</span>
<span class="line">119</span>
<span class="line">120</span>
<span class="line">121</span>
<span class="line">122</span>
<span class="line">123</span>
<span class="line">124</span>
<span class="line">125</span>
<span class="line">126</span>
<span class="line">127</span>
<span class="line">128</span>
<span class="line">129</span>
<span class="line">130</span>
<span class="line">131</span>
<span class="line">132</span>
<span class="line">133</span>
<span class="line">134</span>
<span class="line">135</span>
<span class="line">136</span>
<span class="line">137</span>
<span class="line">138</span>
<span class="line">139</span>
<span class="line">140</span>
<span class="line">141</span>
<span class="line">142</span>
<span class="line">143</span>
<span class="line">144</span>
<span class="line">145</span>
<span class="line">146</span>
<span class="line">147</span>
<span class="line">148</span>
<span class="line">149</span>
<span class="line">150</span>
<span class="line">151</span>
<span class="line">152</span>
<span class="line">153</span>
<span class="line">154</span>
<span class="line">155</span>
<span class="line">156</span>
<span class="line">157</span>
<span class="line">158</span>
<span class="line">159</span>
<span class="line">160</span>
<span class="line">161</span>
<span class="line">162</span>
<span class="line">163</span>
<span class="line">164</span>
<span class="line">165</span>
<span class="line">166</span>
<span class="line">167</span>
<span class="line">168</span>
<span class="line">169</span>
<span class="line">170</span>
<span class="line">171</span>
<span class="line">172</span>
<span class="line">173</span>
<span class="line">174</span>
<span class="line">175</span>
<span class="line">176</span>
<span class="line">177</span>
<span class="line">178</span>
<span class="line">179</span>
<span class="line">180</span>
<span class="line">181</span>
<span class="line">182</span>
<span class="line">183</span>
<span class="line">184</span>
<span class="line">185</span>
<span class="line">186</span>
<span class="line">187</span>
<span class="line">188</span>
<span class="line">189</span>
<span class="line">190</span>
<span class="line">191</span>
<span class="line">192</span>
<span class="line">193</span>
<span class="line">194</span>
<span class="line">195</span>
<span class="line">196</span>
<span class="line">197</span>
<span class="line">198</span>
<span class="line">199</span>
<span class="line">200</span>
<span class="line">201</span>
<span class="line">202</span>
<span class="line">203</span>
<span class="line">204</span>
<span class="line">205</span>
<span class="line">206</span>
<span class="line">207</span>
<span class="line">208</span>
<span class="line">209</span>
<span class="line">210</span>
<span class="line">211</span>
<span class="line">212</span>
<span class="line">213</span>
<span class="line">214</span>
<span class="line">215</span>
<span class="line">216</span>
<span class="line">217</span>
<span class="line">218</span>
<span class="line">219</span>
<span class="line">220</span>
<span class="line">221</span>
<span class="line">222</span>
<span class="line">223</span>
<span class="line">224</span>
<span class="line">225</span>
<span class="line">226</span>
<span class="line">227</span>
<span class="line">228</span>
<span class="line">229</span>
<span class="line">230</span>
<span class="line">231</span>
<span class="line">232</span>
<span class="line">233</span>
<span class="line">234</span>
<span class="line">235</span>
<span class="line">236</span>
<span class="line">237</span>
<span class="line">238</span>
<span class="line">239</span>
<span class="line">240</span>
<span class="line">241</span>
<span class="line">242</span>
<span class="line">243</span>
<span class="line">244</span>
<span class="line">245</span>
<span class="line">246</span>
<span class="line">247</span>
<span class="line">248</span>
<span class="line">249</span>
<span class="line">250</span>
<span class="line">251</span>
<span class="line">252</span>
<span class="line">253</span>
<span class="line">254</span>
<span class="line">255</span>
<span class="line">256</span>
<span class="line">257</span>
<span class="line">258</span>
<span class="line">259</span>
<span class="line">260</span>
<span class="line">261</span>
</pre></td><td class="code"><pre><span class="line"><span class="comment">/*</span>
<span class="line"> * start up the RADOS connection and then handle HTTP messages as they come in</span>
<span class="line"> */</span></span>
<span class="line"><span class="function"><span class="keyword">int</span> <span class="title">main</span><span class="params">(<span class="keyword">int</span> argc, <span class="keyword">const</span> <span class="keyword">char</span> **argv)</span></span>
<span class="line"></span>&#123;</span>
<span class="line"> <span class="comment">// dout() messages will be sent to stderr, but FCGX wants messages on stdout</span></span>
<span class="line"> <span class="comment">// Redirect stderr to stdout.</span></span>
<span class="line"> TEMP_FAILURE_RETRY(close(STDERR_FILENO));</span>
<span class="line"> <span class="keyword">if</span> (TEMP_FAILURE_RETRY(dup2(STDOUT_FILENO, STDERR_FILENO) &lt; <span class="number">0</span>)) &#123;</span>
<span class="line"> <span class="keyword">int</span> err = errno;</span>
<span class="line"> <span class="built_in">cout</span> &lt;&lt; <span class="string">"failed to redirect stderr to stdout: "</span> &lt;&lt; cpp_strerror(err)</span>
<span class="line"> &lt;&lt; <span class="built_in">std</span>::endl;</span>
<span class="line"> <span class="keyword">return</span> ENOSYS;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="comment">//设置默认参数</span></span>
<span class="line"> <span class="comment">/* alternative default for module */</span></span>
<span class="line"> <span class="built_in">vector</span>&lt;<span class="keyword">const</span> <span class="keyword">char</span> *&gt; def_args;</span>
<span class="line"> def_args.push_back(<span class="string">"--debug-rgw=1/5"</span>);</span>
<span class="line"> def_args.push_back(<span class="string">"--keyring=$rgw_data/keyring"</span>);</span>
<span class="line"> def_args.push_back(<span class="string">"--log-file=/var/log/radosgw/$cluster-$name.log"</span>);</span>
<span class="line"></span>
<span class="line"> <span class="built_in">vector</span>&lt;<span class="keyword">const</span> <span class="keyword">char</span>*&gt; args;</span>
<span class="line"> argv_to_vec(argc, argv, args); <span class="comment">//循环将 argv 中的参数加到 args 中</span></span>
<span class="line"> env_to_vec(args); <span class="comment">//获取环境变量 CEPH_ARGS ，并加到 args 中</span></span>
<span class="line"> global_init(&amp;def_args, args, CEPH_ENTITY_TYPE_CLIENT, CODE_ENVIRONMENT_DAEMON,</span>
<span class="line"> CINIT_FLAG_UNPRIVILEGED_DAEMON_DEFAULTS); <span class="comment">// 创建目录/var/run/ceph 等初始化操作</span></span>
<span class="line"></span>
<span class="line"> <span class="comment">//判断 args 中是否有 -h 或 --help</span></span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">std</span>::<span class="built_in">vector</span>&lt;<span class="keyword">const</span> <span class="keyword">char</span>*&gt;::iterator i = args.begin(); i != args.end(); ++i) &#123;</span>
<span class="line"> <span class="keyword">if</span> (ceph_argparse_flag(args, i, <span class="string">"-h"</span>, <span class="string">"--help"</span>, (<span class="keyword">char</span>*)<span class="literal">NULL</span>)) &#123;</span>
<span class="line"> usage();</span>
<span class="line"> <span class="keyword">return</span> <span class="number">0</span>;</span>
<span class="line"> &#125;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> check_curl(); <span class="comment">//无操作</span></span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (g_conf-&gt;daemonize) &#123;</span>
<span class="line"> global_init_daemonize(g_ceph_context, <span class="number">0</span>);</span>
<span class="line"> &#125;</span>
<span class="line"> <span class="function">Mutex <span class="title">mutex</span><span class="params">(<span class="string">"main"</span>)</span></span>;</span>
<span class="line"> <span class="function">SafeTimer <span class="title">init_timer</span><span class="params">(g_ceph_context, mutex)</span></span>;</span>
<span class="line"> init_timer.init();</span>
<span class="line"> mutex.Lock();</span>
<span class="line"> init_timer.add_event_after(g_conf-&gt;rgw_init_timeout, <span class="keyword">new</span> C_InitTimeout);</span>
<span class="line"> mutex.Unlock();</span>
<span class="line"></span>
<span class="line"> common_init_finish(g_ceph_context); <span class="comment">//start service thread</span></span>
<span class="line"></span>
<span class="line"> rgw_tools_init(g_ceph_context); <span class="comment">//初始化mine map，读取文件/etc/mime.types</span></span>
<span class="line"></span>
<span class="line"> rgw_init_resolver(); <span class="comment">// new RGWResolver()</span></span>
<span class="line"> rgw_rest_init(g_ceph_context); <span class="comment">// 设置http header map、http code map</span></span>
<span class="line"></span>
<span class="line"> curl_global_init(CURL_GLOBAL_ALL); <span class="comment">// null</span></span>
<span class="line"> </span>
<span class="line"> FCGX_Init(); <span class="comment">// null</span></span>
<span class="line"></span>
<span class="line"> <span class="comment">// 获得实例 store，获得实例meta_mgr、data_log，</span></span>
<span class="line"> <span class="comment">// init region、zone，初始化并启动 gc ，获得实例 qutoa_hander</span></span>
<span class="line"> <span class="keyword">int</span> r = <span class="number">0</span>;</span>
<span class="line"> RGWRados *store = RGWStoreManager::get_storage(g_ceph_context, <span class="literal">true</span>, <span class="literal">true</span>);</span>
<span class="line"> <span class="keyword">if</span> (!store) &#123;</span>
<span class="line"> derr &lt;&lt; <span class="string">"Couldn't init storage provider (RADOS)"</span> &lt;&lt; dendl;</span>
<span class="line"> r = EIO;</span>
<span class="line"> &#125;</span>
<span class="line"> <span class="keyword">if</span> (!r)</span>
<span class="line"> r = rgw_perf_start(g_ceph_context);</span>
<span class="line"></span>
<span class="line"> mutex.Lock();</span>
<span class="line"> init_timer.cancel_all_events();</span>
<span class="line"> init_timer.shutdown();</span>
<span class="line"> mutex.Unlock();</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (r) </span>
<span class="line"> <span class="keyword">return</span> <span class="number">1</span>;</span>
<span class="line"></span>
<span class="line"> rgw_user_init(store-&gt;meta_mgr); <span class="comment">// 注册 user_meta_handler</span></span>
<span class="line"> rgw_bucket_init(store-&gt;meta_mgr); <span class="comment">//注册 bucket_meta_handler、bucket_instance_meta_handler</span></span>
<span class="line"> rgw_log_usage_init(g_ceph_context, store);</span>
<span class="line"></span>
<span class="line"> RGWREST rest;</span>
<span class="line"></span>
<span class="line"> <span class="built_in">list</span>&lt;<span class="built_in">string</span>&gt; apis;</span>
<span class="line"> <span class="keyword">bool</span> do_swift = <span class="literal">false</span>;</span>
<span class="line"></span>
<span class="line"> get_str_list(g_conf-&gt;rgw_enable_apis, apis); <span class="comment">// 获得apis，默认 s3, swift, swift_auth, admin</span></span>
<span class="line"></span>
<span class="line"> <span class="built_in">map</span>&lt;<span class="built_in">string</span>, <span class="keyword">bool</span>&gt; apis_map;</span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">list</span>&lt;<span class="built_in">string</span>&gt;::iterator li = apis.begin(); li != apis.end(); ++li) &#123;</span>
<span class="line"> apis_map[*li] = <span class="literal">true</span>;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (apis_map.count(<span class="string">"s3"</span>) &gt; <span class="number">0</span>)</span>
<span class="line"> rest.register_default_mgr(set_logging(<span class="keyword">new</span> RGWRESTMgr_S3));</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (apis_map.count(<span class="string">"swift"</span>) &gt; <span class="number">0</span>) &#123;</span>
<span class="line"> do_swift = <span class="literal">true</span>;</span>
<span class="line"> swift_init(g_ceph_context);</span>
<span class="line"> rest.register_resource(g_conf-&gt;rgw_swift_url_prefix, set_logging(<span class="keyword">new</span> RGWRESTMgr_SWIFT));</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (apis_map.count(<span class="string">"swift_auth"</span>) &gt; <span class="number">0</span>)</span>
<span class="line"> rest.register_resource(g_conf-&gt;rgw_swift_auth_entry, set_logging(<span class="keyword">new</span> RGWRESTMgr_SWIFT_Auth));</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (apis_map.count(<span class="string">"admin"</span>) &gt; <span class="number">0</span>) &#123;</span>
<span class="line"> RGWRESTMgr_Admin *admin_resource = <span class="keyword">new</span> RGWRESTMgr_Admin;</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"usage"</span>, <span class="keyword">new</span> RGWRESTMgr_Usage);</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"user"</span>, <span class="keyword">new</span> RGWRESTMgr_User);</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"bucket"</span>, <span class="keyword">new</span> RGWRESTMgr_Bucket);</span>
<span class="line"> </span>
<span class="line"> <span class="comment">/*Registering resource for /admin/metadata */</span></span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"metadata"</span>, <span class="keyword">new</span> RGWRESTMgr_Metadata);</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"log"</span>, <span class="keyword">new</span> RGWRESTMgr_Log);</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"opstate"</span>, <span class="keyword">new</span> RGWRESTMgr_Opstate);</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"replica_log"</span>, <span class="keyword">new</span> RGWRESTMgr_ReplicaLog);</span>
<span class="line"> admin_resource-&gt;register_resource(<span class="string">"config"</span>, <span class="keyword">new</span> RGWRESTMgr_Config);</span>
<span class="line"> rest.register_resource(g_conf-&gt;rgw_admin_entry, admin_resource);</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="comment">//日志文件</span></span>
<span class="line"> OpsLogSocket *olog = <span class="literal">NULL</span>;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (!g_conf-&gt;rgw_ops_log_socket_path.empty()) &#123;</span>
<span class="line"> olog = <span class="keyword">new</span> OpsLogSocket(g_ceph_context, g_conf-&gt;rgw_ops_log_data_backlog);</span>
<span class="line"> olog-&gt;init(g_conf-&gt;rgw_ops_log_socket_path);</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> r = signal_fd_init();</span>
<span class="line"> <span class="keyword">if</span> (r &lt; <span class="number">0</span>) &#123;</span>
<span class="line"> derr &lt;&lt; <span class="string">"ERROR: unable to initialize signal fds"</span> &lt;&lt; dendl;</span>
<span class="line"> <span class="built_in">exit</span>(<span class="number">1</span>);</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> init_async_signal_handler();</span>
<span class="line"> register_async_signal_handler(SIGHUP, sighup_handler);</span>
<span class="line"> register_async_signal_handler(SIGTERM, handle_sigterm);</span>
<span class="line"> register_async_signal_handler(SIGINT, handle_sigterm);</span>
<span class="line"> register_async_signal_handler(SIGUSR1, handle_sigterm);</span>
<span class="line"> sighandler_alrm = signal(SIGALRM, godown_alarm);</span>
<span class="line"></span>
<span class="line"> <span class="built_in">string</span> frontend_frameworks = g_conf-&gt;rgw_frontends;</span>
<span class="line"></span>
<span class="line"> <span class="built_in">list</span>&lt;<span class="built_in">string</span>&gt; frontends;</span>
<span class="line"></span>
<span class="line"> get_str_list(g_conf-&gt;rgw_frontends, <span class="string">","</span>, frontends);</span>
<span class="line"></span>
<span class="line"> <span class="built_in">multimap</span>&lt;<span class="built_in">string</span>, RGWFrontendConfig *&gt; fe_map;</span>
<span class="line"> <span class="built_in">list</span>&lt;RGWFrontendConfig *&gt; configs;</span>
<span class="line"> <span class="keyword">if</span> (frontends.empty()) &#123;</span>
<span class="line"> frontends.push_back(<span class="string">"fastcgi"</span>);</span>
<span class="line"> &#125;</span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">list</span>&lt;<span class="built_in">string</span>&gt;::iterator iter = frontends.begin(); iter != frontends.end(); ++iter) &#123;</span>
<span class="line"> <span class="built_in">string</span>&amp; f = *iter;</span>
<span class="line"></span>
<span class="line"> RGWFrontendConfig *config = <span class="keyword">new</span> RGWFrontendConfig(f);</span>
<span class="line"> <span class="keyword">int</span> r = config-&gt;init();</span>
<span class="line"> <span class="keyword">if</span> (r &lt; <span class="number">0</span>) &#123;</span>
<span class="line"> <span class="built_in">cerr</span> &lt;&lt; <span class="string">"ERROR: failed to init config: "</span> &lt;&lt; f &lt;&lt; <span class="built_in">std</span>::endl;</span>
<span class="line"> <span class="keyword">return</span> EINVAL;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> configs.push_back(config);</span>
<span class="line"></span>
<span class="line"> <span class="built_in">string</span> framework = config-&gt;get_framework();</span>
<span class="line"> fe_map.insert(make_pair&lt;<span class="built_in">string</span>, RGWFrontendConfig *&gt;(framework, config));</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="built_in">list</span>&lt;RGWFrontend *&gt; fes;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">multimap</span>&lt;<span class="built_in">string</span>, RGWFrontendConfig *&gt;::iterator fiter = fe_map.begin(); fiter != fe_map.end(); ++fiter) &#123;</span>
<span class="line"> RGWFrontendConfig *config = fiter-&gt;second;</span>
<span class="line"> <span class="built_in">string</span> framework = config-&gt;get_framework();</span>
<span class="line"> RGWFrontend *fe;</span>
<span class="line"> <span class="keyword">if</span> (framework == <span class="string">"fastcgi"</span> || framework == <span class="string">"fcgi"</span>) &#123;</span>
<span class="line"> RGWProcessEnv fcgi_pe = &#123; store, &amp;rest, olog, <span class="number">0</span> &#125;;</span>
<span class="line"></span>
<span class="line"> fe = <span class="keyword">new</span> RGWFCGXFrontend(fcgi_pe, config);</span>
<span class="line"> &#125; <span class="keyword">else</span> <span class="keyword">if</span> (framework == <span class="string">"civetweb"</span> || framework == <span class="string">"mongoose"</span>) &#123;</span>
<span class="line"> <span class="built_in">string</span> err;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">int</span> port;</span>
<span class="line"> config-&gt;get_val(<span class="string">"port"</span>, <span class="number">80</span>, &amp;port);</span>
<span class="line"></span>
<span class="line"> RGWProcessEnv env = &#123; store, &amp;rest, olog, port &#125;;</span>
<span class="line"></span>
<span class="line"> fe = <span class="keyword">new</span> RGWMongooseFrontend(env, config);</span>
<span class="line"> &#125; <span class="keyword">else</span> <span class="keyword">if</span> (framework == <span class="string">"loadgen"</span>) &#123;</span>
<span class="line"> <span class="keyword">int</span> port;</span>
<span class="line"> config-&gt;get_val(<span class="string">"port"</span>, <span class="number">80</span>, &amp;port);</span>
<span class="line"></span>
<span class="line"> RGWProcessEnv env = &#123; store, &amp;rest, olog, port &#125;;</span>
<span class="line"></span>
<span class="line"> fe = <span class="keyword">new</span> RGWLoadGenFrontend(env, config);</span>
<span class="line"> &#125; <span class="keyword">else</span> &#123;</span>
<span class="line"> dout(<span class="number">0</span>) &lt;&lt; <span class="string">"WARNING: skipping unknown framework: "</span> &lt;&lt; framework &lt;&lt; dendl;</span>
<span class="line"> <span class="keyword">continue</span>;</span>
<span class="line"> &#125;</span>
<span class="line"> dout(<span class="number">0</span>) &lt;&lt; <span class="string">"starting handler: "</span> &lt;&lt; fiter-&gt;first &lt;&lt; dendl;</span>
<span class="line"> <span class="keyword">int</span> r = fe-&gt;init();</span>
<span class="line"> <span class="keyword">if</span> (r &lt; <span class="number">0</span>) &#123;</span>
<span class="line"> derr &lt;&lt; <span class="string">"ERROR: failed initializing frontend"</span> &lt;&lt; dendl;</span>
<span class="line"> <span class="keyword">return</span> -r;</span>
<span class="line"> &#125;</span>
<span class="line"> fe-&gt;run();</span>
<span class="line"></span>
<span class="line"> fes.push_back(fe);</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> wait_shutdown();</span>
<span class="line"></span>
<span class="line"> derr &lt;&lt; <span class="string">"shutting down"</span> &lt;&lt; dendl;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">list</span>&lt;RGWFrontend *&gt;::iterator liter = fes.begin(); liter != fes.end(); ++liter) &#123;</span>
<span class="line"> RGWFrontend *fe = *liter;</span>
<span class="line"> fe-&gt;stop();</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">list</span>&lt;RGWFrontend *&gt;::iterator liter = fes.begin(); liter != fes.end(); ++liter) &#123;</span>
<span class="line"> RGWFrontend *fe = *liter;</span>
<span class="line"> fe-&gt;join();</span>
<span class="line"> <span class="keyword">delete</span> fe;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> <span class="keyword">for</span> (<span class="built_in">list</span>&lt;RGWFrontendConfig *&gt;::iterator liter = configs.begin(); liter != configs.end(); ++liter) &#123;</span>
<span class="line"> RGWFrontendConfig *fec = *liter;</span>
<span class="line"> <span class="keyword">delete</span> fec;</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> unregister_async_signal_handler(SIGHUP, sighup_handler);</span>
<span class="line"> unregister_async_signal_handler(SIGTERM, handle_sigterm);</span>
<span class="line"> unregister_async_signal_handler(SIGINT, handle_sigterm);</span>
<span class="line"> unregister_async_signal_handler(SIGUSR1, handle_sigterm);</span>
<span class="line"> shutdown_async_signal_handler();</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> (do_swift) &#123;</span>
<span class="line"> swift_finalize();</span>
<span class="line"> &#125;</span>
<span class="line"></span>
<span class="line"> rgw_log_usage_finalize();</span>
<span class="line"></span>
<span class="line"> <span class="keyword">delete</span> olog;</span>
<span class="line"></span>
<span class="line"> rgw_perf_stop(g_ceph_context);</span>
<span class="line"></span>
<span class="line"> RGWStoreManager::close_storage(store);</span>
<span class="line"></span>
<span class="line"> rgw_tools_cleanup();</span>
<span class="line"> rgw_shutdown_resolver();</span>
<span class="line"> curl_global_cleanup();</span>
<span class="line"></span>
<span class="line"> dout(<span class="number">1</span>) &lt;&lt; <span class="string">"final shutdown"</span> &lt;&lt; dendl;</span>
<span class="line"> g_ceph_context-&gt;put();</span>
<span class="line"></span>
<span class="line"> ceph::crypto::shutdown();</span>
<span class="line"></span>
<span class="line"> signal_fd_finalize();</span>
<span class="line"></span>
<span class="line"> <span class="keyword">return</span> <span class="number">0</span>;</span>
<span class="line">&#125;</span>
</pre></td></tr></table></figure>
