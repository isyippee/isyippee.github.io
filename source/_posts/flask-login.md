---
title: flask login
tags: [python]
date: 2016-09-22 10:22:51
---

## [](https://ly798.github.io/2016/09/22/flask-login/#u4ECB_u7ECD "介绍")介绍

使用flask-login很方便的来保存用户的登陆状态。
 <!-- more --> 

常用的有
<figure class="highlight livescript"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
</pre></td><td class="code"><pre><span class="line"><span class="keyword">from</span> flask_login <span class="keyword">import</span> current_user, login_required, login_user, logout_user</span>
<span class="line"></span>
<span class="line"><span class="property">@login_manager</span>.user_loader</span>
</pre></td></tr></table></figure>

### [](https://ly798.github.io/2016/09/22/flask-login/#login_user_u548Clogout_user "login_user和logout_user")login_user和logout_user
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
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">login_user</span><span class="params">(user, remember=False, force=False, fresh=True)</span>:</span></span>
<span class="line"> <span class="keyword">if</span> <span class="keyword">not</span> force <span class="keyword">and</span> <span class="keyword">not</span> user.is_active:</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">False</span></span>
<span class="line"> <span class="comment"># current_app.login_manager.id_attribut 即 `get_id`，也就变成 user.get_id()，</span></span>
<span class="line"> <span class="comment"># 这也就是为什么user model 需要重写 get_id的原因，而不必强行规定写是因为继承的 </span></span>
<span class="line"> <span class="comment"># UserMixin 实现了 get_id() 方法</span></span>
<span class="line"> user_id = getattr(user, current_app.login_manager.id_attribute)()</span>
<span class="line"> session[<span class="string">'user_id'</span>] = user_id</span>
<span class="line"> session[<span class="string">'_fresh'</span>] = fresh</span>
<span class="line"> session[<span class="string">'_id'</span>] = _create_identifier()</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> remember:</span>
<span class="line"> session[<span class="string">'remember'</span>] = <span class="string">'set'</span></span>
<span class="line"> </span>
<span class="line"> <span class="comment"># 将该user放到请求上下文的栈上</span></span>
<span class="line"> _request_ctx_stack.top.user = user</span>
<span class="line"> <span class="comment"># 发送用户登陆的信号</span></span>
<span class="line"> user_logged_in.send(current_app._get_current_object(), user=_get_user())</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">True</span></span>
<span class="line"></span>
<span class="line"><span class="function"><span class="keyword">def</span> <span class="title">logout_user</span><span class="params">()</span>:</span></span>
<span class="line"> <span class="comment"># 从ctx取的user</span></span>
<span class="line"> user = _get_user()</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> <span class="string">'user_id'</span> <span class="keyword">in</span> session:</span>
<span class="line"> session.pop(<span class="string">'user_id'</span>)</span>
<span class="line"></span>
<span class="line"> <span class="keyword">if</span> <span class="string">'_fresh'</span> <span class="keyword">in</span> session:</span>
<span class="line"> session.pop(<span class="string">'_fresh'</span>)</span>
<span class="line"></span>
<span class="line"> cookie_name = current_app.config.get(<span class="string">'REMEMBER_COOKIE_NAME'</span>, COOKIE_NAME)</span>
<span class="line"> <span class="keyword">if</span> cookie_name <span class="keyword">in</span> request.cookies:</span>
<span class="line"> session[<span class="string">'remember'</span>] = <span class="string">'clear'</span></span>
<span class="line"> <span class="comment">#发送信号</span></span>
<span class="line"> user_logged_out.send(current_app._get_current_object(), user=user)</span>
<span class="line"></span>
<span class="line"> current_app.login_manager.reload_user()</span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">True</span></span>
</pre></td></tr></table></figure> 

### [](https://ly798.github.io/2016/09/22/flask-login/#login_required "login_required")login_required

判断current_user是否登录，否则会找到app的login_view，使用redrect重定向。
<figure class="highlight python"><table><tr><td class="gutter"><pre><span class="line">1</span>
<span class="line">2</span>
<span class="line">3</span>
<span class="line">4</span>
<span class="line">5</span>
<span class="line">6</span>
<span class="line">7</span>
<span class="line">8</span>
<span class="line">9</span>
</pre></td><td class="code"><pre><span class="line"><span class="function"><span class="keyword">def</span> <span class="title">login_required</span><span class="params">(func)</span>:</span></span>
<span class="line"><span class="decorator"> @wraps(func)</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">decorated_view</span><span class="params">(*args, **kwargs)</span>:</span></span>
<span class="line"> <span class="keyword">if</span> current_app.login_manager._login_disabled:</span>
<span class="line"> <span class="keyword">return</span> func(*args, **kwargs)</span>
<span class="line"> <span class="keyword">elif</span> <span class="keyword">not</span> current_user.is_authenticated:</span>
<span class="line"> <span class="keyword">return</span> current_app.login_manager.unauthorized()</span>
<span class="line"> <span class="keyword">return</span> func(*args, **kwargs)</span>
<span class="line"> <span class="keyword">return</span> decorated_view</span>
</pre></td></tr></table></figure>

### [](https://ly798.github.io/2016/09/22/flask-login/#current_user "current_user")current_user

使用了werlzeng 的 LocalProxy代理获取到ctx top的user

### [](https://ly798.github.io/2016/09/22/flask-login/#user_loader "user_loader")user_loader

就是将装饰的方法作为了回调函数用来通过id获取到user对象

### [](https://ly798.github.io/2016/09/22/flask-login/#u4E00_u4E2A_u95EE_u9898 "一个问题")一个问题

在一些例子上验证当前用户是否登陆时使用的是`current_user.is_authenticated()`，自己使用会发生错误，

在代码中的 UserMixin类实现这几个方法，但是使用了`@property`
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
</pre></td><td class="code"><pre><span class="line"><span class="class"><span class="keyword">class</span> <span class="title">UserMixin</span><span class="params">(object)</span>:</span></span>
<span class="line"><span class="decorator"> @property</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">is_active</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">True</span></span>
<span class="line"></span>
<span class="line"><span class="decorator"> @property</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">is_authenticated</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">True</span></span>
<span class="line"></span>
<span class="line"><span class="decorator"> @property</span></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">is_anonymous</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">return</span> <span class="keyword">False</span></span>
<span class="line"></span>
<span class="line"> <span class="function"><span class="keyword">def</span> <span class="title">get_id</span><span class="params">(self)</span>:</span></span>
<span class="line"> <span class="keyword">try</span>:</span>
<span class="line"> <span class="keyword">return</span> unicode(self.id)</span>
<span class="line"> <span class="keyword">except</span> AttributeError:</span>
<span class="line"> <span class="keyword">raise</span> NotImplementedError(<span class="string">'No `id` attribute - override `get_id`'</span>)</span>
</pre></td></tr></table></figure>

所以使用`current_user.is_authenticated`即可，之前的用法有两种可能：

*   版本问题，在0.3.0之前包括0.3.0使用`current_user.is_authenticated()`
*   在 User model 中重写了方法，不使用`@property`，两种方式都可用
