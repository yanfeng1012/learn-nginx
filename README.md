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
		
3. nginx 反向代理

>反向代理（Reverse Proxy）方式是指以代理服务器来接受Internet上的连接请求，然后将请求转发给内部网络上的服务器；并将从服务器上得到的结果返回给Internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器

- eg:

		upstream tornado {
		    server 127.0.0.1:8888;
		}
		  
		server {
			listen   80;
			root /root/nmapp2_venv;
			index index.php index.html;
				  
			server_name server;
			  
			location ~ /index {
				proxy_pass_header Server;
				proxy_set_header Host $http_host;
				proxy_set_header X-Real-IP $remote_addr;
				proxy_set_header X-Scheme $scheme;
				proxy_pass http://tornado;
			}
		}
	
		
	
