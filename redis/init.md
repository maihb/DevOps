一行命令：    
```sh
docker run --name rds -p 6379:6379 -d redis:7.0 --requirepass b12345
```

配置 host，方便后续测试

```sh
# 主机1
hostnamectl set-hostname a1
echo  172.26.15.54 a2 >> /etc/hosts
# 主机2
hostnamectl set-hostname a2
echo  172.26.8.158 a1 >> /etc/hosts
```