# Nginx的使用
---

## 编译Nginx
Nginx的第三方模块会被编译进Nginx的二进制文件中，如果想要使用第三方模块，则需要使用编译安装的方法来安装Nginx。  

Nginx的编译安装是标准的Linux源码安装方式： 
* Configure
* make
* make install

### 下载Nginx源码
```sh
$ wget http://nginx.org/download/nginx-1.16.1.tar.gz
$ tar -xzf nginx-1.16.1.tar.gz
$ cd nginx-1.16.1
```

### 源码目录分析
```sh
$ tree -L 2 nginx-1.16.1
nginx-1.16.1
├── auto                # 用于编译的目录
│   ├── cc              # dir
│   ├── define
│   ├── endianness
│   ├── feature
│   ├── have
│   ├── have_headers
│   ├── headers
│   ├── include
│   ├── init
│   ├── install
│   ├── lib             # dir
│   ├── make
│   ├── module
│   ├── modules
│   ├── nohave
│   ├── options
│   ├── os              # dir
│   ├── sources
│   ├── stubs
│   ├── summary
│   ├── threads
│   ├── types           # dir
│   └── unix
├── CHANGES             # 每个版本的变化
├── CHANGES.ru
├── conf                # 配置文件的示例文件
│   ├── fastcgi.conf
│   ├── fastcgi_params
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── nginx.conf
│   ├── scgi_params
│   ├── uwsgi_params
│   └── win-utf
├── configure           # 生成中间文件，是编译前的必备动作
├── contrib             
│   ├── geo2nginx.pl
│   ├── README
│   ├── unicode2nginx
│   └── vim
├── html                # 标准的html文件
│   ├── 50x.html
│   └── index.html
├── LICENSE
├── man                 # linux的帮助文件
│   └── nginx.8
├── README              
└── src                 # 源码文件夹
    ├── core
    ├── event
    ├── http
    ├── mail
    ├── misc
    ├── os
    └── stream
```

### 配置
```sh
$ ./configure --help
...
$ ./configure --prefix=/home/qniu/nginx
```

### 配置生成的中间文件
```sh
$ tree objs
objs
├── autoconf.err
├── Makefile
├── ngx_auto_config.h
├── ngx_auto_headers.h
├── ngx_modules.c           # 决定了编译时有哪些模块会被编译进nginx
└── src
    ├── core
    ├── event
    │   └── modules
    ├── http
    │   ├── modules
    │   │   └── perl
    │   └── v2
    ├── mail
    ├── misc
    ├── os
    │   ├── unix
    │   └── win32
    └── stream
```

### 编译
```sh
$ make
# 在objs里面生成了目标文件
```

### 安装

首次安装
```sh
$ make install
$ cd /home/qniu/nginx
$ tree .
.   
├── conf                    # 配置文件
│   ├── fastcgi.conf
│   ├── fastcgi.conf.default
│   ├── fastcgi_params
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types
│   ├── mime.types.default
│   ├── nginx.conf
│   ├── nginx.conf.default
│   ├── scgi_params
│   ├── scgi_params.default
│   ├── uwsgi_params
│   ├── uwsgi_params.default
│   └── win-utf
├── html
│   ├── 50x.html
│   └── index.html
├── logs                # 日志文件
└── sbin
    └── nginx           # nginx二进制文件
```

## 配置Nginx
Nginx配置语法：
* 配置文件由指令与指令块构成
* 每条指令以；分号结尾，指令与参数键以空格符号分隔
* 指令块以{}大括号将多条指令组织在一起
* include语句允许组合多个配置文件以提升可维护性
* 使用#符号添加注释，提高可读性
* 使用$符号使用变量
* 部分指令的参数支持正则表达式

配置参数：
* 时间的单位
    * ms：毫秒
    * s： 秒
    * m： 分钟
    * h： 小时
    * d： 天
    * w： 周
    * M： 月，30天
    * y： 年
* 空间的单位：
    * 默认，什么都不加： 字节
    * k/K： 千字节
    * m/M： 兆字节
    * g/G： 吉字节

## Nginx命令行工具
```sh
nginx -h
nginx version: nginx/1.16.1
Usage: nginx [-?hvVtTq] [-s signal] [-c filename] [-p prefix] [-g directives]

Options:
  -?,-h         : this help 帮助文档
  -v            : show version and exit 打印版本信息
  -V            : show version and configure options then exit
  -t            : test configuration and exit   测试配置文件是否有语法错误
  -T            : test configuration, dump it and exit
  -q            : suppress non-error messages during configuration testing
  -s signal     : send signal to a master process: stop, quit, reopen, reload   发送信号
  -p prefix     : set prefix path (default: /home/qniu/nginx/)  指定运行目录
  -c filename   : set configuration file (default: conf/nginx.conf) 配置文件
  -g directives : set global directives out of configuration file   指定配置指令
```

## 重载配置文件
```sh
$ ps ax | grep nginx | grep -v grep
4661 ?        Ss     0:00 nginx: master process nginx
 4663 ?        S      0:00 nginx: worker process
# 修改配置文件
$ nginx -s reload
$ ps ax | grep nginx | grep -v grep
4661 ?        Ss     0:00 nginx: master process nginx
 4689 ?        S      0:00 nginx: worker process
```
在reload之后，nginx的master进行没变，但是worker进程被杀掉重启了

## 热部署
用一个新的nginx二进制文件更换正在运行的nginx文件
```sh
# 编译一个新的nginx二进制文件
$ ./configure --with-http_ssl_module --with-http_gzip_static_module
$ make
# 不要使用make install
$ cd /home/qniu/nginx/sbin
$ cp nginx nginx.old
$ cd -
$ cd objs
$ cp -r nginx /home/qniu/nginx/sbin/ -f
$ ls -l
total 9884
-rwxrwxr-x 1 qniu qniu 5379800 Feb 16 03:11 nginx
-rwxrwxr-x 1 qniu qniu 4737176 Feb 16 02:57 nginx.old
$ ps -ef | grep nginx | grep -v grep
qniu     15241     1  0 03:19 ?        00:00:00 nginx: master process nginx -p /home/qniu/nginx
qniu     15243 15241  0 03:19 ?        00:00:00 nginx: worker process
# 给nginx的master进程发送热部署信号
$ kill -USR2 15241
$ ps -ef | grep nginx | grep -v grep
qniu     15241     1  0 03:19 ?        00:00:00 nginx: master process nginx -p /home/qniu/nginx
qniu     15243 15241  0 03:19 ?        00:00:00 nginx: worker process
qniu     15253 15241  0 03:19 ?        00:00:00 nginx: master process nginx -p /home/qniu/nginx
qniu     15255 15253  0 03:19 ?        00:00:00 nginx: worker process
$ kill -WINCH 13195
$ ps -ef | grep nginx | grep -v grep
qniu     15241     1  0 03:19 ?        00:00:00 nginx: master process nginx -p /home/qniu/nginx
qniu     15253 15241  0 03:19 ?        00:00:00 nginx: master process nginx -p /home/qniu/nginx
qniu     15255 15253  0 03:19 ?        00:00:00 nginx: worker process
```
老的worker还在运行，但是不接受新的请求了，新的master和worker已经启动，新的请求进来之后会交给新的worker处理。老的worker处理完请求之后就会被kill掉。老的master还在运行，对于老的master进程，还能给他发送reload信号，让他重新把worker进程启动起来（回退操作），所以它不会被kill掉

## 日志切割
```sh
$ cd logs
# 备份当前日志
$ cp qniu_access.log bak.log
$ nginx -s reload
# 可以写成bash脚本然后放在crontab中
```

