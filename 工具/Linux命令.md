# 文件相关
## 查看文件夹下文件数量
```shell
ls -l | grep "^-" | wc -l
```
## 统计当前文件夹下目录的个数
```shell
ls -l | grep "^d"| wc -l
```

grep "^-"
这里将长列表输出信息过滤一部分，只保留一般文件，如果只保留目录就是 `^d`

wc -l 
统计输出信息的行数，因为已经过滤得只剩一般文件了，所以统计结果就是一般文件信息的行数，又由于一行信息对应一个文件，所以也就是文件的个数


# 网络相关

## netstat  用来显示网络状态

netstat -anp：显示系统端口使用情况  
netstat -nupl：UDP类型的端口  
netstat -ntpl：TCP类型的端口  
netstat -na|grep ESTABLISHED|wc -l：统计已连接上的，状态为"established"  
netstat -l：只显示所有监听端口  
netstat -lt：只显示所有监听**tcp**端口

## tcpdump  抓取所有网络包，

抓取所有的网络包，并存到 `result.cap` 文件中。
```text
tcpdump -w result.cap
```
03、抓取所有的经过`eth0`网卡的网络包，并存到`result.cap` 文件中。
```text
tcpdump -i eth0 -w result.cap
```
04、抓取源地址是`192.168.1.100`的包，并将结果保存到 `result.cap` 文件中。
```text
tcpdump src host 192.168.1.100 -w result.cap
```
05、抓取地址包含是`192.168.1.100`的包，并将结果保存到 `result.cap` 文件中。
```text
tcpdump host 192.168.1.100 -w result.cap
```