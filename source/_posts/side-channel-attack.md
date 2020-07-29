---
title: 旁路攻击：一道典型题的分析
categories:
	- CTF
tags:
	- PWN
date: 2020/7/29 10:12
---
分析 **side-channel attack** 的典型题 [CSAW CTF 2015 - wyvern](https://github.com/ctfs/write-ups-2015/tree/master/csaw-ctf-2015/reverse/wyvern-500)。

<!--more-->

## CSAW QUALS 2015: wyvern-500

**Category:** Reversing **Points:** 500 **Solves:** 96 **Description:**

> There's a dragon afoot, we need a hero. Give us the dragon's secret and we'll give you a flag.HINT: static is only 1 of 2 methods to RE. IDA torrent unnecessary
>
> <a href="https://github.com/ctfs/write-ups-2015/blob/master/csaw-ctf-2015/reverse/wyvern-500/wyvern_c85f1be480808a9da350faaa6104a19b"><span id="inline-blue">下载文件</span></a>

程序是 C++写的 64 位 ELF，运行时需要输入 secret 验证，然后输出验证结果。

```basic
+-----------------------+
|    Welcome Hero       |
+-----------------------+

[!] Quest: there is a dragon prowling the domain.
        brute strength and magic is our only hope. Test your skill.

Enter the dragon's secret: flag

[-] You have failed. The dragon's power, speed and intelligence was greater.
```

使用 IDA Pro 反编译，查看伪码，注意到关键语句：

```c
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_100);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_214);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_266);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_369);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_417);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_527);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_622);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_733);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_847);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_942);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1054);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1106);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1222);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1336);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1441);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1540);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1589);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1686);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1796);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1891);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_1996);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2112);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2165);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2260);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2336);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2412);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2498);
std::vector<int,std::allocator<int>>::push_back(&hero, &secret_2575);
v6 = std::string::length(v11) - 1LL != legend >> 2;
```

## 思路一：找规律

观察到 `secret_\d*` 字段中的数字每次相差不大，且 100 正好是 d（dragon 首字母） 的 ASCII 符号，所以猜测 secret 为数字差代表的 ASCII。（[这篇文章](https://ohaithe.re/post/129657401392/csaw-quals-2015-reversing-500-wyvern) 从伪码中直接读出了加密方法）

```c
a = [0,100,214,266,369,417,527,622,733,847,942,1054,1106,1222,1336,1441,1540,1589,1686,1796,1891,1996,2112,2165,2260,2336,2412,2498,2575]
for i in range(1,len(a)):
	print(chr(a[i]-a[i-1]),end="")
```

得到结果 dr4g0n_or_p4tric1an_it5_LLVM

## 思路二：side-channel attack

legend 的值为初始的 0x73 ，故结果长度为 legend >> 2 = 28，观察发现（或者直接假设）检查函数会将输入的字符一个个检测，一旦错误就直接退出，因此前几位正确的 secret 执行的指令更多。通过外部工具观察执行指令次数来破解 secret。

### Pintools

工具使用 Intel 的 [Pintools](https://software.intel.com/en-us/articles/pin-a-dynamic-binary-instrumentation-tool) 中 inscount0.so 模块，代码如下：

```c
import os
import re
flag = ''
chars = []
for i in range(47,58):
	chars.append(chr(i))
for i in range(65,91):
	chars.append(chr(i))
for i in range(97,123):
	chars.append(chr(i))
chars.append('@')
chars.append('_')
move = True
cur_max = 0
while(move):
	move = False
	for c in chars:
		s = flag + c
		s = s.ljust(28,'A') #补全 28 位
		cmd = os.popen("echo '%s' | ../../../../pin -t inscount0.so -- /root/pwn/wyvern;cat ./inscount.out" % s).read()
		cnt = int(re.search("Count ([0-9]*)", cmd).group(1)) #统计指令数量
		if c=='/':
			cur_max = max(cnt,cur_max)
		if cnt > cur_max:
			print(s)
			flag = flag + c
			cur_max = cnt
			move = True
			index = 0
			break
print("flag:")
print(flag)
```

（思路来自 [这篇文章](https://bruce30262.github.io/csaw-ctf-2015-wyvern/)）

## ltrace

strace 用来跟踪一个进程的系统调用或信号产生的情况，ltrace 用来跟踪进程调用库函数的情况。

思路和使用 Pin 工具一致，实现更加简洁，速度也更快（代码来自 [inaz2](https://gist.github.com/inaz2/1682e7254b1c7a2cf641)）。

```python
from subprocess import Popen, PIPE

secret_length = None

for i in xrange(40):
        p = Popen(['ltrace', './wyvern'], stdin=PIPE, stdout=PIPE, stderr=PIPE)
        line = 'A' * i + '\\n'
        stdout, stderr = p.communicate(line)
        num_lines = len(stderr.split('\\n'))
        if num_lines != 42:
                secret_length = i
                break

print "[+] secret length = %d" % secret_length

secret = bytearray('A' * i)
for i in xrange(secret_length):
        results = []
        for c in xrange(0x20, 0x7f):
                p = Popen(['ltrace', './wyvern'], stdin=PIPE, stdout=PIPE, stderr=PIPE)
                secret[i] = chr(c)
                line = str(secret) + '\\n'
                stdout, stderr = p.communicate(line)
                num_lines = len(stderr.split('\\n'))
                results.append((num_lines, secret[i]))
        results.sort(reverse=True)
        secret[i] = results[0][1]
        print "[+] secret = %s" % str(secret)
```

可以很快得到结果。

```bash
[+] secret length = 28
[+] secret = dAAAAAAAAAAAAAAAAAAAAAAAAAAA
[+] secret = drAAAAAAAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4AAAAAAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4gAAAAAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4g0AAAAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4g0nAAAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4g0n_AAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4g0n_oAAAAAAAAAAAAAAAAAAAA
[+] secret = dr4g0n_orAAAAAAAAAAAAAAAAAAA
[+] secret = dr4g0n_or_AAAAAAAAAAAAAAAAAA
[+] secret = dr4g0n_or_pAAAAAAAAAAAAAAAAA
[+] secret = dr4g0n_or_p4AAAAAAAAAAAAAAAA
[+] secret = dr4g0n_or_p4tAAAAAAAAAAAAAAA
[+] secret = dr4g0n_or_p4trAAAAAAAAAAAAAA
[+] secret = dr4g0n_or_p4triAAAAAAAAAAAAA
[+] secret = dr4g0n_or_p4tricAAAAAAAAAAAA
[+] secret = dr4g0n_or_p4tric1AAAAAAAAAAA
[+] secret = dr4g0n_or_p4tric1aAAAAAAAAAA
[+] secret = dr4g0n_or_p4tric1anAAAAAAAAA
[+] secret = dr4g0n_or_p4tric1an_AAAAAAAA
[+] secret = dr4g0n_or_p4tric1an_iAAAAAAA
[+] secret = dr4g0n_or_p4tric1an_itAAAAAA
[+] secret = dr4g0n_or_p4tric1an_it5AAAAA
[+] secret = dr4g0n_or_p4tric1an_it5_AAAA
[+] secret = dr4g0n_or_p4tric1an_it5_LAAA
[+] secret = dr4g0n_or_p4tric1an_it5_LLAA
[+] secret = dr4g0n_or_p4tric1an_it5_LLVA
[+] secret = dr4g0n_or_p4tric1an_it5_LLVM
```

最后将 secret 输入，得到 flag：

```bash
+-----------------------+
|    Welcome Hero       |
+-----------------------+

[!] Quest: there is a dragon prowling the domain.
        brute strength and magic is our only hope. Test your skill.

Enter the dragon's secret: dr4g0n_or_p4tric1an_it5_LLVM
success

[+] A great success! Here is a flag{dr4g0n_or_p4tric1an_it5_LLVM}
```
