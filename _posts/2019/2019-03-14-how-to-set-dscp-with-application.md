---
layout: post
title: 应用如何设置DSCP
categories:
- 编程
tags:
- 编程
---

# DSCP是什么

DSCP：差分服务代码点（Differentiated Services Code Point），IETF于1998年12月发布了Diff-Serv（Differentiated Service）的QoS分类标准。
它在每个数据包IP头部的服务类别TOS标识字节中，利用已使用的6比特和未使用的2比特，通过编码值来区分优先级。DSCP 使用6个bit，DSCP的值得范围为0~63。
换言之它为QOS提供了比TOS标记更多的优先级范围。

# DSCP优先级

RFC 791中 OS位的IP Precedence划分成了8个优先级，可以应用于流分类，数值越大表示优先级越高。

```bash
   0     1     2     3     4     5     6     7  
+-----+-----+-----+-----+-----+-----+-----+-----+
|   PRECEDENCE    |  t3 | t2  |  t1 | t0 |m
-----+-----+-----+-----+-----+-----+-----+-----+
            111 - Network Control
            110 - Internetwork Control
            101 - CRITIC/ECP
            100 - Flash Override
            011 - Flash
            010 - Immediate
            001 - Priority
            000 – Routine
```

但是在网络中实际部署的时候这8个优先级是远远不够的，于是在RFC 2474中又对TOS进行了重新的定义。把前六位定义成DSCP，后两位保留。

```bash
 0   1   2   3   4   5   6   7
+---+---+---+---+---+---+---+---+
|         DSCP          |  CU   |
+---+---+---+---+---+---+---+---+
DSCP: differentiated services codepoin
CU:   currently unused 
```

但是由于DSCP和IP PRECEDENCE是共存的于是存在了一些兼容性的问题，DSCP的可读性比较差，比如DSCP 43我们并不知道对应着IP PRECEDENCE的什么取值，
于是就把DSCP进行了进一步的分类。DSCP总共分成了4类。

```bash
                 Class Selector(CS)           aaa 000
                 Expedited Forwarding(EF)     101 110
                 Assured Forwarding(AF)       aaa bb0
                 Default(BE)                  000 000
```

1，默认的DSCP为000 000  
2，CS的DSCP后三位为0，也就是说CS仍然沿用了IP PRECEDENCE只不过CS定义的DSCP=IP PRECEDENCE*8，比如CS6=6*8=48，CS7=7*8=56  
3，EF含义为加速转发，也可以看作为IP PRECEDENCE为5，是一个比较高的优先级，取值为101110(46)，但是RFC并没有定义为什么EF的取值为46。  
4，AF分为两部分，a部分和b部分，a部分为3 bit仍然可以和IP PRECEDENCE对应，b部分为2 bit表示丢弃性，可以表示3个丢弃优先级，可以应用于RED或者WRED。
目前a部分由于有三个bit最大取值为8，但是目前只用到了1~4。为了迅速 的和10进制转换，可以用如下方法，先把10进制数值除8得到的整数就是AF值，
余数换算成二进制看前两位就是丢弃优先级，比如34/8=4余数为2，2 换算成二进制为010，那么换算以后可以知道34代表AF4丢弃优先级为middle的数据报。  

如果把CS EF AF和BE做一个排列可以发现一个有趣的现象，如下表。这个表也就是我们在现实当中应用最多的队列。根据IP PRECEDENCE的优先级，CS7最高依次排列BE最低。  
一般情况下这些队列的用途看这个表的Usage字段   

```bash
对应的服务 IPv4优先级/EXP/802.1P    DSCP(二进制) DSCP[dec][Hex] TOS(十六进制)       应用  丢包率  

BE         0                        0            0　　　　　   0                Internet  
AF1        Green 1       001 010      10[0x0a]      40[0x28]      Leased Line  L  
AF1        Green 1       001 100      12[0x0c]      48[0x30]      Leased Line   M
AF1        Green 1       001 110      14[0x0e]      56[0x38]      Leased Line   H
AF2        Green 2                  010 010      18[0x12]      72[0x48]       IPTV VOD   L
AF2        Green 2                  010 100      20[0x14]      80[0x50]          IPTV VOD   M
AF2        Green 2                  010 110      22[0x16]      88[0x58]       IPTV VOD   H
AF3        Green 3                  011 010      26[0x1a]      104[0x68]       IPTV Broadcast  L
AF3        Green 3                  011 100      28[0x1c]      112[0x70]      IPTV Broadcast  M
AF3        Green 3                  011 110      30[0x1e]      120[0x78]       IPTV Broadcast  H
AF4        Green 4                  100 010      34[0x22]      136[0x88]       NGN/3G Singaling  L
AF4        Green 4                  100 100      36[0x24]      144[0x90]      NGN/3G Singaling M 
AF4        Green 4                  100 110      38[0x26]      152[0x98]       NGN/3G Singaling H
EF         5                        101 110      46[0x2E]      184[0xB8]      NGN/3G voice 
CS6(INC)   6                        110 000      48[0x30]      192[0xC0]      Protocol 
CS7(NC)    7                        111 000      56[0x38]      224[0xE0]      Protocol 

```
1，CS6和CS7默认用于协议报文，比如说OSPF报文，BGP报文等应该优先保障，因为如果这些报文无法接收的话会引起协议中断。而且是大多数厂商硬件队列里最高优先级的报文。  
2，EF用于承载语音的流量，因为语音要求低延迟，低抖动，低丢包率，是仅次于协议报文的最重要的报文。  
3，AF4用来承载语音的信令流量，这里大家可能会有疑问为什么这里语音要优先于信令呢？其实是这样的，这里的信令是电话的呼叫控制，你是可以忍受在接通的时候等待几秒钟的，但是绝对不能允许在通话的时候的中断。所以语音要优先于信令。   
4，AF3可以用来承载IPTV的直播流量，直播的时时性很强需要连续性和大吞吐量的保证。  
5，AF4可以用来承载VOD的流量，相对于直播VOD要求时时性不是很强，允许有延迟或者缓冲。  
6，AF5可以承载不是很重要的专线业务，因为专线业务相对于IPTV和VOICE来讲，IPTV和VOICE是运营商最关键的业务，需要最优先来保证。当然面向银行之类需要钻石级保证的业务来讲，可以安排为AF4甚至为EF。  
7，最不重要的业务是INTERNET业务，可以放在BE模型来传输。  
而在硬件队列里是如何保证协议报文（CS6和CS7中的数据）优先传输呢？在制作路由器的时候一般都是把CS6和CS7中的数据做PQ也就是绝对优先处理，  
无论下面是否有数据也是要优先来传递这两个队列中的数据。而其他EF到AF1的队列中是用WFQ来做的，保证所有队列都可以得到带宽来传输。  

fwd: https://blog.csdn.net/wang_hufeng/article/details/40514989
