## [nginx 并发优化](https://www.cnblogs.com/yuanzai12345/p/5951860.html)

- nginx **配置文件**中对**优化**比较有作用的为以下几项

	- **`worker_processes 8;`**

		>nginx 进程数，建议按照cpu 数目来指定，一般为它的倍数 (如,2个四核的cpu计为8)。
		
	- **`worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;`**

		>为每个进程分配cpu，上例中将8 个进程分配到8 个cpu，当然可以写多个，或者将一个进程分配到多个cpu。
		
	- **`worker_rlimit_nofile 65535;`**

		>这个指令是指当一个nginx 进程打开的最多文件描述符数目，理论值应该是最多打开文件数（ulimit -n）与nginx 进程数相除，但是nginx 分配请求并不是那么均匀，所以最好与ulimit -n 的值保持一致。
		
		>现在在linux 2.6内核下开启文件打开数为65535，worker_rlimit_nofile就相应应该填写65535。
		
		>这是因为nginx调度时分配请求到进程并不是那么的均衡，所以假如填写10240，总并发量达到3-4万时就有进程可能超过10240了，这时会返回502错误。
		
			查看linux系统文件描述符的方法：
	
			[root@web001 ~]# sysctl -a | grep fs.file
			
			fs.file-max = 789972
			
			fs.file-nr = 510 0 789972
			
	- **use epoll**;
	
	 	>使用epoll 的I/O 模型 Epoll: 使用于Linux内核2.6版本及以后的系统。
	 
	- **`worker_connections 65535`**;

		>每个进程允许的最多连接数， 理论上每台nginx 服务器的最大连接数为worker_processes*worker_connections。	
		
	- **`keepalive_timeout 60`**;

		>keepalive 超时时间。
		
	- **`client_header_buffer_size 4k`**;

		>客户端请求头部的缓冲区大小，这个可以根据你的系统分页大小来设置，一般一个请求头的大小不会超过1k，不过由于一般系统分页都要大于1k，所以这里设置为分页大小。

		>分页大小可以用命令`getconf PAGESIZE` 取得。
		
	- **`open_file_cache max=65535 inactive=60s`**;

		>这个将为打开文件指定缓存，默认是没有启用的，max 指定缓存数量，建议和打开文件数一致，inactive 是指经过多长时间文件没被请求后删除缓存。
		
	- **`open_file_cache_valid 80s`**;

		>这个是指多长时间检查一次缓存的有效信息。
		
	- **`open_file_cache_min_uses 1`**;

		>open_file_cache 指令中的inactive 参数时间内文件的最少使用次数，如果超过这个数字，文件描述符一直是在缓存中打开的，如上例，如果有一个文件在inactive 时间内一次没被使用，它将被移除。
		
- **关于内核参数的优化**：

	- **`net.ipv4.tcp_max_tw_buckets = 6000`**

		>timewait 的数量，默认是180000。

	- **`net.ipv4.ip_local_port_range = 1024 65000`**

		>允许系统打开的端口范围。

	- **`net.ipv4.tcp_tw_recycle = 1`**

		>启用timewait 快速回收。

	- **`net.ipv4.tcp_tw_reuse = 1`**

		>开启重用。允许将TIME-WAIT sockets 重新用于新的TCP 连接。

	- **`net.ipv4.tcp_syncookies = 1`**

		>开启SYN Cookies，当出现SYN 等待队列溢出时，启用cookies 来处理。

	- **`net.core.somaxconn = 262144`**

		>web 应用中listen 函数的backlog 默认会给我们内核参数的net.core.somaxconn 限制到128，而nginx 定义的NGX_LISTEN_BACKLOG 默认为511，所以有必要调整这个值。

	- **`net.core.netdev_max_backlog = 262144`**

		>每个网络接口接收数据包的速率比内核处理这些包的速率快时，允许送到队列的数据包的最大数目。

	- **`net.ipv4.tcp_max_orphans = 262144`**

		>系统中最多有多少个TCP 套接字不被关联到任何一个用户文件句柄上。如果超过这个数字，孤儿连接将即刻被复位并打印出警告信息。这个限制仅仅是为了防止简单的DoS 攻击，不能过分依靠它或者人为地减小这个值，更应该增加这个值(如果增加了内存之后)。

	- **`net.ipv4.tcp_max_syn_backlog = 262144`**

		>记录的那些尚未收到客户端确认信息的连接请求的最大值。对于有128M 内存的系统而言，缺省值是1024，小内存的系统则是128。

	- **`net.ipv4.tcp_timestamps = 0`**

		>时间戳可以避免序列号的卷绕。一个1Gbps 的链路肯定会遇到以前用过的序列号。时间戳能够让内核接受这种“异常”的数据包。这里需要将其关掉。

	- **`net.ipv4.tcp_synack_retries = 1`**

		>为了打开对端的连接，内核需要发送一个SYN 并附带一个回应前面一个SYN 的ACK。也就是所谓三次握手中的第二次握手。这个设置决定了内核放弃连接之前发送SYN+ACK 包的数量。

	- **`net.ipv4.tcp_syn_retries = 1`**

		>在内核放弃建立连接之前发送SYN 包的数量。

	- **`net.ipv4.tcp_fin_timeout = 1`**

		>如果套接字由本端要求关闭，这个参数决定了它保持在FIN-WAIT-2 状态的时间。对端可以出错并永远不关闭连接，甚至意外当机。缺省值是60 秒。2.2 内核的通常值是180 秒，3你可以按这个设置，但要记住的是，即使你的机器是一个轻载的WEB 服务器，也有因为大量的死套接字而内存溢出的风险，FIN- WAIT-2 的危险性比FIN-WAIT-1 要小，因为它最多只能吃掉1.5K 内存，但是它们的生存期长些。

	- **`net.ipv4.tcp_keepalive_time = 30`**

		>当keepalive 起用的时候，TCP 发送keepalive 消息的频度。缺省是2 小时

