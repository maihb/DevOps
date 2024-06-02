前提准备

压力测试

```sh
sudo apt install -y redis-tools 

# a1
clear; redis-benchmark -a b12345 -p 6379 -c 50 -n 10000

# 主机2
clear; redis-benchmark -h a1 -a b12345 -p 6379 -c 50 -n 10000

#实测局域网方式，效率更高（本机的 2 倍）， 因为客户端 cpu 占了一半消耗。
```