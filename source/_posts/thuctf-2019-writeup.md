---
title: THUCTF 2019 - Writeup
tags:
  - CTF
url: thuctf2019.html
id: thuctf2019
categories:
  - CS
date: 2019-10-10 15:46:06
---

第一次参加CTF比赛，虽然水，但Writeup还是要写的。（不然没有奖金）

show_me_more

1.入域：一道快乐的送分题
-------------

![img](/images/thuctf2019001.png)

得到flag：THUCTF{Welcome_To_THUCTF2019}

早上7点起床，发现大佬[mcfx](https://mcfx.us/)已经提交了三道题，于是决定沿着他的路径做题。

![img](/images/thuctf2019002.png)

2.xor cipher
------------

Google一下，发现是异或密码，找到相关文章：[CTF题目中XOR加密解法脚本](https://blog.csdn.net/qq_41079177/article/details/89196428)和相关题目[问鼎杯 CTF writeup](https://chybeta.github.io/2017/09/16/问鼎杯-CTF-writeup/)，安装xortool(为了正常运行还装了ubuntu虚拟机)

![img](/images/thuctf2019001-3358223.png)

大约得到THUCTF{xor_es_eteres=&enq}，还需要进一步处理。

得到初步解密的结果，找到原文:[Wikiped – XOR cipher](https://en.wikipedia.org/wiki/XOR_cipher)

![img](/images/thuctf2019002-3358223.png)

现在已知明文和密文，只需求解密钥，由异或运算性质A⨁B=C⇒A⨁C=B或B⨁C=A，再对明文和密文进行一次异或运算即得flag。下面是加密/解密源码：

```python
#!/usr/bin/python

key = 'THUCTF{xo3_1s_1terestr1nq}' 
flag = ''
with open('cipher.txt') as f:
    con = f.read()
    for i in range(len(con)):
        flag += chr(ord(con[i]) ^ ord(key[i%26]))
f = open('flag.txt', 'w')
print(flag)
f.write(flag)
f.close()
```


3.ListenToMe
------------

文件只有一段音频，Google查找类似题，发现使用Audacity的频谱图观察，没看出什么东西。

然后去听原始音频，发现是摩尔斯电码，解密后得到：

![img](/images/thuctf2019001-3358253.png)

还是我去看Spectrogram频谱图…

又自己折腾了一会儿Audacity，发现要“缩放到合适大小“（为什么以往write up都不说 …(｡•ˇ‸ˇ•｡) … 还是我太水了吗），然后Flag就出来了。

![img](/images/thuctf2019002-3358253.png)THUCTF{Can_You_Hear_UlTRASOUND}

4.ICS Scavengers
----------------

分析文件格式，发现是一道流量分析题，用Wireshark打开分析，追踪TCP流，找到有意义的信息。

![img](/images/thuctf2019001-3358267.png)

第一个是Base64加密字符串，直接解密即可。

![img](/images/thuctf2019002-3358267.png)![img](/images/thuctf2019003.png)

第二个明文给出。

![img](/images/thuctf2019004.png)

第三个是Base64编码的图片，解码即可。

![img](/images/thuctf2019005.png)![img](/images/thuctf2019006.png)

三段Flag合成最终的Flag：

![img](/images/thuctf2019007.png)

5.后记
----

为什么我只写了3道题呢？

*   完全没有基础，100%萌新
*   后天被各种活动和微积分线代离散新生研讨课按在地上摩擦，无法拿出大量时间学Pwn或者Web，也没有时间想题
*   懒

这应该算是一个好的开头吧，我想。最近一直遇到的都是学习难度大、曲线陡的问题，都耗时间，但都要坚持。 **只要，一直提高就好了吧。**