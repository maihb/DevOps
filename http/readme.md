
## 适合大量并发的系统参数：   
```sh
#cat /etc/sysctl.conf

net.ipv4.tcp_max_tw_buckets = 500
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_max_syn_backlog = 20000
net.ipv4.tcp_synack_retries = 2
kernel.sysrq=1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_tw_recycle = 0  #1的话，客户端NAT后连接失败，而且建立连接超时，高并发时候
net.ipv4.tcp_fin_timeout = 15
net.core.somaxconn = 20000
net.ipv4.ip_local_port_range = 55001 65000
net.core.rmem_max = 26214400
net.core.wmem_max = 26214400
#sudo sysctl -p
```

**验证上述参数**

先安装wrk
go get github.com/adeven/go-wrk

➜ go-wrk -t=8 -c=100 -n=200000 -k http://10.10.0.2:8086/api/v1/posts
==========================BENCHMARK==========================
URL:				http://10.10.0.2:8086/api/v1/posts

Used Connections:		100
Used Threads:			8
Total number of calls:		200000

===========================TIMINGS===========================
Total time passed:		862.78s
Avg time per request:		419.31ms
Requests per second:		231.81
Median time per request:	8.69ms
99th percentile time:		13149.77ms
Slowest time for request:	26012.00ms

=============================DATA=============================
Total response body sizes:		0
Avg response body per request:		0.00 Byte
Transfer rate per second:		0.00 Byte/s (0.00 MByte/s)
==========================RESPONSES==========================
20X Responses:		198407	(99.20%)
30X Responses:		0	(0.00%)
40X Responses:		0	(0.00%)
50X Responses:		0	(0.00%)
Errors:			1593	(0.80%)

## 测试浏览器跨域问题步骤

跨域测试方法：
浏览器随便打开一个网页，
然后打开调试，选控制台，复制粘贴下面代码：

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://api.iot.xa.com/v5/clients/wx-MUFBQWZVdw/devices?timestamp=1593654023&client_id=avbEnPqq3lw&nonce=1593654023&signature=06a4c41f04b63326cd5c031e3f18f9b50c6ff131');
xhr.setRequestHeader("session","C8pONvqryXX5Me7jm60nAE5vWCTD69rF-u_WVo7j8gw");
xhr.setRequestHeader("content-type","application/json; charset=UTF-8");
xhr.send({"account":3});
xhr.onload = function(e) {
var xhr = e.target;
console.log(xhr.responseText);
}
```

## for循环测试网页访问 

```sh
tmp=$1
min=1
num=10
if [ $tmp -ge $min ];then
	num=$tmp
fi
sum=0
t=5
for ((i=0;i<$num;i++))
do
	startTime=`date +%s`
	curl baidu.com
	endTime=`date +%s`
	costTime=$[$endTime-$startTime]
	if [ $costTime -ge $t ];then
		let sum++
	fi
	usleep 2000
done
	echo "总共测试测试:$num,大于1秒次数:$sum"
```