---
title: "数据结构与算法（Python版）之概述"
date: 2023-03-11T10:47:55+08:00
draft: true
Tags : ["Python", "算法"]
Categories : ["算法"]
---

# 数据结构与算法（Python版）

课程地址：<https://www.icourse163.org/learn/PKU-1206307812?tid=1470091590#/learn/content?type=detail&id=1253445962>

# 一、概述

## 101 引子：数据时代

### 数据结构与算法是讲什么的课？

- 首先，这是一门关于编程的课
  - 学了Python语言基础之后的进阶课
- 但又不仅仅讲编程
- 本课为你展示
  - 如何把数据组织起来
  - 进行有效的处理
  - 以解决问题
- 数据为什么这么重要？

无处不在的数据

### 信息时代就是数据的时代

- 人类在各个领域的生产生活
- 无时无刻在产生着巨量的数据

### 数据化的科学、技术、工程、商业

- 地球观测系统：地面、地下、空中
- 地球观测系统：人造地球卫星
- 地球之外的观测系统：火星车
- 地球之外的观测系统：太阳探测器
- 地球之外的观测系统：冥王星探测器
- 地球之外的观测系统：哈勃望远镜
- 地球重力场分布数据
- 地形高程和地貌数据
- 地下岩层结构和矿藏数据
- 太阳风和空间气象数据
- 高分辨率卫星影像数据
- 实时交通流量、路况数据
- 引力波探测的突破（2015.9.14）

### 数据主义（Dataism）

- 整个世界就是数据及其算法
- 尤瓦尔·赫拉利：《未来简史》
  - 将生命活动理解为数据流传输及处理算法，人类智慧和自由意识也无法例外

### 数据之桥，通向宇宙

- 超时空接触.Contact.1997  (电影)



## 102 问题求解的计算之道

### 问题：未知的事物

- 人们在生活、生产、学习、探索、创造过程中会遇到各种未知的事物(what why how)
  - 云是什么？
  - 这种草（虫子）可以吃么？
  - 什么是无理数？
  - 什么事万物的起源？
  - 为什么会下雨？
  - 为什么事物放久了会发霉？
  - 为什么✔️2是无理数？
  - 生命的意义是什么？
  - 怎么让粮食长得更多？
  - 怎么将楼房建到101层？
  - 怎么求最大公约数？
  - 怎么维护公平与正义？

### 问题解决之道

- 如何从未知到已知
  - 感觉、经验、占卜、求神、逻辑、数学、实验、工程、计算模型、模拟、仿真哲学？
  - 神农尝百草、部落战争占卜和求神、自然科学工程
- 有些问题已经解决
- 很多问题尚未解决
- 有些问题似乎无法完全解决
- 尚未解决和无法解决问题的共性
  - 表述含混、标准不一、涉及主观、结果不确定

### 数学：解决问题的终极工具

- 在长期的发展过程中，人们把已经解决的问题逐渐表述为属性命题与模型
  - 尚未解决的问题，人们试图通过数学建模，采用数学工具来解决
  - 无法解决的问题，人们试图转换表述、明晰问题来部分解决

### 为什么是数学？

- 数学具有清晰明确的符号表述体系
- 严密确定的推理系统
- 但正如科学不是万能的，数学也不是万能的
  - 有些问题天然无法明确表述（主观、价值观、意识形态、哲学问题等）
  - 有些可明确表述的问题仍然无法解决
  - 伽利略：数学是上帝书写宇宙的文字
  - Donald in Mathmagic Land.1959  （数学科普片）

### 问题解决的“计算”之道

- 20世纪20年代，为了解决数学本身的可检验性问题，大数学家希尔伯特提出“能否找到一种基于有穷观点的能行方法，来判定任何一个数学命题的真假”

### 抽象的“计算”概念提出

- 基于有穷观点的能行方法
  - 由有限数量的明确有限指令构成
  - 指令执行在有限步骤后终止
  - 指令每次执行都总能得到唯一结果
  - 原则上可以由人单独采用纸笔完成，而不依靠其他辅助
  - 每条指令可以机械地被精确执行，而不需要智慧和灵感

### 关于“计算”的数学模型

- 20世纪30年代，几位逻辑学家各自独立提出了几个关于“计算”的属性模型
  - 哥德尔和克莱尼的递归函数模型
  - 丘奇的Lambda演算模型
  - 波斯特的Post机模型
  - 图灵的图灵机模型
- 研究证明，这几个“基于有穷观点的能行方法”的计算模型，全都是等价的
- 虽然希尔伯特的计划最终被证明无法实现
  - 不存在“能行方法”可判定所有数学命题的真假
  - 总有数学命题，其真假是无法证明的
- 但“能行可计算”概念成为计算理论的基础
  - 其中的一些数学模型（如图灵机）也成为现代计算机的理论基础
  - 计算机是数学家一次失败思考的产物。 ——无名氏



## 103 图灵机计算模型

### 图灵机 Turing Machine

- 1936年，Alan Turing提出的一种抽象计算模型
  - 基本思想是用机器模拟人们用纸笔进行数学运算的过程，但比数值计算更为简单

### 图灵机 Turing Machine 基本概念

- 在纸上写上或擦除某个符号
- 把注意力从纸的一个位置转向另一个位置
- 在每个阶段，要决定下一步动作依赖于：
  - 此人当前所关注的纸上某个位置的符号和
  - 此人当前思维的状态
- The.Imitation.Game.2014 (电影)
- 图灵机由以下几部分构成
  - 一条无限长的分格纸带，每格可以记录1个符号
  - 一个读写头，可在纸带上左右移动，能读出和擦写格子的字符
  - 一个状态寄存器，记录有限状态中的1个状态
  - 一系列有限的控制规则：（5个）
    - 某个状态，读入某个字符时
    - 要改写成什么字符
    - 要如何移动读写头
    - 要改变为什么状态

### 看一个图灵机例子

- 判定 {a^mb^m|m>=0} ：左半部全是a，右半部全是b，且ab数量相等的字符串
  - 如：ab、aabb、aaaabbbb，进入“接受”状态
  - 如：b、ba、abb，进入“拒绝”状态
- 规则思路：读写头来回移，将a和b一一对消，如果最后剩下空白B则接受，否则拒绝
  - 初始状态s0是读写头停在第一个字符处
  - s1状态是读写头正在右移
  - s2状态是读写头道字符串最右边
  - s3状态是读写头正在向回左移

图灵机模拟软件 Visual Turing



## 104 算法和计算复杂性

### 问题的分类

- What：是什么？
  - 面向判断与分类的问题
- Why：为什么？
  - 面向求因与证明的问题
- How：怎么做？
  - 面向过程与构建的问题

### 可以通过“计算”解决的问题

- 用任何一个“有限能行方法”下的计算模型可以解决的问题，都算是“可计算”的
- What：分类问题
  - 可以通过树状的判定分支解决
- Why：证明问题
  - 可以通过有限的公式序列来解决
  - 数学定理证明从不证自明的公理出发，一步步推理得出最后待证明的定理，我们在以往学习过的定理证明即为此类解决方法
- How：过程问题
  - 可以通过算法流程来解决
  - 解决问题的过程：算法和相应数据结构的研究，即为本课主要内容

### 世界上最早的算法：欧几里德算法

- 公元前3世纪，记载于《几何原本》
  - 辗转相除法求最大公约数
- 辗转相除法处理大数时非常高效
- 它需要的步骤不会超过小数位数的5倍
  - 加百利·拉梅(Gabriel Lame)于1844年证明了这个结论，并开创了计算复杂性理论

### 计算复杂性

- “基于有穷观点的能行方法”的“可计算”概念
  - 仅仅涉及到问题的解决是否能在有限资源内（时间/空间）完成
  - 并不关心具体要花费多少计算步骤或多少存储空间
- 由于资源（时间/空间）相当有限，对于问题的解决需要考虑其可行性如何
- 人们发现各种不同的可计算问题，其难易程度是不一样的
  - 有些问题非常容易解决，如基本数值计算
  - 有些问题的解决程度尚能令人满意，如表达式求值、排序等
  - 有些问题的解决会爆炸性地吞噬资源，虽有解法，但没什么可行性，如哈密顿回路、货郎担问题等
- 定义一些衡量指标，对问题的难易程度（所需的执行步骤数/存储空间大小）进行分类，是计算复杂性理论的研究范围
- 但对于同一个问题，也会有不同的解决方案，其解决效率上也是千差万别
- 如排序问题，以n张扑克牌作为排序对象
  - 一般人们会想到的是“冒泡“排序，即每次从牌堆里选出一张最小的牌，这样全部排完大概会需要n^2量级的比较次数
- 另一种有趣的”Bogo“排序方法
  - 洗一次牌，看是否排好序，没有的话，接着洗牌，直到排序成功！这样全部排完，平均需要n^*n!量级的比较次数

### 计算复杂性与算法

- 计算复杂性理论研究问题的本质，将各种问题按照其难易程度分类，研究各类问题的难度级别，并不关心解决问题的具体方案
- 而算法则研究问题在不同现实资源约束情况下的不同解决方案，致力于找到效率最高的方案
  - 不同硬件配置（手持设备、PC设备、超级计算机）
  - 不同运行环境（单机、多机环境、网络环境、小内存）
  - 不同应用领域（消费、工业控制、医疗系统、航天领域）
  - 甚至不同使用状况（正常状况、省电状况）
- 如何对具体的算法进行分析，并用衡量指标评价其复杂度

### 不可计算问题

- 有不少定义清晰，但无法解决的问题
  - 并不是尚未找到解，而是在”基于有穷观点的能行方法“的条件下，已被证明并不存在解决方案
- 停机问题：判断任何一个程序在任何一个输入情况下是否能够停机 （不可计算 不可解的问题）
- 不可计算数：几乎所有的无理数，都无法通过算法来确定其任意一位是什么数字
  - 可计算数很少：如圆周率Pi，自然对数的底e
- 似乎计算之道解决问题存在边界和极限？



## 105 突破计算极限

### 超大规模分布式计算

- SEIT@home 是一项利用全球联网计算机共同搜寻地外文明（SETI）的科学实验计划
  - 项目组把射电望远镜采集到的海量信息分成一个个小数据包，发送到互联网上
  - 每台安装了 SETI@home 软件的电脑都可以自动下载这些数据，以运行屏幕保护或者后台程序的方式参与数据分析。
  - 从1999年5月开始，目前，有170万人、450万台计算机正在参加搜寻
- SETI@home 从2005年开始并入BOINC平台
- BOINC也是公众参与科学计算的超大型分布式系统，托管了众多学科的计算搜寻项目
  - 天文、生命、数学、物理和化学问题
- 社会公众也能通过贡献计算力参与众包，进行科学探索

### 新型计算技术：光子计算

- 用超微透镜取代晶体管、以光信号代替电信号进行运算
- 光芯片无需改变二进制计算机的软件原理，但可以轻易实现极高的运算频率，同时能耗非常低，不需要复杂的散热装置

### 新型计算技术：DNA计算

- 以 DNA分子和酶的相互作用实现逻辑运算和数据存储，获得极高的计算并行度和存储能力。

### 新型计算技术：量子计算

- 利用量子力学态叠加原理，让信息单元处于多种可能性的叠加状态，从而实现指数级别的并行计算，根本上解决最高复杂度计算问题。

### 突破计算极限：分布式智慧——众包

- 突破”基于有穷观点的能行方法“？
  - ”如果无数多的猴子在无数多的打字机上随机地乱敲，并持续无限久的时间，那么在某个时候，必然有只猴子会打出莎士比亚的全部著作。”
- 如果是具有智慧和直觉的众多人脑一起来共同解决问题呢？

### 游戏化众包学术研究

- 一篇有57000位作者的 Nature论文
  - 《通过多人在线游戏预测蛋白质结构》2010.8

### Foldit：游戏化众包蛋白质结构分析

- 多人在线游戏，众多玩家在给定一个目标蛋白的情况下，用各种氨基酸进行组装，最终拼凑出这个蛋白的完全体
- 玩家只需要掌握基本方块的拼插技巧，即可跟全世界众多玩家一起协同工作，攻克科研难题，有60万人玩过这个游戏

### 更多智慧众包科研项目

- 突破算法的约束
  - 相比分布式计算的闲置计算力
  - 革命性地利用了空闲智力

## 106 什么是抽象和实现

### 计算机科学研究什么

- 计算机科学不仅仅是对计算机的研究
  - 虽然计算机是非常重要的计算工具
- 计算机科学主要研究的是问题、问题解决过程、以及问题的解决方案
  - 包括了前述的计算复杂性理论以及对算法的研究

### 抽象（Abstraction）

- 为了更好地处理机器相关性或独立性，引入了“抽象”的概念
- 用以从“逻辑 Logical”或者“物理Physical”的不同层次上看待问题及解决方案

### 什么是抽象？例子：汽车

- 从司机观点看来，汽车是一台可以带人去往目的地的代步工具
  - 司机上车、点火、换挡、踩油门加速、刹车
- 从抽象角度说，司机看到汽车的“逻辑”层次
  - 司机可以通过操作各个机构来达到运输的目的
- 这些操纵机构（方向盘、油门、档位）就称为“接口Interface”
- 而从汽车修理工的角度来看同一辆汽车，就会相当不同，他还需要清楚每项功能是如何实现的
  - 如发动机工作原理，档位操作的机械结构，发动机舱内各处温度如何测量和控制等等
- 这些内部构造构成了汽车的“物理”层次
- 工作过程就称为“实现 Implementation”

### 什么是抽象？例子：计算机

- 从一般大众用户观点看来，计算机可以用来编辑文档、收发邮件、上网聊天、处理照片等等
- 并不需要具备计算机内部如何处理的知识
  - 利用这些功能是计算机的“逻辑”层次
- 而对于计算机科学家、程序员、技术支持、系统管理员来说
- 就必须要了解从硬件结构、操作系统原理到网络协议等各方面的低层次细节
  - 内部如何实现，是计算机的“物理层次”

### “抽象”发生在各个不同层次上

- 逻辑还是物理是一个相对的概念
- 即使对于程序员来说，使用编程语言进行编程，也会涉及到“抽象”
- 如计算一个数的平方根
  - 程序员可以调用库函数 math.sqrt，直接得到结果，而无需关系起内部是如何实现
  - 这种功能上的“黑盒子”称作“过程抽象 Procedural Abstraction”

```python
import math
math.sqrt(16)  # 4.0
```

### 抽象与实现：编程

- 编程是通过一种程序设计语言，将抽象的算法实现为计算机可以执行的代码的过程
  - 没有算法，编程无从谈起
- 图灵奖获得者Niklaus Wirth 的著名公式：算法 + 数据结构 = 程序
  - 此公式相当于物理中的 E=mc²  
  - Pascal 语言设计者

### 程序设计语言实现算法的基本机制

- 程序设计语言需要为算法的实现提供实现“过程”和“数据”的机制
  - 具体表现为“控制结构”和“数据类型”
- 程序设计语言均有语句对应控制结构
  - 顺序处理、分支选择、循环迭代
- 程序设计语言也提供最基本的数据类型来表示数据，如整数、字符等
  - 但对于复杂的问题而言，直接使用这些基本数据类型不利于算法的表达



## 107 为什么研究数据结构与算法

### 清晰高效地表达算法

- 为了控制问题和问题解决过程的复杂度，利用抽象来保持问题的“整体感”。而不会陷入到过多的细节中去
- 这要求对现实问题进行建模的时候，对算法所要处理的数据，也要保持与问题本身的一致性，不要有太多与问题无关的细节

系统 问题

子系统 子问题  子问题

模块 步骤 1  步骤2  步骤

### 数据抽象：ADT 抽象数据类型

- “过程抽象” 启发我们进行“数据抽象”
- 相对于程序设计语言中基本数据类型，抽象数据类型（ADT:Abstract Data Type）
  - ADT 是对数据进行处理的一种逻辑描述，并不涉及如何实现这些处理
  - 用户 接口 实现 操作
- 抽象数据类型ADT建立了一种对数据的“封装 Encapsulation”
  - 封装技术将可能得处理实现细节隐蔽起来能有效控制算法的复杂度

### 数据结构是对ADT的具体实现

- 同一ADT可以采用不同的数据结构来实现
- 采用程序设计语言的控制结构和基本数据类型来实现ADT所提供的逻辑接口
  - 属于ADT的“物理”层次

### ADT：数据结构Data Structure

- 对数据实现“逻辑”层次和“物理”层次的分离，可以定义复杂的数据模型来解决问题，而不需要立即考虑此模型如何实现

### 接口的两端：抽象与实现

- 如电动车与汽油车
  - 底层动力、能源都不同
  - 但开车的操作接口（方向盘、油门、刹车、档位）基本都是相同的
  - 司机无需重新考驾照，而车厂可以持续改进实现技术
- 由于对抽象数据类型可以有多种实现方案
- 独立于实现的数据模型
  - 让底层开发程序员专注于实现和优化数据处理，又不改变数据的使用接口
  - 让用户专注于用数据接口来进行问题的解决，而无需考虑如何具体实现这些接口
- 通过层次抽象，降低问题解决过程的复杂度

### 为什么要研究和学习算法

- 首先，学习各种不同问题的解决方案
  - 有助于我们在面对未知问题的时候，能够根据类似问题的解决方案来更好解决
- 其次，各种算法通常有较大差异
  - 我们可以通过算法分析技术来评判算法本身特性
  - 而不仅仅根据实现算法的程序在特定机器和特定数据上运行的表现来评判他
  - 即使同一个程序，在不同的运行环境和输入数据的情况下，其表现的差异可能也会很大
- 在某些情况下，当我们碰到棘手的难题
  - 得能区分这种问题是根本不存在算法
  - 还是能找到算法，但需要耗费大量的资源
- 某些问题解决需要一些折衷的处理方式
  - 我们需要学会在不同算法之间进行选择，以适合当前条件的要求

## 108 从C转换到Python

### 从 计概C 转 数算Python

- 从Hello World 开始

### C：Hello World！

```c
#include <stdio.h>

int main()
{
  // say hello
  printf("Hello World!\n");
}
```

1- Compile 编译到机器码

2- Link 与各种库链接

3- Execute 执行目标程序

### Python：Hello World

```python
def main():
  # say hello
  print("Hello World!")
  
  
main()
```

1- Run! 跑代码

```python
# say hello
print("Hello World!")
```

main 都是太啰嗦，一行OK

- 帮高斯的同学回家

```c
#include <stdio.h>

int main()
{
  int s, i;
  s = 0;
  // adding numbers i to 100
  for (i = 0; i < 100; i++)
  {
    s += (i + 1);
  }
  printf("Sum= %d", s);
}
```

Python ：也帮高斯的同学回家

```python
s = 0
# adding numbers 1 to 100
for i in range(100):
  s += (i + 1)
print("Sum=", s)
```



- 检验素数

```c
#include <stdio.h>
#include <math.h>

int main()
{
  int n, i, flag = 0;
  printf("Please input number:");
  scanf("%d", &n);  // 输入
  for (i = 2; i < sqrt(n); i++)  // sqrt 开平方
    if (n % i == 0)
    {
      flag = 1;
      break;
    }
  if (flag == 1)
    printf("%d is NOT a prime number.", n);
  else
    printf("%d is a prime number.", n);
}
```

Python：检验素数

```python
from math import sqrt
n = int(input("Please input number:"))
for i in range(2, int(sqrt(n))):
  if n % i == 0:
    print(f"{n} is NOT a prime number.")
    break
else:
  print(f"{n} is a prime number.")
```



- 打印一个朴素的三角形

```c
#include <stdio.h>

int main()
{
  int n, i, j;
  printf("Please input number:");
  scanf("%d", &n);
  for (i = 0; i < n; i++)
  {
    for (j = 0; j < i; j++)
      printf("*");
    printf("\n");
  }
}
```

Python：打印一个朴素的三角形

```python
n = int(input("Please input number:"))
for i in range(n):
  print("*" * i)
```



- 进一步洗白
  - 从c口音转到Python不太容易
  -  <地空数算的PyLn编程学习平台  http://gis4g.pku.edu.cn>  
  - B站学习 **av87882118**



人生苦短我用Python




















