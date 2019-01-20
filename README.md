# learn-nginx

1. nginx 的安装部署

	- Nginx 的启停

		`/usr/local/nginx/sbin/Nginx -g TERM | INT | QUIT | HUP [-c newConFile]`<br>
		其中TERM 和 INT 信号用于快速停止，QUIT 用于平缓停止，HUP平滑重启。
		
2. nginx conf

		...                 #全局块
		events              #events块
		{
			...
		
		}
		http                # http块
		{
			...             # http 全局块
		}
		server              # server块
		{
			...
			location [PATTERN]  #location块
			{
				...
			}
			location [PATTERN]
			{
				...
			}
		}
		server              # server块
		{
			...
		}
		...                 # http 全局块
		

	
		
	
