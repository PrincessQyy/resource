### nginx请求的11个阶段

一、nginx接收请求过程
1、读取请求头

分为解析请求行和解析请求头两部分组成

2、读取请求体

二、nginx处理请求阶段及openresty对应的各指令

配置加载阶段：

init_by_lua：初始化配置、加载全局模块等

init_worker_by_lua

1、POST_READ阶段：读取请求阶段，这个阶段其实并不是处理请求阶段的，读取请求头和请求体。

2、SERVER_REWRITE阶段：根据请求查找对应的虚拟主机(SERVER，即找对应的server区段)。

主要指令：set_by_lua

3、FIND_CONFIG阶段：查找配置阶段，即根据请求uri查找匹配的lcation。

4、REWRITE：uri重写阶段，对lcation级别的重写。

主要指令是：rewrite_by_lua*

5、POST_REWRITE:判断上一阶段是否进行uri重写。

6、PREACCESS：访问控制预处理阶段，表明已经找到了匹配的location或者server。

主要指令：header_filter_by_lua, body_filter_by_lua

7、ACCESS:访问控制阶段，在该阶段主要做权限控制。

主要指令是：access_by_lua*

8、POST_ACCESS：处理ACCESS的处理结果

9、TRY_FILES：仅当配置了try_files指令时生效，实际上该指令不常用，它的功能是指定一个或者多个文件或目录，最后一个参数可以指定为一个location或一个返回码

10、CONTENT：内容处理阶段，业务逻辑

对应的openresty指令：content_by_lua*

11、LOG：日志记录阶段

对应的openresty指令：log_by_lua* (主要是log_by_lua,log_by_lua_block等)