---
title: tcpdump抓包
date: 2022-07-29 22:17:18
categories:
- tcp
tags:
- tcp
- socket
- linux
---

# 抓包进阶

## 抓包命令

```shell
tcpdump -i eth1 -vvv -w test.pcap
# -i 指定网口
# -vvv 在shell中显示抓到的包
# -w 将抓到的包写入文件
```

- 指定主机名

  ```shell
  tcpdump -i eth1 host www.baidu.com
  ```

- 指定ip

  ```shell
   tcpdump -i eth1 host 192.168.1.12
  ```

- 指定端口

  ```shell
   tcpdump -i eth1 port 80
   # 指定源端口
   tcpdump -i eth1 src port 80
   # 指定目的端口
   tcpdump -i eth1 dst port 80
  ```

- 指定协议

  ```shell
   # 对于传输层之上的协议需加port
   tcpdump -i eth1 port http
  ```

- 指定icmp

  ```shell
   # ping打包
   ping -s 4800 ip
   # 抓取ping包
   # 传输层及传输层之下的协议直接跟协议名
   tcpdump -i eth1 icmp
  ```

- 当需要长时间抓包时，为避免单个抓包文件过大导致打开查询缓慢，可设置当个抓包文件大小

  ```shell
  # 指定单个抓包文件大小为100M，超过100M会保存在新文件中
  tcpdump -i eth1 -C 100M -w test.pcap
  ```

## 拆分包

当单个文件报文个数过多时，打开和查询报文缓慢。可以使用wireshark相关工具对其进行拆分。

进入wireshark的安装目录，可以看到有一个editcap的程序。

```shell
# -c 指定多少个包拆分为一个文件， src待拆分的包文件路径及文件名，dst是拆分后的文件路径及文件名
editcap.exe -c 200000 src dst
```

## 合并包

Mergecap 从名字上 可以看出该命令的功能是合并多个报文为一个报文。

```shell
C:\Program Files\Wireshark>mergecap -h
Mergecap (Wireshark) 3.4.2 (v3.4.2-0-ga889cf1b1bf9)
Merge two or more capture files into one.
See https://www.wireshark.org for more information.

Usage: mergecap [options] -w <outfile>|- <infile> [<infile> ...]

Output:
  -a                concatenate rather than merge files.
                    default is to merge based on frame timestamps.
  -s <snaplen>      truncate packets to <snaplen> bytes of data.
  -w <outfile>|-    set the output filename to <outfile> or '-' for stdout.
  -F <capture type> set the output file type; default is pcapng.
                    an empty "-F" option will list the file types.
  -I <IDB merge mode> set the merge mode for Interface Description Blocks; default is 'all'.
                    an empty "-I" option will list the merge modes.

Miscellaneous:
  -h                display this help and exit.
  -v                verbose output.
```

```shell
mergecap -v -a -F pcap -w test.pcap
mergecap -v -a userip-1.pcap userip-2.pcap -F pcap -w merge_out.pcap
    -v参数表示打印每一片报文的编号，报文较多的话会打屏。
    -a参数表示的是按照文件中顺序将报文进行合并，默认情况是按照时间戳的顺序进行合并。由于每一片报文头部都是由有时间戳信息的，不明白的可以查看这里。
    -F表示文件的存储格式，例如pcap,pcapng等等。在-F后面参数为空的情况下，会列出该命令支持的所有文件格式，对照选择即可。
    -w即文件输出的
```

## 合并一定比例的包

使用python脚本构造固定比例的包，如1:1024等。

```python
import os

base_file = "5gc.pcap"
arp_file = "arp_request.pcap"
ratio = 0
dst_file = ""
pwd = os.curdir
for num in range(2,20):
    ratio = pow(2,num)
    copy_file = "5gc_copy_" + str(num) + ".pcap"
    os.system('copy %s %s' % (base_file, copy_file))
    dst_file = "5gc_" + str(ratio) + ".pcap"
    os.system("mergecap.exe -v -a %s %s -F pcap -w %s" % (base_file, copy_file, dst_file))
    base_file = dst_file
    dst_arp_file = "5gc_arp_1_" + str(ratio) + ".pcap"
    os.system("mergecap.exe -v -a %s %s -F pcap -w %s" % (base_file, arp_file, dst_arp_file))
```

## 查询包

一般获取抓包文件后，需要使用wireshark对包进行查询和分析。常见的查询命令如下：

```shell
tcp # 过滤tcp报文
ip.addr == 127.0.0.1 # 根据ip地址过滤
tcp.flags.reset == 1 # 过滤reset报文
tcp.port == 8080 # 根据端口过滤
tcp.len > 0 # 根据tcp数据长度过滤
```



# reference

1.https://www.freesion.com/article/9386244731/