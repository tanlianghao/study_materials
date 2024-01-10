### window下的安装使用

#### nginx的作用

主要是起到反向代理的功能：反向代理实际上就是一台负责转发的代理服务器，看起来是充当了真正服务器的功能，但实际上不是这样的，代理服务器只是充当一个转发的功能，并且能从真正的服务器那获取到返回的数据。

#### 下载安装及配置流程

1. 直接百度nginx官网，下载相对应的应用。解压出来。

1. 在解压出来的文件下面，打开命令行，使用start nginx开启nginx，ps：这个地方命令行会一闪而过，不要怕这个是正常现象，nginx已经成功开启。

1. 在该文件夹下面有一个conf文件夹，找到nginx.conf文件，打开可以看到一段代码

```
server {
        listen       80;
        server_name  localhost;
        location / {
            root   html;
            index  index.html index.htm;
        }
```

server相当与一个代理服务器，可以配置多个，

- listen：表示的是当前的代理服务器监听的端口，默认是80，如果我们要配置多个server，这个端口必须要配置不一样，不然会出错。

- server_name：表示监听之后需要监听的域名，代码中我们直接转到本地，这时是直接转到nginx文件夹内。

- location：表示匹配的路径，代码中配置的是/ 表示所有的请求都被匹配到这里。location的语法规则

```
location [=|~|~*|^~]/uri/{...}
= 开头表示精确匹配
^~ 开头表示uri以某个常规的字符串开头，理解为匹配url路径即可，nginx不对url做编码，因此请求如果是
/static/20%/aa,可以被规则^~/static/ /aa匹配到（注意中间是空格）
~ 开头表示区分大小写的正则匹配
~* 开头表示不区分大小写的正则匹配
!~和!~*分别为区分大小写不匹配及不区分大小写不匹配 的正则
/ 通用匹配，任何请求都会被匹配到
如果是在server中有多个location配置的情况下，匹配的顺序是首先匹配=,其次匹配^~,其次是按文件中顺序的
正则匹配，最后交给/通用匹配。停止匹配的时候，按当前匹配规则处理请求。
location 匹配的路径是在location块级下面root/proxy_pass的基础上去查找的。
```

- root：配置了root这时表示当匹配这个请求的路径时，将会在这个文件夹中寻找对应的文件

- index：当没有指定主页的时候，默认会选择指定的文件，可以有多个，当第一个不匹配的时候再往后面找。

怎么在访问localhost的时候转到指定的页面上去。代码如下。修改的其实就两个地方，其中有一个新元素，proxy_pass,表示的是代理路径。相当是转发，而不是像root指定文件夹，修改之后，使用命令nginx -s reload进行重新启动，(ps:如果是需要查看配置文件有没有问题，可以输入nginx -t。)，记得给浏览器清理缓存。

```javascript
server_name localhost:80;  
  
location / {  
    proxy_pass http://baidu.com；  
}
```



### mac下的安装和使用

1. 根据网上介绍，mac是自带nginx的，不过我尝试访问http://127.0.0.1显示的是无法访问，所以重头安装了一下。主要是通过Homebrew套件管理器下载的，安装Homebrew之后，使用命令 brew install nginx进行下载，验证是否安装成功则使用命令 nginx -v进行检验。

1. 进入nginx的安装目录。我的是安装在 /usr/local/etc/nginx/ ，打开该目录下面的nginx.conf文件，可以使用命令 cd  /usr/local/etc/nginx/  进入该目录 ，然后使用sudo vim nginx.conf快速打开文件。该文件中其他地方都可以不用动。之后如果要设置反向代理的时候需要在什么地方设置conf文件，在该文件的最后有一段代码,我的显示是这样 include servers/*; 这段表明之后设置的conf文件默认会在servers这个目录下面，有些版本是默认在conf.d这个文件夹下面。

1. 到这步的时候差不多就成功了，在修改servers下面的文件之后，执行 sudo nginx -s reload的时候出现错误，错误提示为`nginx: [error] open() "/usr/local/var/run/nginx.pid"` `failed (2: No such file or directory)` ，原因是缺失nginx.pid文件，解决方案是执行以下命令。

   ```javascript
   sudo nginx -c /usr/local/etc/nginx/nginx.conf
   ```

到此 nginx安装完成。

#### 代理

1. 首先，需要在servers文件夹下面创建conf文件。并创建如下格式的内容；

```javascript
// 负载均衡
upstream fronted {
    sever 127.0.0.1:9080;
}
server {
        listen       80;//监听的端口
        //监听的域名，输入该域名可以访问到对应的项目,此处自定义的域名必须要有对应的ip地址，否则
    // 无法进行DNS解析。从而导致页面可以在处于加载中。
        server_name  docker-web0.test4.9game.cn;
        location / {
          proxy_pass http://froned;//需要代理的ip
        }
}
```

1. 完成上面的工作之后，在mac中找到hosts文件。把本机ip指向你上面需要监听的域名；比如` 127.0.0.1   aaa.9game.cn`

#### 解决跨域

nginx可以被用来解决跨域问题。比如我现在a.html访问地址是http://127.0.0.1:5500/lsProject/a.html，现在需要通过ajax发起一个请求，请求地址是http://127.0.0.1:3000/test?name=xiaohong;，由于端口不同所以存在跨域问题。解决办法是通过nginx给两个都统一一个端口，并使用nginx全局变量rewrite重写url，代码如下：

```javascript
// ngm.conf 配置文件
server {
    listen    80;
    server_name    127.0.0.1;

    localion /lsProject {
        rewrite    ^/lsProject/(.*)$ $1 break;
        proxy_pass    http://127.0.0.1:5500; 
    }

    location /test {
        rewrite    ^/test(\?.*)$ $1 break;
        proxy_pass    http://127.0.0.1:3000;
    }
}
代理同一个端口80，并根据不同的路径把两个不同的路径给分离开，$1是为了把匹配的后面路径重写到新的url中，
并使用break停止后面的匹配

需要注意的是 location匹配上的路径，使用proxy_pass重定向url后，location后面匹配的路径同样
会写到新的url后面。
```

