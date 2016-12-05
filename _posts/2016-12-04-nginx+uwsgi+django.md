---
layout: post
title:  "nginx+uWsgi+django"
date:   2016-12-04 23:19:22 +0800
categories: django
---

- nginx是处理静态请求是强项,nginx+uwsgi的方式,将nginx作为服务器最前端，它将接收请求,将所有静态请求自己来处理,将所有非静态请求通过uwsgi传递给Django，由Django来进行处理，从而完成一次Web请求。

- 首先安装uwsgi

```
sudo pip3 install uwsgi
```
这一步基本上没什么问题,也不用测试uwsgi了

- 安装nginx

```
sudo apt-get install nginx
```
nginx命令

```
/etc/init.d/nginx start  #启动
/etc/init.d/nginx stop  #关闭
 /etc/init.d/nginx restart  #重启
```

启动之后,在浏览器里输入127.0.0.1或者localhost,出现欢迎页面就OK,稍后再修改配置

- 配置
在django项目的目录下(manage.py的同级目录)下新建配置文件my_project.ini,名字根据自己意愿,内容如下

```
[uwsgi]
socket =:8000 #指定项目执行的端口号
chdir  = /home/myweb # 项目目录
module  = myweb.wsgi   #可以认为指向项目里的wsgi.py文件
master   = true
processes    = 4  #进程数
vacuum = true
```

也有写成xml的,没试过,应该可以

```xml
<uwsgi><socket>:8000</socket><chdir>/home/work/src/sites/testdjango1/testdjango/mysite</chdir><module>django_wsgi</module><processes>4</processes> <!-- 进程数 --><daemonize>uwsgi.log</daemonize></uwsgi>
```

通过uwsgi命令读取myweb_uwsgi.ini文件启动项目

```
uwsgi --ini my_project.ini
```


正常最后几行如下


```
spawned uWSGI master process (pid: 18554)
spawned uWSGI worker 1 (pid: 18556, cores: 1)
spawned uWSGI worker 2 (pid: 18557, cores: 1)
spawned uWSGI worker 3 (pid: 18558, cores: 1)
spawned uWSGI worker 4 (pid: 18559, cores: 1)
```


修改nginx的配置文件,不同版本和系统不太一样,本人的ubun16版本,配置文件目录为


```
/etc/nginx/sites-enabled/default
```


或者在conf.d下新建配置以cnf结尾的配置文件


```
server {
	listen 8011;
	server_name 127.0.0.1;
	location / {
		include uwsgi_params;
		uwsgi_pass 127.0.0.1:8000;  #要和ini文件里的一个端口号
		uwsgi_read_timeout 3;
	}
	location /static {
		expires 5d;
		autoindex on;
  add_header Cache-Control private;
		alias /home/myweb/collected_static; #静态文件目录
	}
```

linsen是对外的端口号
- 其他
把setting.py里的debug设置为false
关于静态文件

```
STATIC_ROOT = os.path.join(BASE_DIR, 'collected_static')
```

运行```python manage.py collectstatic```会把静态文件全部拷贝到 settings.py 中设置的 STATIC_ROOT 文件夹中
static_root文件夹就设置成nginx static localtion 的alias
这样就可以了
设置完毕后,访问localhost:8011就可以访问页面了
