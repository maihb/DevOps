# redis 生产问题集合

## 安装redis
apt install redis -y
service redis stop

## 因 monitor 引起的 redis内存占用飙升

模拟实验：
1.  开启一个空的Redis(最简，直接redis-server)
```sh
$ redis-server 
$ redis-cli  info|grep memory
used_memory:858992
used_memory_human:838.86K
used_memory_rss:7823360
used_memory_rss_human:7.46M
used_memory_peak:858992
used_memory_peak_human:838.86K
used_memory_peak_perc:100.00%
used_memory_overhead:845766
used_memory_startup:796072
used_memory_dataset:13226
total_system_memory_human:3.82G

$ redis-cli  info|grep client
connected_clients:1
client_recent_max_input_buffer:2
client_recent_max_output_buffer:0
blocked_clients:0
mem_clients_slaves:0
mem_clients_normal:49694
```
 
2. 开启一个monitor：
```sh
redis-cli -h 127.0.0.1 -p 6379 monitor
```

3. 使用redis-benchmark：
```sh
redis-benchmark -h 127.0.0.1 -p 6379 -c 500 -n 200000
```
4. 观察
(1) info memory：内存一直增加，直到benchmark结束，monitor输出完毕，但是used_memory_peak_human（历史峰值）依然很高--观察附件中日志
(2)info clients: client_longest_output_list： 一直在增加，直到benchmark结束，monitor输出完毕，才变为0 --观察附件中日志
(3)redis-cli -h host -p port client list | grep "monitor" omem一直很高，直到benchmark结束，monitor输出完毕，才变为0 --观察附件中日志
监控脚本：

```sh
while [ 1 == 1 ]
do
now=$(date "+%Y-%m-%d_%H:%M:%S")
echo "=========================${now}==============================="
echo " #Client-Monitor"
redis-cli -h 127.0.0.1 -p 6379 client list | grep monitor
redis-cli -h 127.0.0.1 -p 6379 info clients
redis-cli -h 127.0.0.1 -p 6379 info memory
#休息100毫秒
usleep 100000
done
```
