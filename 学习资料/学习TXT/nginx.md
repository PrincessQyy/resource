# nginx
###  nginx configure

- /root/nginx-1.13.4/configure文件

-

	[root@server6 nginx-1.13.4]# cat configure 
	#!/bin/sh
	
	# Copyright (C) Igor Sysoev
	# Copyright (C) Nginx, Inc.
	
	
	LC_ALL=C
	export LC_ALL

	#处理configure命令的参数
	. auto/options  

	#初始化后续将产生的文件路径
	. auto/init 

	#分析Nginx的源码结构
	. auto/sources 

	#编译过程中所有目录文件生成的路径，默认objs 
	test -d $NGX_OBJS || mkdir -p $NGX_OBJS 
	
	#开始准备建立ngx_auto_headers.h、autoconf.err等必要的编译文件	
	echo > $NGX_AUTO_HEADERS_H
	echo > $NGX_AUTOCONF_ERR
	
	#向abjs/ngx_auto_config.h写入命令行
	echo "#define NGX_CONFIGURE \"$NGX_CONFIGURE\"" > $NGX_AUTO_CONFIG_H
	
	#判断DEBUG标志，如果有，那么在objs/ngx_auto_config.h文件中写入DEBUG宏
	if [ $NGX_DEBUG = YES ]; then
	    have=NGX_DEBUG . auto/have
	fi
	
	#检查操作系统参数是否支持后续编译
	if test -z "$NGX_PLATFORM"; then
	    echo "checking for OS"
	
	    NGX_SYSTEM=`uname -s 2>/dev/null`
	    NGX_RELEASE=`uname -r 2>/dev/null`
	    NGX_MACHINE=`uname -m 2>/dev/null`
	
	    echo " + $NGX_SYSTEM $NGX_RELEASE $NGX_MACHINE"
	
	    NGX_PLATFORM="$NGX_SYSTEM:$NGX_RELEASE:$NGX_MACHINE";
	
	    case "$NGX_SYSTEM" in
	        MINGW32_* | MINGW64_* | MSYS_*)
	            NGX_PLATFORM=win32
	        ;;
	    esac
	
	else
	    echo "building for $NGX_PLATFORM"
	    NGX_SYSTEM=$NGX_PLATFORM
	fi
	
	#检查并设置编译器，如GCC是否安装、GCC版本是否支持后续编译nginx
	. auto/cc/conf
	
	#对非window操作系统定义一些必要的头文件，并检查是否存在，以此决定configure后续步骤成功
	if [ "$NGX_PLATFORM" != win32 ]; then
	    . auto/headers
	fi
	
	#定义一些特定的操作系统相关的方法并检查当前环境是否支持
	. auto/os/conf
	
	#定义类UNIX操作系统中递用的头文件和系统调用，并检查当前环境是否支持
	if [ "$NGX_PLATFORM" != win32 ]; then
	    . auto/unix
	fi
	
	. auto/threads
	
	#生成ngx_modules.c文件，这个文件会被编译到Nginx中，文件定义了ngx_modules数组。ngx_modules指明运行期间会参与到请求的处理中，而且顺序非常敏感
	. auto/modules

	#检查Nginx在链接期间需要链接的第三方静态库、动态库或者目标文件是否存在
	. auto/lib/conf
	

	#处理Nginx安装后的路径
	case ".$NGX_PREFIX" in
	    .)
	        NGX_PREFIX=${NGX_PREFIX:-/usr/local/nginx}
	        have=NGX_PREFIX value="\"$NGX_PREFIX/\"" . auto/define
	    ;;
	
	    .!)
	        NGX_PREFIX=
	    ;;
	
	    *)
	        have=NGX_PREFIX value="\"$NGX_PREFIX/\"" . auto/define
	    ;;
	esac
	

	#处理Nginx安装后，二进制文件、pid、lock等其他文件的路径
	if [ ".$NGX_CONF_PREFIX" != "." ]; then
	    have=NGX_CONF_PREFIX value="\"$NGX_CONF_PREFIX/\"" . auto/define
	fi
	
	have=NGX_SBIN_PATH value="\"$NGX_SBIN_PATH\"" . auto/define
	have=NGX_CONF_PATH value="\"$NGX_CONF_PATH\"" . auto/define
	have=NGX_PID_PATH value="\"$NGX_PID_PATH\"" . auto/define
	have=NGX_LOCK_PATH value="\"$NGX_LOCK_PATH\"" . auto/define
	have=NGX_ERROR_LOG_PATH value="\"$NGX_ERROR_LOG_PATH\"" . auto/define
	
	have=NGX_HTTP_LOG_PATH value="\"$NGX_HTTP_LOG_PATH\"" . auto/define
	have=NGX_HTTP_CLIENT_TEMP_PATH value="\"$NGX_HTTP_CLIENT_TEMP_PATH\""
	. auto/define
	have=NGX_HTTP_PROXY_TEMP_PATH value="\"$NGX_HTTP_PROXY_TEMP_PATH\""
	. auto/define
	have=NGX_HTTP_FASTCGI_TEMP_PATH value="\"$NGX_HTTP_FASTCGI_TEMP_PATH\""
	. auto/define
	have=NGX_HTTP_UWSGI_TEMP_PATH value="\"$NGX_HTTP_UWSGI_TEMP_PATH\""
	. auto/define
	have=NGX_HTTP_SCGI_TEMP_PATH value="\"$NGX_HTTP_SCGI_TEMP_PATH\""
	. auto/define
	

	#创建编译时使用的objs/Makefile文件
	. auto/make

	#为objs/Makefile加入需要连接的第三方静态库、动态库或者目标文件
	. auto/lib/make
	
	#为objs/Makefile加入install功能，当执行make install时将编译生成的必要问价复制到安装建立必要的目录
	. auto/install
	
	# 在ngx_auto_config.h文件中加入NGX_SUPPRESS_WARN宏、NGX_SMP宏
	. auto/stubs
	
	#在ngx_auto_config.h文件中指定NGX_USER和NGX_GROUP宏，如果执行configure时没有参数指定，默认两者皆为nobody
	have=NGX_USER value="\"$NGX_USER\"" . auto/define
	have=NGX_GROUP value="\"$NGX_GROUP\"" . auto/define
	
	if [ ".$NGX_BUILD" != "." ]; then
	    have=NGX_BUILD value="\"$NGX_BUILD\"" . auto/define
	fi
	
	#显示configure执行的结果，如果失败，则给出原因
	. auto/summary
### Nginx的命令行控制

1. 默认启动方式:/usr/local/nginx/sbin/nginx
2. 另行指定配置文件的启动方式：/usr/local/nginx/sbin/nignx -c /tmp/nginx.conf
3. 另行指定安装目录的启动方式：/usr/local/nginx/sbin/nginx -p /usr/local/nginx/
4. 另行指定全局配置项的启动方式：/usr/local/nginx/sbin/nginx -g "pid /va/nginx/test.pid;"
5. 测试配置信息是否有错误：/usr/lcoal/nginx/sbin/nginx -t
6. 在测试配置信息是否有错误：/usr/local/nginx/sbin/nginx -t -q
7. 显示版本信息：/usr/local/nginx/sbin/nginx -v
8. 显示编译阶段的参数：/usr/local/nginx/sbin/nginx -v
9. 快速地停止服务：/usr/local/nginx/sbin/nginx -s stop；相当于kill -s SIGQUIT <nginx master pid>
10. “优雅”地停止服务：/usr/local/nginx/sbin/nginx -s quit;快速停止服务时，worker进程与master进程在收到信号后会立刻跳出循环，退出进程。而“优雅”地停止服务时，首先会关闭监听端口，停止接收新的连接，然后把当期那正在处理的连接全部处理完，最后再退出进程。相当于kill -s SIGWINCH <nginx worker pid>
11. 使运行中的Nginx重读配置项并生效：/usr/local/nginx/sbin/nignx -s reload ;nginx先检查新的配置项是否有误，如全部正确就以“优雅”的方式关闭，再重新启动Nginx来实现这个目的。相当于kill -s SIGHUP <nginx master pig>
12. 日志文件回滚：/usr/local/nginx/sbin/nginx -s reopen
13. 平滑升级Nginx

-

	1)通知正在运行的旧版本Nginx准备升级。通过向master进程发送USR2信号可达到目的。运行中的Nginx会将pid文件重命名；
	2）启动新版本的Nginx，这时通过ps命令可发现新旧版本的Nginx在同时运行。
	3）通过kill命令向旧版本的master进程发送SIGQUIT信号，以“优雅”的方式关闭旧版本的Nginx；新版本的Nginx服务运行，此时平滑升级完成。

### Nginx的配置
#### 运行中的Nginx进程间的关系
- 一般情况下，worker进程的数量与服务器上的CPU核心数相等。
- master负责管理worker进程。
- worker进程之间通过共享内存、原子操作等一些进程间通信机制来实现负载均衡等功能。
- 一个worker进程可以同时处理的请求数只受限于内存大小，且在架构设计上，不同的worker进程之间处理并发请求几乎没有同步锁的限制，worker进程通常不会进入睡眠状态。
#### 模块


-

	#user  nobody;
	worker_processes  1;
	
	#error_log  logs/error.log;
	#error_log  logs/error.log  notice;
	#error_log  logs/error.log  info;
	
	#pid        logs/nginx.pid;
	
	
	events {
	    worker_connections  1024;
	}
	
	
	http {
	    include       mime.types;
	    default_type  application/octet-stream;
	
	    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
	    #                  '$status $body_bytes_sent "$http_referer" '
	    #                  '"$http_user_agent" "$http_x_forwarded_for"';
	
	    #access_log  logs/access.log  main;
	
	    sendfile        on;
	    #tcp_nopush     on;
	
	    #keepalive_timeout  0;
	    keepalive_timeout  65;
	
	    #gzip  on;
	
	    server {
	        listen       80;
	        server_name  localhost;
	
	        #charset koi8-r;
	
	        #access_log  logs/host.access.log  main;
	
	        location / {
	            root   html;
	            index  index.html index.htm;
	        }
	
	        #error_page  404              /404.html;
	
	        # redirect server error pages to the static page /50x.html
	        #
	        error_page   500 502 503 504  /50x.html;
	        location = /50x.html {
	            root   html;
	        }
	
	        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
	        #
	        #location ~ \.php$ {
	        #    proxy_pass   http://127.0.0.1;
	        #}
	
	        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
	        #
	        #location ~ \.php$ {
	        #    root           html;
	        #    fastcgi_pass   127.0.0.1:9000;
	        #    fastcgi_index  index.php;
	        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
	        #    include        fastcgi_params;
	        #}
	
	        # deny access to .htaccess files, if Apache's document root
	        # concurs with nginx's one
	        #
	        #location ~ /\.ht {
	        #    deny  all;
	        #}
	    }
	
	
	    # another virtual host using mix of IP-, name-, and port-based configuration
	    #
	    #server {
	    #    listen       8000;
	    #    listen       somename:8080;
	    #    server_name  somename  alias  another.alias;
	
	    #    location / {
	    #        root   html;
	    #        index  index.html index.htm;
	    #    }
	    #}
	
	
	    # HTTPS server
	    #
	    #server {
	    #    listen       443 ssl;
	    #    server_name  localhost;
	
	    #    ssl_certificate      cert.pem;
	    #    ssl_certificate_key  cert.key;
	
	    #    ssl_session_cache    shared:SSL:1m;
	    #    ssl_session_timeout  5m;
	
	    #    ssl_ciphers  HIGH:!aNULL:!MD5;
	    #    ssl_prefer_server_ciphers  on;
	
	    #    location / {
	    #        root   html;
	    #        index  index.html index.htm;
	    #    }
	    #}
	
	}
块设备项可以嵌套，内层块直接继承外层块；
当内外层块中的配置发生冲突时，究竟是以内层块还是外层快的配置为准，取决于解析这个配置项的模块。
### 配置项的语法格式

配置项名 配置项值1 配置项值1 ···
配置项值取决于解析这个配置项的模块
### 配置项的注释
‘#’
### 配置项的单位
空间：K或k千字节；M或m兆字节
时间：ms,s,m（分钟）,h,d,w（月）,M,y(年)
### Nginx服务的基本配置
4类：用于调试、定位问题的配置项；正常运行的必备配置项；优化性能的配置项；事件类配置项

- 用于调试、定位问题的配置项：

-

	1）是否是守护进程（脱离终端且后台运行）方式运行nginx。
	语法：daemon on|off;
	默认：daemon on;
	脱离终端是为了避免进程执行过程中的信息在任何一个终端上显示，进程不会被任何终端所产生的信息打断。提供关闭守护进程模式是为了方便恩宗调试Nginx
	2）是否以master/worker方式工作。
	语法：master_process on|off;
	默认：master_process on;
	提供master_process配置也是为了方便跟踪调试Nginx
	3）error日志的设置
	语法：error_log /path/file level;
	默认：error_log logs/error.log error;
	error日志是定位Nginx问题的最佳工具。
	/path/file默认情况是logs/error.log文件；
	/path/file可以是/dev/null,关闭error日志的唯一手段 
	/path/stderr，日志输出到标准错误文件中
	level是日志输出级别，取值范围：debug info notice warn error crit alert emerg ,左至右级别依次增大，大于或等于该级别的日志会被输出到/path/file文件中，小于该级别的日志则不会输出。
	4）是否处理几个特殊的调试点
	语法：debug_points[stop|abort]
	用于帮助跟踪调试nginx的。
	如果设置为debug_points为stop，那么Nginx的代码执行到这些调试点的时候会发出SIGSTOP信号以用于调试。
	如果debug_points设置为abort，会产生一个coredump文件，可以使用gdb来查看Nginx当时的各种信息。
	5）仅对指定的客户端debug级别的日志
	语法：debug_connection[IP|IDR]
	events {
		debug_connection xxx.xxx.xx.xx;
		debug_connection xx.xxx.xx.xx/24;
	}
	对修复bug很有用，特别是定位高并发请求下才会发生的问题。
	注意：使用debug_connection前，需确保在执行configure时已经加入--with-debug参数，否则不会生效。
	6）限制coredump核心转储文件的大小
	语法：worker_rlimit_core size;
	当进程发生错误或收到信号而终止时，系统会将进程执行时的内存内容写入一个文件（core文件），以作为调试之用，这就是所谓的核心转储(core dumps)。如果不加以限制，可能一个core文件会达到几GB，这样随便coredumps几次就会把磁盘占满。配置worker_rlimit_core配置可限制core文件的大小，从而有效帮助用户定位问题。
	7）指定coredump文件生成目录
	语法：working_directory path;
	worker进程的工作目录。配置项唯一用途就是设置coredump文件所放置的目录，协助定位问题。因此，需确保worker进程有权限向working_directory指定的目录中写入文件。


- 正常运行的配置项

-

	1）定义环境变量
	语法：env VAR|VAR=VALUE;
	env TESTPATH=/tmp/;
	2）嵌入其他配置文件
	语法：include /path/file;
	将其他配置文件嵌入到nginx.conf文件中，路径可以绝对或相对
	3）pid文件的路径
	语法：pid path/file
	默认 pid logs/nginx.pid
	注意：应确保Nginx有权在相应的目标中创建pid文件，该文件直接影响Nginx是否可以运行
	4）Nginx worker进程运行的用户及用户组
	语法：user username [groupname]
	默认：user nobody nobody
	user用于设置master进程启动后，fork出的worker进程运行在哪个用户和用户组下；
	当按照“user username；”设置时，用户组名和用户名相同
	5）指定Nginx worker进程可以打开的最大句柄描述符个数
	语法：worker_rlimit_nofile limit;
	设置一个worker进程可以打开的最大文件句柄数
	6）限制信号队列
	语法：worker_rlimit_sigpending limit;
	设置每个用户发往Nginx的信号队列的大小。当某个用户的信号队列满了，这个用户再发送信号量会被丢掉。

- 优化性能的配置项

-

	1）Nginx worker进程个数
	语法：worker_process number;
	默认：worker_process 1;
	一般配置为CPU内核数
	2）绑定Nginx worker进程到指定的CPU内核
	语法：worker_cpu_affinity cpumask [cpumask...]
	例子：
	worker_process 4;
	worker_cpu_affinity 1000 0100 0010 0001;
	3）SSL硬件加速
	语法：ssl_engine device;
	查看硬件加速设备命令 openssl engine -t
	4）系统调用gettimeofday的执行频率
	语法：timer_resolution t;
	在目前的大多数内核中，如x86_64体系架构。gettimeofday只是一次vsyscall，仅仅对共享内存页中的数据做访问，并不是通常的系统调用，代价并不大，一般不必使用这个配置。
	5）Nginx worker进程优先级设置
	语法： worker_priority nice;
	默认：worker_priority 0;
	优先级由静态优先级和内核根据进程执行情况所做的动态调整。
	nice值是进程的静态优先级，取值范围-20~+19,-20是最高优先级；一般不建议比内核进程（-5）的nice值小；


-

# 多核操作系统3种处理模式（SMP+AMP+BMP）
http://blog.csdn.net/honour2sword/article/details/45248121