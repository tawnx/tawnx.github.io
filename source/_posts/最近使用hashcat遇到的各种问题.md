---
title: 最近使用hashcat遇到的各种问题
date: 2022-3-22 17:28:38
tags: 
- Security
- Hashcat
categories: 
- 记录
---

# 最近使用hashcat遇到的各种问题

## 简介

Hashcat 是世界上最快的密码破解程序，是一个支持多平台、多算法的开源的分布式工具。

官网：https://hashcat.net/hashcat/

Github：https://github.com/hashcat/hashcat

## 安装

```
# 安装hashcat
brew install hashcat

# 查看版本
hashcat --version
```

macOS下使用homebrew安装hashcat后， 提示没有对应的文件夹

```
/Users/tawn/.local/share/hashcat/sessions/backend info.pid: No such file or directory
/Users/tawn/.local/share/hashcat: No such file or directory
```

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-20.57.13.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-20.57.13.png)

然后谷歌搜到用命令创建一个文件夹即可：https://github.com/hashcat/hashcat/issues/3044，但是实际操作下来提示权限不够（但是在我的MacBook Air上可以），然后我手动创建解决

接下来可以检测到设备，但是还是不能跑benchmark，提示缺少某个头文件(MacBook Air上没有这个问题)：

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-21.05.54.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-21.05.54.png)

这个链接：https://github.com/hashcat/hashcat/issues/3044下，也有提到这个问题，回答是：`停止使用 brew 并开始使用 github 版本`。再次经过[谷歌](https://github.com/hashcat/hashcat/issues/2270)，发现了一个临时解决办法，就是进入到homebrew安装的hashcat的opencl目录下` /usr/local/Cellar/hashcat/6.2.5/share/hashcat/OpenCL` 运行hashcat命令：

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-21.15.07.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-21.15.07.png)

但是目前还是不够完美，因为只能使用CPU，指定使用显卡的话会报错：`No devices found/left. `（windows下默认使用的就是显卡）。而且MacBook Air上根本无法跑，不指定设备都直接提示这个，好像需要用到专门的m1分支版本hashcat

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-21.19.25.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/截屏2022-03-22-21.19.25.png)

不知道是否可以通过升级OpenCL来解决？

再次寻找谷歌，看了很多，大概意思是Apple的OpenCL驱动不可靠，hashcat6.2.4（存疑？）及以上版本已经无法使用：https://forums.macrumors.com/threads/if-m1-pro-igpu-is-2x-faster-and-m1-max-4x-faster-than-m1-then-theyre-about-1-4-and-1-2-as-fast-as-3060-100w.2317863/

> 原文是这样描述的
>
> Apple's OpenCL driver is known to be buggy and unreliable, and this could just be an exact example of it. In fact hashcat does not even run as of 6.2.4 because it is not compatible with Apple's OpenCL driver. (you have to use an older version).
>
> I can still run hashcat 6.1.1 on macOS 11.6, but 6.2.4 no longer runs. Apple's OpenCL support is just broken after the deprecation announcement.

先这样吧，macOS下CPU勉强跑跑也还行，以后再来补充吧

## 多平台跑分测试

测试硬件：10850k+6600xt（黑苹果写完了这里可以补充一下）

macOS monterey下通过parallels desktop 17安装的kali linux 2022.1中的hashcat用10850k分配的两个核心：

```
┌──(root㉿kali)-[~]
└─# hashcat -b
hashcat (v6.2.5) starting in benchmark mode

Benchmarking uses hand-optimized kernel code by default.
You can use it in your cracking session by setting the -O option.
Note: Using optimized kernel code limits the maximum supported password length.                                                                           
To disable the optimized kernel code in benchmark mode, use the -w option.

OpenCL API (OpenCL 2.0 pocl 1.8  Linux, None+Asserts, RELOC, LLVM 11.1.0, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=====================================================================================================================================
* Device #1: pthread-Intel(R) Core(TM) i9-10850K CPU @ 3.60GHz, 1438/2941 MB (512 MB allocatable), 2MCU

Benchmark relevant options:
===========================
* --optimized-kernel-enable

-------------------
* Hash-Mode 0 (MD5)
-------------------

Speed.#1.........:   153.7 MH/s (3.24ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

----------------------
* Hash-Mode 100 (SHA1)
----------------------

Speed.#1.........:   107.9 MH/s (4.83ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------

Speed.#1.........: 61305.1 kH/s (8.31ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

---------------------------
* Hash-Mode 1700 (SHA2-512)
---------------------------

Speed.#1.........: 24136.9 kH/s (21.80ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

-------------------------------------------------------------
* Hash-Mode 22000 (WPA-PBKDF2-PMKID+EAPOL) [Iterations: 4095]
-------------------------------------------------------------

Speed.#1.........:     5157 H/s (15.32ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

-----------------------
* Hash-Mode 1000 (NTLM)
-----------------------

Speed.#1.........:   233.9 MH/s (2.11ms) @ Accel:256 Loops:1024 Thr:1 Vec:8
                                                                                                                            
---------------------                                                                                                       
* Hash-Mode 3000 (LM)                                                                                                       
---------------------                                                                                                       
                                                                                                                            
Speed.#1.........: 59242.7 kH/s (8.50ms) @ Accel:256 Loops:1024 Thr:1 Vec:8                                                 
                                                                                                                            
--------------------------------------------                                                                                
* Hash-Mode 5500 (NetNTLMv1 / NetNTLMv1+ESS)                                                                                
--------------------------------------------                                                                                
                                                                                                                            
Speed.#1.........:   170.8 MH/s (2.92ms) @ Accel:256 Loops:1024 Thr:1 Vec:8                                                 
                                                                                                                            
----------------------------                                                                                                
* Hash-Mode 5600 (NetNTLMv2)
----------------------------

Speed.#1.........: 21568.1 kH/s (24.18ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

--------------------------------------------------------
* Hash-Mode 1500 (descrypt, DES (Unix), Traditional DES)
--------------------------------------------------------

Speed.#1.........:  3180.8 kH/s (79.42ms) @ Accel:128 Loops:1024 Thr:1 Vec:8

------------------------------------------------------------------------------
* Hash-Mode 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5)) [Iterations: 1000]
------------------------------------------------------------------------------

Speed.#1.........:    11449 H/s (43.85ms) @ Accel:256 Loops:1000 Thr:1 Vec:8

----------------------------------------------------------------
* Hash-Mode 3200 (bcrypt $2*$, Blowfish (Unix)) [Iterations: 32]
----------------------------------------------------------------

Speed.#1.........:       15 H/s (2.14ms) @ Accel:2 Loops:32 Thr:1 Vec:1

--------------------------------------------------------------------
* Hash-Mode 1800 (sha512crypt $6$, SHA512 (Unix)) [Iterations: 5000]
--------------------------------------------------------------------

Speed.#1.........:     1601 H/s (31.62ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

--------------------------------------------------------
* Hash-Mode 7500 (Kerberos 5, etype 23, AS-REQ Pre-Auth)
--------------------------------------------------------

Speed.#1.........:  1594.0 kH/s (82.03ms) @ Accel:128 Loops:512 Thr:1 Vec:8

-------------------------------------------------
* Hash-Mode 13100 (Kerberos 5, etype 23, TGS-REP)
-------------------------------------------------

Speed.#1.........:  1595.7 kH/s (81.93ms) @ Accel:64 Loops:1024 Thr:1 Vec:8

---------------------------------------------------------------
* Hash-Mode 15300 (DPAPI masterkey file v1) [Iterations: 23999]
---------------------------------------------------------------

Speed.#1.........:     1333 H/s (15.79ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

---------------------------------------------------------------
* Hash-Mode 15900 (DPAPI masterkey file v2) [Iterations: 12899]
---------------------------------------------------------------

Speed.#1.........:      911 H/s (42.88ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

------------------------------------------------------------------
* Hash-Mode 7100 (macOS v10.8+ (PBKDF2-SHA512)) [Iterations: 1023]
------------------------------------------------------------------

Speed.#1.........:    10553 H/s (47.11ms) @ Accel:256 Loops:1023 Thr:1 Vec:4

---------------------------------------------
* Hash-Mode 11600 (7-Zip) [Iterations: 16384]
---------------------------------------------

Speed.#1.........:     1908 H/s (66.62ms) @ Accel:256 Loops:4096 Thr:1 Vec:8

------------------------------------------------
* Hash-Mode 12500 (RAR3-hp) [Iterations: 262144]
------------------------------------------------

Speed.#1.........:      280 H/s (57.00ms) @ Accel:128 Loops:16384 Thr:1 Vec:8

--------------------------------------------
* Hash-Mode 13000 (RAR5) [Iterations: 32799]
--------------------------------------------

Speed.#1.........:      834 H/s (18.46ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

-----------------------------------------------------------------------
* Hash-Mode 6211 (TrueCrypt RIPEMD160 + XTS 512 bit) [Iterations: 1999]
-----------------------------------------------------------------------

Speed.#1.........:     4799 H/s (51.89ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

-----------------------------------------------------------------------------------
* Hash-Mode 13400 (KeePass 1 (AES/Twofish) and KeePass 2 (AES)) [Iterations: 24569]
-----------------------------------------------------------------------------------

Speed.#1.........:      659 H/s (32.23ms) @ Accel:256 Loops:1024 Thr:1 Vec:8

----------------------------------------------------------------
* Hash-Mode 6800 (LastPass + LastPass sniffed) [Iterations: 499]
----------------------------------------------------------------

Speed.#1.........:    51554 H/s (8.51ms) @ Accel:256 Loops:499 Thr:1 Vec:8

--------------------------------------------------------------------
* Hash-Mode 11300 (Bitcoin/Litecoin wallet.dat) [Iterations: 200459]
--------------------------------------------------------------------

Speed.#1.........:      105 H/s (12.58ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

Started: Tue Mar 22 17:09:15 2022
Stopped: Tue Mar 22 17:17:02 2022
```

macOS下homebrew安装的hashcat用10850k（感觉散热不太行，也许有抑制性能发挥），感觉兼容性有问题，有些项目没跑直接跳过了：

```
hashcat (v6.2.5) starting in benchmark mode

Benchmarking uses hand-optimized kernel code by default.
You can use it in your cracking session by setting the -O option.
Note: Using optimized kernel code limits the maximum supported password length.
To disable the optimized kernel code in benchmark mode, use the -w option.

OpenCL API (OpenCL 1.2 (Feb 12 2022 01:56:27)) - Platform #1 [Apple]
====================================================================
* Device #1: Intel(R) Core(TM) i9-10850K CPU @ 3.60GHz, 16352/32768 MB (4096 MB allocatable), 20MCU
* Device #2: AMD Radeon RX 6600 XT Compute Engine, skipped

Benchmark relevant options:
===========================
* --optimized-kernel-enable

-------------------
* Hash-Mode 0 (MD5)
-------------------

Speed.#1.........:   981.3 MH/s (21.37ms) @ Accel:1024 Loops:1024 Thr:1 Vec:4

----------------------
* Hash-Mode 100 (SHA1)
----------------------

Speed.#1.........:   485.8 MH/s (42.99ms) @ Accel:1024 Loops:1024 Thr:1 Vec:4

---------------------------
* Hash-Mode 1400 (SHA2-256)
---------------------------

Speed.#1.........:   173.2 MH/s (60.39ms) @ Accel:1024 Loops:512 Thr:1 Vec:4

---------------------------
* Hash-Mode 1700 (SHA2-512)
---------------------------

Speed.#1.........: 58734.0 kH/s (89.09ms) @ Accel:256 Loops:1024 Thr:1 Vec:2

-------------------------------------------------------------
* Hash-Mode 22000 (WPA-PBKDF2-PMKID+EAPOL) [Iterations: 4095]
-------------------------------------------------------------

Speed.#1.........:    22846 H/s (55.69ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

-----------------------
* Hash-Mode 1000 (NTLM)
-----------------------

Speed.#1.........:  1649.4 MH/s (12.62ms) @ Accel:1024 Loops:1024 Thr:1 Vec:4

---------------------
* Hash-Mode 3000 (LM)
---------------------

* Device #1: Skipping (hash-mode 3000)
             This is due to a known CUDA/HIP/OpenCL runtime/driver issue (not a hashcat issue)
             You can use --force to override, but do not report related errors.

--------------------------------------------
* Hash-Mode 5500 (NetNTLMv1 / NetNTLMv1+ESS)
--------------------------------------------

Speed.#1.........:  1042.5 MH/s (19.98ms) @ Accel:1024 Loops:1024 Thr:1 Vec:4

----------------------------
* Hash-Mode 5600 (NetNTLMv2)
----------------------------

Speed.#1.........: 92023.8 kH/s (56.81ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

--------------------------------------------------------
* Hash-Mode 1500 (descrypt, DES (Unix), Traditional DES)
--------------------------------------------------------

* Device #1: Skipping (hash-mode 1500)
             This is due to a known CUDA/HIP/OpenCL runtime/driver issue (not a hashcat issue)
             You can use --force to override, but do not report related errors.

------------------------------------------------------------------------------
* Hash-Mode 500 (md5crypt, MD5 (Unix), Cisco-IOS $1$ (MD5)) [Iterations: 1000]
------------------------------------------------------------------------------

Speed.#1.........:   150.3 kH/s (66.88ms) @ Accel:512 Loops:1000 Thr:1 Vec:4

----------------------------------------------------------------
* Hash-Mode 3200 (bcrypt $2*$, Blowfish (Unix)) [Iterations: 32]
----------------------------------------------------------------

Speed.#1.........:     7030 H/s (2.58ms) @ Accel:20 Loops:32 Thr:1 Vec:1

--------------------------------------------------------------------
* Hash-Mode 1800 (sha512crypt $6$, SHA512 (Unix)) [Iterations: 5000]
--------------------------------------------------------------------

Speed.#1.........:     6605 H/s (30.80ms) @ Accel:1024 Loops:1024 Thr:1 Vec:2

--------------------------------------------------------
* Hash-Mode 7500 (Kerberos 5, etype 23, AS-REQ Pre-Auth)
--------------------------------------------------------

Speed.#1.........: 13755.2 kH/s (95.12ms) @ Accel:128 Loops:512 Thr:1 Vec:4

-------------------------------------------------
* Hash-Mode 13100 (Kerberos 5, etype 23, TGS-REP)
-------------------------------------------------

Speed.#1.........: 13697.0 kH/s (95.53ms) @ Accel:256 Loops:256 Thr:1 Vec:4

---------------------------------------------------------------
* Hash-Mode 15300 (DPAPI masterkey file v1) [Iterations: 23999]
---------------------------------------------------------------

Speed.#1.........:     3884 H/s (54.68ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

---------------------------------------------------------------
* Hash-Mode 15900 (DPAPI masterkey file v2) [Iterations: 12899]
---------------------------------------------------------------

Speed.#1.........:     1988 H/s (49.34ms) @ Accel:128 Loops:512 Thr:1 Vec:2

------------------------------------------------------------------
* Hash-Mode 7100 (macOS v10.8+ (PBKDF2-SHA512)) [Iterations: 1023]
------------------------------------------------------------------

Speed.#1.........:    24237 H/s (51.06ms) @ Accel:64 Loops:1023 Thr:1 Vec:2

---------------------------------------------
* Hash-Mode 11600 (7-Zip) [Iterations: 16384]
---------------------------------------------

Speed.#1.........:     7324 H/s (86.98ms) @ Accel:128 Loops:4096 Thr:1 Vec:4

------------------------------------------------
* Hash-Mode 12500 (RAR3-hp) [Iterations: 262144]
------------------------------------------------

Speed.#1.........:     1110 H/s (71.89ms) @ Accel:64 Loops:16384 Thr:1 Vec:4

--------------------------------------------
* Hash-Mode 13000 (RAR5) [Iterations: 32799]
--------------------------------------------

Speed.#1.........:     2187 H/s (70.79ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

-----------------------------------------------------------------------
* Hash-Mode 6211 (TrueCrypt RIPEMD160 + XTS 512 bit) [Iterations: 1999]
-----------------------------------------------------------------------

Speed.#1.........:    17844 H/s (70.99ms) @ Accel:256 Loops:512 Thr:1 Vec:4

-----------------------------------------------------------------------------------
* Hash-Mode 13400 (KeePass 1 (AES/Twofish) and KeePass 2 (AES)) [Iterations: 24569]
-----------------------------------------------------------------------------------

Speed.#1.........:     3974 H/s (53.53ms) @ Accel:256 Loops:1024 Thr:1 Vec:4

----------------------------------------------------------------
* Hash-Mode 6800 (LastPass + LastPass sniffed) [Iterations: 499]
----------------------------------------------------------------

Speed.#1.........:   141.6 kH/s (69.65ms) @ Accel:512 Loops:499 Thr:1 Vec:4

--------------------------------------------------------------------
* Hash-Mode 11300 (Bitcoin/Litecoin wallet.dat) [Iterations: 200459]
--------------------------------------------------------------------

Speed.#1.........:      284 H/s (18.40ms) @ Accel:1024 Loops:1024 Thr:1 Vec:2

Started: Tue Mar 22 21:38:19 2022
Stopped: Tue Mar 22 21:40:43 2022
```

win10中的hashcat用6600xt：

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/hashcat_6600xt_benchmark_1.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/hashcat_6600xt_benchmark_1.png)

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/hashcat_6600xt_benchmark_2.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/hashcat_6600xt_benchmark_2.png)

[![img](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/hashcat_6600xt_benchmark_3.png)](https://image-bed-0.oss-cn-hongkong.aliyuncs.com/2022-03/hashcat_6600xt_benchmark_3.png)

可以看到一个好的硬件还是很重要的，最高速度快了百倍以上，也许是多跑两位的复杂度。也就是原先在虚拟机中跑5位的时间，可以用显卡跑7位。

