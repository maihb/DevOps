
## nginx 启动失败，报端口没权限:
```sh
semanage port -l | grep http_port_t      --查看能用哪些端口

Nginx：转发tcp设置：
解决方法：yum install nginx-mod-stream -y 即可
stream {
    server {
        listen 8701;
        proxy_connect_timeout 5s;
        proxy_timeout 20s;
    proxy_pass 127.0.0.1:3306;
    }
}

Nginx 报错：
2021/09/02 16:17:53 [crit] 4073 #4073: *15 connect() to 127.0.0.1:3306 failed (13: Permission denied) while connecting to upstream, client: 127.0.0.1, server: 0.0.0.0:8009, upstream: "127.0.0.1:3306", bytes from/to client:0/0, bytes from/to upstream:0/0

解决办法：
vim /etc/selinux/config
找到：
SELINUX=enforcing  改为： SELINUX=disabled
执行下面的命令（不用重启）　　　　　　
setsebool -P httpd_can_network_connect 1

```

## Nginx tcp转发

```
stream {
    upstream svr{
        server 138.2.42.30:443 weight=1;
    }
    server {
        listen 443;
        proxy_connect_timeout 5s;
        proxy_timeout 130s;
        proxy_pass svr;
    }
}

#静态文件服务， 挂在http对象内（过时了， 换 miniserve ）
	server {
		listen     8091;
		location / {
			alias /upload;
			autoindex on;
		}
	}
```