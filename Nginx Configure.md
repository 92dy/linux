## 主要配置文件结构

### 正常运行相关的配置

- `user`：指定worker进程的运行身份，如组不指定，默认和用户名同名 
- `pid /PATH/TO/PID_FILE;`：指定存储nginx主进程PID的文件路径 
- `include file;`：指明包含进来的其它配置文件片断 
- `load_module file ;`：
    - `/usr/share/nginx/modules/*.conf `：模块加载配置文件
    - `/usr/lib64/nginx/modules `：指明要装载的动态模块路径

### 性能优化相关的配置

- `worker_processes number | auto;`：worker进程的数量；通常应该为当前主机的cpu的物理核心数 
- `worker_cpu_affinity 00000001 00000010 00000100 ...;`：cpu亲缘性绑定，提高缓存命中率 
- `worker_priority number;`：指定worker进程的nice值，设定worker进程优先级：[-20,20] 
- `worker_rlimit_nofile 65535;`：worker进程所能够打开的文件数量上限
- `、tcp_nodelay on | off;`：在keepalived模式下的连接是否启用TCP_NODELAY选项  当为off时，延迟发送，合并多个请求后再发送 
- `sendfile on | off;`：是否启用sendfile功能，在内核中封装报文直接发送,默认Off 


### 事件驱动相关的配置

- `worker_connections 1024;`：每个worker进程所能够打开的最大并发连接数数量
- `use epoll`：指明并发连接请求的处理方法 ,默认自动选择最优方法 
- `accept_mutex on | off`：处理新的连接请求的方法，on是轮流处理新请求，off会造成“惊群”,影响性能

### 定位调试问题

- `daemon on|off;`  是否以守护进程方式运行nignx，默认是守护进程方式 
- `master_process on|off;`  是否以master/worker模型运行nginx；默认为on,off 将不启动worker 
- `error_log file [debug|info|notice|warn|error|crit|alter|emerg ];`：错误日志文件及其级别，debug仅在编译时使 用了“--with-debug”选项时才有效 

### 套接字相关的配置

- `server {...}`：配置一个虚拟主机 
    - `listen PORT|IP:PORT:UNIX;`：监听端口
    - `server_name *.deng.com   www.deng.* `：虚拟主机的主机名称后可跟多个由空白字符分隔的字符串 
        - `server_name  www.example.com;`：精确匹配
        - `server_name *.example.com;`：左侧`*`通配符匹配
        - `server_name www.example.*;`：右侧`*`通配符匹配
        - `server_name ~^.*\.example\.com$;`：正则表达式匹配
> **Note**：匹配优先级：`精确匹配> 左侧通配 > 右侧通配 > 正则表达式 > default_server`

### 安全相关的配置

- `server_tokens off`：是否在响应报文的Server首部显示nginx版本 
- `error_page  404  =200  /404.html`：定义错误页，以指定的响应状态码进行响应


### 定义路径相关的配置 

- `root /data/www/vhost;`：设置web资源的路径映射
    - `http://www.du.com/images/logo.png` --`/data/www/vhost/images/logo.png`

- `location {}`：在一个server中location配置段可存在多个，用于实现从uri到文件系统的路 径映射；ngnix会根据用户请求的URI来检查定义的所有location，并找出一个最 佳匹配，而后应用其配置 
    - `location = / { [ configuration A ]}`：精确匹配
    - `location ~ ^/images/ { [ configuration D ] }`：以某个常规字符串开头的匹配
    - `location ~ \.(gif|jpg|jpeg)$ { [ configuration E ] }`: 对URI做正则表达式模式匹配，区分字符大小写
    - `location ~* \.(gif|jpg|jpeg)$ { [ configuration F ] }`：对URI做正则表达式模式匹配，不区分字符大小写
    > - 匹配优先级从高到低：`=` 、`^~`、 `～/～*`、 `不带符号` 

- `location /images { alias /data/www/vhost; }` : 注意`/`需要匹配
    - `http://www.du.com/images/logo.png` -- `/data/www/vhost/logo.png`

### 客户端请求的相关配置 

- `keepalive_timeout timeout [header_timeout];`:设定保持连接超时时长，0表示禁止长连接，默认为75s 
- `keepalive_requests number;`:在一次长连接上所允许请求的资源的最大数量，默认为100 
- `keepalive_disable none | browser ...;`：对哪种浏览器禁用长连接  
- `send_timeout time;`: 向客户端发送响应报文的超时时长，此处是指两次写操作之间的间隔时长， 而非整个响应过程的传输时长 
- `client_body_buffer_size size`：用于接收每个客户端请求报文的body部分的缓冲区大小；默认为16k
- `client_body_temp_path   /var/tmp/client_body  1 2 2`：设定存储客户端请求报文的body部分的临时存储路径及子目录结构和数量，目录名为16进制的数字

### 对客户端进行限制的相关配置 

- `limit_rate rate; ` 限制响应给客户端的传输速率，单位是bytes/second  默认值0表示无限制  
- `limit_except method ... { ... };`：仅用于location  限制客户端使用除了指定的请求方法之外的其它方法  

### 文件操作优化的配置 

- `aio on | off | threads[=pool]; `：是否开启aio功能
- `directio size | off; `: 当文件大于等于给定大小时，例如directio 4m，同步（直接）写磁盘，而非写缓存 
- `open_file_cache off; `
- `open_file_cache max=N [inactive=time]; `
- `open_file_cache_errors on | off;`：是否缓存查找时发生错误的文件一类的信息，默认值为off
- `open_file_cache_min_uses number; `：open_file_cache指令的inactive参数指定的时长内，至少被命中此处指定 的次数方可被归类为活动项  默认值为1 
- `open_file_cache_valid time;`：缓存项有效性的检查频率  默认值为60s
  
  