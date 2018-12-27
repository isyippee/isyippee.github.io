---
title: Python ConfigParser模块和Openstack olso.cfg模块
tags: [python]
date: 2016-06-25 10:32:42
---

## [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#u4ECB_u7ECD "介绍")介绍

### [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#ConfigParser "ConfigParser")ConfigParser

ConfigParser实现了一种可用来解析ini配置文件配置项的语言。关于ini配置文件的有一些规则，

1.  需要使用`[section]`来标识
2.  配置使用kv形式，两种方式`k = v`或者`k : v`
3.  使用`#`和`;`来进行注释 <!-- more --> 

结构类似如下：
<figure class="highlight"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
</pre></td><td class="code"><pre><span class="line">[default]</span>
<span class="line"># comments</span>
<span class="line">; comments</span>
<span class="line">name = value</span>
<span class="line">name : value</span>
</pre></td></tr></table></figure>

#### [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#ConfigParser_u6A21_u5757_u5E38_u7528_u65B9_u6CD5 "ConfigParser模块常用方法")ConfigParser模块常用方法

##### [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#u8BFB_u53D6_u64CD_u4F5C "读取操作")读取操作

1.  read(file)
2.  sections()
3.  get(section, option)
4.  getint(section, option)
5.  getboolean(section, option)
6.  has_section(section)
7.  has_option(section, option) 

##### [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#u5199_u5165_u64CD_u4F5C "写入操作")写入操作

1.  add_section(section)
2.  set(section, option, value)
3.  remove_option(section, option)
4.  remove_section(section)
5.  write(file) 

### [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#oslo-config "oslo.config")oslo.config

oslo是用来为openstak各个组建提供一个common库，其中也就包括了olso.config，用来解析命令行参数和配置文件。

*   解析自定义的参数,在openstack中，一般都提供默认值
*   解析一些公共的参数，如`--debug`、`--verbose`
*   olso.config使用的配置文件是服务启动命令带有的参数`--config-file`，可指定多个。若未指定，会到默认的几个路径搜索。 

使用步骤是：
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
</pre></td><td class="code"><pre><span class="line"><span class="keyword">from</span> oslo_config <span class="keyword">import</span> cfg</span>
<span class="line"></span>
<span class="line">backup_manager_opts = [</span>
<span class="line"> cfg.StrOpt(<span class="string">'backup_driver'</span>,</span>
<span class="line"> default=<span class="string">'cinder.backup.drivers.swift'</span>,</span>
<span class="line"> help=<span class="string">'Driver to use for backups.'</span>,</span>
<span class="line"> deprecated_name=<span class="string">'backup_service'</span>),</span>
<span class="line">]</span>
<span class="line"></span>
<span class="line">CONF = cfg.CONF</span>
<span class="line">CONF.register_opts(backup_manager_opts)</span>
<span class="line"></span>
<span class="line"><span class="keyword">print</span> CONF.backup_manager_opts</span>
</pre></td></tr></table></figure>

相比ConfigParser，oslo.config强大的多，但在自己的小程序中需要自定义配置时，使用ConfigParser足够了。

## [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#use "use")use

使用ConfigParser来为一般的daemon程序来写一个简单的cfg模块
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
</pre></td><td class="code"><pre><span class="line"><span class="string">"""</span>
<span class="line">config.py</span>
<span class="line">for example:</span>
<span class="line"> import config as cfg</span>
<span class="line"></span>
<span class="line"> CONF = cfg.CONF</span>
<span class="line"></span>
<span class="line"> CONF.get('default', 'test')</span>
<span class="line">"""</span></span>
<span class="line"></span>
<span class="line"><span class="keyword">import</span> ConfigParser</span>
<span class="line"></span>
<span class="line">cfg_file = <span class="keyword">None</span></span>
<span class="line"></span>
<span class="line"><span class="class"><span class="keyword">class</span> <span class="title">ConfigOpts</span><span class="params">(ConfigParser.ConfigParser)</span>:</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__init__</span><span class="params">(self)</span>:</span></span>
<span class="line"> ConfigParser.ConfigParser.__init__(self)</span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">__call__</span><span class="params">(self, *args, **kwargs)</span>:</span></span>
<span class="line"> self.read(args[<span class="number">0</span>])</span>
<span class="line"></span>
<span class="line">CONF = ConfigOpts()</span>
</pre></td></tr></table></figure> 

在程序的main函数中，获取args调用cfg的**call**，通过该方法来制定配置文件
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
</pre></td><td class="code"><pre><span class="line"><span class="string">"""</span>
<span class="line">main.py</span>
<span class="line">"""</span></span>
<span class="line"></span>
<span class="line"><span class="keyword">import</span> sys</span>
<span class="line"><span class="keyword">import</span> config <span class="keyword">as</span> cfg</span>
<span class="line"></span>
<span class="line">CONF = cfg.CONF</span>
<span class="line"></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">main</span><span class="params">()</span>:</span></span>
<span class="line"> CONF(sys.argv[<span class="number">1</span>:])</span>
<span class="line"> </span>
<span class="line"><span class="keyword">if</span> __name__ == <span class="string">'__main__'</span>:</span>
<span class="line"> main()</span>
</pre></td></tr></table></figure>

这样，运行程序的时候，如`python main.py /etc/test.conf`，就能读取到该配置文件的配置项，使用`CONG.get(section, option)`就可以取的配置项。

## [](https://ly798.github.io/2016/06/25/Python-ConfigParser-%E6%A8%A1%E5%9D%97%E5%92%8C-Openstack-olso.cfg-%E6%A8%A1%E5%9D%97/#u53C2_u8003 "参考")参考

`https://docs.python.org/2/library/configparser.html`
`https://wiki.openstack.org/wiki/Oslo/Config`
