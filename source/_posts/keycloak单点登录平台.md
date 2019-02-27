---
title: keycloak单点登录平台
tags: [keycloak]
date: 2018-12-03 10:15:45
---

introduction
============

keycloak[(https://github.com/keycloak/keycloak)](https://github.com/keycloak/keycloak)提供了统一鉴权和权限控制功能，支持SSO(单点登录)，适合来统一多个应用的认证系统。常用的协议有[oidc](https://openid.net/connect/)
<!-- more -->

简单部署
======

使用docker方式部署，生产环境作好数据库备份
```
docker run --name db -d -e POSTGRES_USER=keycloak -e POSTGRES_PASSWORD=password -e POSTGRES_DB=keycloak postgres:11.1
docker run --name keycloak -d --link db:db -e DB_VENDOR=postgres -e DB_ADDR=db -e KEYCLOAK_USER=admin -e KEYCLOAK_PASSWORD=admin -p 8080:8080 jboss/keycloak:4.8.1.Final
```
浏览器登录`http://127.0.0.1:8080`为keycloak的管理界面

应用集成
=======

keycloak支持对部分语言的adapter(可以直接将adapter集成到应用中)，adapter方式是应用直接将适配器集成到应用，应用调用adapter的接口来进行认证即可:
 - java(servlet、jboos、sprintboot、jetty、tomcat)
 - javascript(第三方的angular-keycloak)
 - nodejs

使用这些adapter的架构:
```
                                 ------------
                                 | keycloak |
                                 ------------
                                      |
                                      |
                             ---------|----------------------
   -----------               |   ------------------------   |
   |  浏览器  | <---------------> | app(include adapter) |   |
   -----------               |   ------------------------   |
                             --------------------------------
```

具有adapter的语言应用可以很少代价的集成keycloak，其他语言(如python)集成可以依赖oidc协议实现的库来集成，但或多或少有缺陷(如不支持单点登出)。所以keycloak另外提供了keycloak-gatekeeper代理程序来解决，后面介绍。


keycloak-gatekeeper
===================

如果语言没有adapter支持，可以使用keycloak-gatekeeper代理服务实现。keycloak-gatekeeper是一个web服务的代理程序，可以代替被代理的web服务进行认证，可以当作带认证nginx反向代理，只不过keycloak-gatekeeper的后端使用的还是keycloak。

使用keycloak-gatekeeper的架构:
```
                                 ------------
                                 | keycloak |
                                 ------------
                                      |
                                      |
                             ---------|---------------------------------
   -----------               |   -----------------------      -------  |
   |  浏览器  | <---------------> | keycloak-gatekeeper | <--> | app |  |
   -----------               |   -----------------------      -------  |
                             -------------------------------------------
```

集成
----

以flask为例，在本地运行个简单程序
```python
from flask import Flask, request, redirect, abort
app = Flask(__name__)

@app.route('/')
def hello_world():
    if not request.headers.get('X-Auth-Userid') == 'user_1':
        abort(403)
    txt = '''Hello, World!
    <a href="/oauth/logout?redirect=">logout</a>
    <hr>
    '''
    for h in request.headers:
        txt += "<p>%s: %s</p>" % h
    return txt
```
启动
```
flask run -h 127.0.0.1 -p 5001
```

运行一个keycloak-gatekeeper带代理5001
```
docker run --rm --name keycloak-gatekeeper --net host -v `pwd`/config.yaml:/config.yaml ly798/keycloak-gatekeeper
```
config.yml部分配置如下
```
listen: :8181
redirection-url: http://192.168.110.41:8181
upstream-url: http://127.0.0.1:5001
```

浏览器访问`http://192.168.110.41:8181`即可，这样使没有adapter的应用也能简单的集成。
