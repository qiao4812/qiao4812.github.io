---
title: "Python基础"
date: 2023-02-12T18:30:53+08:00
draft: true
---

# Python基础

## 一、 计算机介绍

### 1.1、 计算机发展

​	电子管-晶体管-集成电路-大规模集成电路计算机时代

​	世界上第一台通用计算机ENIAC于1946年2月14日在美国宾夕法尼亚大学诞生。

### 1.2、 计算机硬件组成

* CPU

* 存储器

* 输入输出设备

  

### 1.3、 冯诺依曼计算机



## 二、编程语言介绍

### 2.1、 什么是编程语言

### 2.2、 编译型语言与解释性语言



## 三、Python语言介绍

### 3.1、 了解Python语言

### 3.2、 Python解释器下载与安装

### 3.3、 第一个Python程序

### 3.4、Pycharm的安装与使用

## 四、基础语法

### 4.1、 变量

#### 4.1.1、 Python的标识符规范

- 标识符是由字符（A~Z 和 a~z）、下划线和数字组成，但第一个字符不能是数字。
- 标识符不能和 Python 中的保留字相同。有关保留字，后续章节会详细介绍。
- Python中的标识符中，不能包含空格、@、% 以及 $ 等特殊字符。
- 在 Python 中，标识符中的字母是严格区分大小写
- Python 语言中，以下划线开头的标识符有特殊含义
- Python 允许使用汉字作为标识符（不推荐）

### 4.2、 语句分隔符

### 4.3、 缩进

### 4.4、 注释

### 4.5、 Python编码规范（PEP 8）



### 4.6、 基本数据类型

#### 4.6.1、整型和浮点型

- 整型
- 浮点型

#### 4.6.2、 布尔类型

#### 4.6.3、 字符串

##### 4.6.3.1、字符串的常见操作

- 获取长度 len
- 查找内容 find index rfind rindex
- 判断 startswith endswith isalpha isdigit isalnum isspace
- 计算出现次数 count
- 替换内容 replace
- 切割字符串 split rsplit splitlines partition rpartition
- 修改大小写 capitalize title upper lower
- 空格处理 ljust rjust center lstrip rstrip strip
- 字符串拼接 join

```python
'''
切片：字符串 列表
格式：字符串变量[start:end]
字符串变量[start:end:step] 默认从左向右取一个一个取元素
step
1.步长
2.方向 step 正数 从左向右
           负数 从右向左
'''
s = 'ABCDEFG'
print(s[1:4])  # BCD
print(s[0:5])  # 从0到index=4
print(s[:5])   # 从0到index=4
print(s[-3:])  # 从-3开始到结尾
print(s[:])
print(s)
print(id(s[:]))
print(id(s))
print(s[1:-1])
print(s[:-1:2])
print(s[1:-1:2])
print(s[1::2])
print(s[::4])
print(s[0:5:4])
print(s[::-1])
print(s[::-2])
print(s[4::-4])

'''
https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png
find: 从左向右查找，只要遇到一个符合要求的则返回位置，如果没有找到任何符合要求的则返回-1
'''
path = 'https://www.baidu.com/img/PCtm_d9c8750bed0b3c7d089fa7d55720d6cf.png'
# find index rfind rindex
i = path.find('_')
print(i)
image_name = path[i+1:]
print(image_name)
# rfind 从右向左查找
i = path.rfind('.')
zhui = path[i:]
print(zhui)
# count 计算出现的次数
n = path.count('.')
print(n)
# index 查找 找不到会报错
# i = path.index('#')
# print(i)
i = path.find('imgd')
print(i)
s = 'd9c8750bed0b3c7d089fa7d55720d6cf.png'
# 判断是否是以xxx开头的
# result = s.startswith('abc')
result = s.startswith('d')
print(result)
# 判断是否是以xxx结尾的
result = s.endswith('png')
print(result)
```



```python
'''
模拟文件上传，键盘输入上传文件的名称（abc.jpg)，判断文件名(abc)是否大于6位以上，扩展名是否是：jpg，gif,png格式
如果不是则提示上传失败，如果名字不满足条件，而扩展名满足条件则随机生成一个6位数字组成的文件名，打印成功上传xxx.png
判断名字
'''
import random
s = 'QWERTYUIOPASDFGHJKLZXCVBNMqwertyuiopasdfghjklzxcvbnm1234567890'

file = input('输入图片文件全称:')
# 判断扩展名
if file.endswith('jpg') or file.endswith('gif') or file.endswith('png'):
    # 判断文件的名字
    i = file.rfind('.')
    name = file[:i]  # 切片
    if len(name) < 6:
        # 重新构建名字
        # n = random.randint(100000, 999999)
        # file = str(n) + file[i:]
        # 字母和数字的组合
        filename = ''
        # s = 'QWERTYUIOPASDFGHJKLZXCVBNMqwertyuiopasdfghjklzxcvbnm1234567890'
        for n in range(6):
            index = random.randint(0, len(s) - 1)

            filename += s[index]  # 获取下标匹配的字母
        # 文件名和后缀进行拼接
        file = filename + file[i:]
    print('成功上传%s文件' % file)
else:
    print('文件格式错误，上传失败！')

```



```python
s = 'HELLO'
# 是否全部是字母组成的
result = s.isalpha()
print(result)
# 数字
s = '100'
result = s.isdigit()
print(result)
s = 'A1234'
# 字母 数字
result = s.isalnum()
print(result)
# 纯空格 有一个
s = '    '
result = s.isspace()
print(result)
# 大写
s = 'HELLO'
result = s.isupper()
print(result)
# 小写
s = 'abc'
result = s.islower()
print(result)
'''
admin123 15811119999 200325
用户名或者手机号码登录+密码
用户名：全部小写，首字母不能是数字，长度必须6位以上
手机号码：纯数字 长度11位
密码必须是6位数字
以上符合条件则进入下层验证：
判断用户名+密码 是否是正确 则成功 否则登录失败
'''
flag = True
while flag:
    name = input('输入用户名/手机号码:')
    # 判断
    # if (name.islower() and not name[0].isdigit()) or '手机号':
    if (name.islower() and name[0].isalpha() and len(name)>=6) or (name.isdigit() and len(name)==11):
        while True:
            # 继续输入密码 多次输入密码
            password = input('密码:')
            # 判断密码是否是6位数字
            if len(password) == 6 and password.isdigit():
                # 验证nam+密码 正确性
                if (name == 'admin123' or name == '15811119999') and password == '200325':
                    print('用户登录成功！')
                    flag = False
                    break
                else:
                    print('用户名或者密码有误！')
                    break
            else:
                print('密码必须是6位数字')
    else:
        print('用户名或者手机号码格式错误')
```



```python
'''
# 替换内容 replace(old,new,count) 默认全部替换 也可以通过count指定次数
s = '李泽说：阿铄你来唱歌吧！阿铄你来跳舞吧！'
result = s.replace('阿铄', '**', 1)
print(result)
# 替换 李泽 阿铄 正则表达式 for循环 列表

# split('分隔符,maxsplit') 返回的结果是一个列表 maxsplit 最多分割次数
# 同理 rsplit
s = '李泽 阿铄 郭琪'
# result = s.split(' ')
result = s.rsplit(' ')
print(result)
'''
s = ''' 阿铄你来唱歌吧！
阿铄你来跳舞吧！
阿铄你来学习吧！
阿铄你来吃饭吧！
'''
'''
# 按行分割
result = s.splitlines()
print(result)
s = '李泽 阿铄 郭琪'
result = s.partition(' ')
print(result)
s = 'hello WORLD'
# 首字母大写
# result = s.title()
# 大写
# result = s.upper()
# 小写
# result = s.lower()
# 第一个字母大写
result = s.capitalize()
print(result)
# 移除头尾空格
s = ' admin  '
print(len(s))
result = s.strip()
print(len(result))
# 去除左侧空格
result = s.lstrip()
# result = s.rstrip()
print(len(result))
# center
s = 'hello world'
result = s.center(30)
print(result)
# ljust 左对齐
result = s.ljust(30)
print(result)
# rjust 右对齐 添加空格
result = s.rjust(30)
print(result)
s = 'hello' + 'world'
print(s)
'''
'''
格式化：
1. %d %s %f 
2. format
'''
# 省略字段名
name = '赵丽颖'
age = 18
result = '美女{}今年{}岁'.format(name,age)
print(result)


# 数字字段名
result = '美女{0}今年{1}岁,我也是{1}岁'.format(name, age)
print(result)
# 变量字段名
result = '美女{name}今年{age}岁,我也是{age}岁'.format(name='赵丽颖', age=19)
print(result)
#
name = '小明'
score_chinese = 100
score_math = 99
s = '{0}本次考试数学分数是：{1}，语文分数：{2}，英语分数{1}'.format(name,score_math,score_chinese)
print(s)
s = '{name}本次考试数学分数是：{score_math}，语文分数：{score_chinese}，英语分数{score_math}'.format(name= '小明',score_math= 99,score_chinese= 100)
print(s)
'''
模拟论坛 输入用户名：小白
反复回复 回复的内容不能为空 len(lun.strip(0))==0
里面不能存在敏感词汇
最多评论20个字，剩余多少个字
回复的内容前后不能有空格
'''
msg = input('发表一句话:')
print('_'*50)
print('以下为回复内容:')
while True:
    # 输入用户名
    username = input('用户名:')
    while True:
        # 输入回复内容
        comment = input('评论:')
        comment = comment.strip()
        # 验证内容
        if len(comment.strip())!=0:
            # 验证长度是否超出20个字
            if len(comment.strip())<=20:
                # 是否存在敏感词汇
                comment= comment.replace('丑陋', '**')
                print('\t{}发表的评论是:\n\t{}'.format(username, comment))
                break
            else:
                print('不能超出20字!')
            pass
        else:
            print('评论内容不能为空')
    # 成功则发出评论 否则重新输入


```



### 4.7、 运算符

### 4.7.1、 算术运算符

```python
# 键盘输入一个3位数的整数 打印个位数、十位数、百位数
# number = input('请输入一个3位数：')
# number = int(number)
number = int(input('请输入一个3位整数：'))
print('个位数:',number % 10)
print('十位数:',number // 10 % 10)
print('百位数:',number // 100)
```





### 4.7.2、 赋值运算符

=  +=  -=  *=   /=  //=   %=

```python
a = 8
b = 4
c = 0
# c = a + 1
# print(a,b,c)
a += 1 # a = a+1
print(a,b,c)

a += b # a = a+b
print(a,b,c)
b += a # b = a+b
print(a,b,c)

b -= 2 # b = b-2
print(a,b,c)

d = 3
b //= d # b=b//d
print(a,b,c,d)
```



### 4.7.3、 比较(关系)运算符

​	>  <  ==  >=  <=  !=  is  is not    结果：True  False

```python
a = 10
b = 23
print(a > b)
print(a < b)

print(a == b)
print(a != b)

x = 'abc'
y = 'abcd'
print(x == y)
print(x < y)

'''
输入考试分数，判断成绩是否在100到80之间
'''
score = float(input('输入分数:'))
print(80 <= score <= 100)
```



### 4.7.4、 逻辑运算符

​	and  or  not  

and 与  并且

A and B

True and True    --->True

True and False    --->False

False and True    --->False

False and False    --->False

or :  或  或者  只要有一侧为真True 结果即为True

not 非

not True    --->False

```python
a = 1
b = 3
# and 两边都是非0数字，结果是最后的数字值
print(b and a)

c = 0
# and 两边只要有一边为0，结果即为0
print(c and a)
print(a > c and a < b)
print(a == c and a < c)

print('#' * 20)
print(b or a)
print(a or b)
print(c or b)
print(a > c or a < b)
print(a == c or a < c)  # 场景：1. 账号密码 or 2. 手机号码 验证码
# 账号名/手机号码 + 密码

flag = True
print(not flag)
print(not a>c)
```

##### 进制转换

已知二进制转化十六进制，将二进制从右侧开始4位为一组

```python
# 十进制转二进制
n = 149
result = bin(n)
print(result)
# 十进制转八进制
result = oct(n)
print(result)
# 十进制转十六进制
result = hex(n)
print(result)

# 0b二进制 0o八进制  0x十六进制 默认十进制

#十六进制转换十进制
n = 0x558
result = int(n)
print(result)
# 十六进制转换二进制
result = bin(n) # 无论当前参数是几进制
print(result)

x = 0b10101011000
print(int(x))
# 转八进制
result =oct(n)
print(result)
```

### 4.8、 位运算

&  |  ^ 异或 上下相同为0不同为1   ~取反  <<  >>

二进制的负数表示：

原码  0110

反码  1001

补码  反码+1  1010

已知十进制负数，求二进制负数：

1.正数的原码

2. 原码取反
3. 加一

已知二进制的负数（判断是否是负的二进制的依据，看二进制的最高位：1111 1010，最高位是1则为负数，0则为正数），求对应的十进制。

1. 二进制（负的） 2. 二进制减一 3. 取反 4.原码 将原码转成十进制，在十进制前面添加负号：-

```python
# << 左移
n = 12
print(n << 1)  # 24 12*2
print(n << 2)  # 48 12*2*2
print(n << 3)  # 96 12*2*2*2
n = 26
print(n << 4)  # 416 26*2*2*2*2
# >> 右移
n = 12
print(n >> 1)  # 6 12//2  整除
print(n >> 2)  # 3 12//4
print(n >> 3)  # 1 12//8
n = 89
print(n >> 5)
n = 93
print(n << 3)
x = 20
result = 20*5**2
print(result)
a = 1
b = 2
print(b > a + x < a + b)
print(b > a + (x < a + b))
print(a + False)
print(a+True)  # False 0 True 1
'''
总结：
1. 算术运算符
2. 赋值运算符
3. 比较运算符
4. 逻辑运算夫 and or not
5. 位运算 & | ^ << >> 负数的二进制
6. 运算符的优先级：（）
'''
```





### 4.9、 输入输出函数

input

print

## 五、流程控制语句

### 5.1、 分支语句

#### 5.1.1、 单分支语句

#### 5.1.2、 双分支语句

```python
'''
 根据今天所学的知识 以及我们之前一直在使用的if语句 完成如下需求的案例
    A:创建用户身份信息存储  存储卡号，姓名，性别，存款 如果不存，默认为0元
       可能用到的函数(int(),str(),float())
    B:比如用户输入银行卡号  返回该用户的姓名 ，性别， 存款余额(保留2位小数)
     如果银行卡号不存在 则输出 请输入正确的银行卡号
'''
uselist = {'name': 'xiaobai', 'sex': '男', 'card': 1234, 'Deposits': 1000.1264}
dep = round(float(uselist['Deposits']), 2)
# print(dep)
card_num = int(input('请输入银行卡号:'))
name = uselist['name']
sex = uselist['sex']
if card_num == uselist['card']:
    print('姓名:%s' % name)
    print('性别:%s' % sex)
    print('存款余额:%.2f' % dep)
    sav = input('请问是否存款(y/n):')
    if sav == 'y':
        sav1 = round(float(input('请输入存款金额:')))
        dep += sav1
        print('这是您的存款余额:%.2f' % dep)

    else:
        print('不存款')
        dep += 0
        print('这是您的存款余额:%.2f' % dep)

else:
    print('请输入正确的银行卡号')
```



### 5.1.3、 多分支语句

```python
"""
条件语句：
if
if...else
if...elif...else

条件：运算符构成 ---》bool
if语句的格式：
if 条件:
    条件成立要执行的语句
"""
# 4,5 有条件打印
# result = input('请输入(y/n):')
# if result == 'y':
#     print(4)
#     print('over~~~')
#
# print('--------')

# 随机数
import random
ran = random.randint(1,10)
print(ran)
guess = input('请输入你猜的数：')
if ran == int(guess):
    print('恭喜你猜对了！')
else:
    print('很遗憾猜错了！')

import random
ran = random.randint(1,22)
print(ran)
# 输入账号和密码（名：admin，密码1234），判断是否正确，正确显示登录成功，否则登录失败
username = input('用户名:')
password = input('密码:')
# 比较判断
if username == 'admin' and password == '1234':
    print('登录成功')
else:
    print('用户名或者密码输入错误！')
# 产生两个随机数1-10，判断两个数字之间的和是否大于8并且差小于等于3，如果是则显示：success 否则显示：fail
number1 = random.randint(1,10)
number2 = random.randint(1,10)
print(number1,number2)
if number1+number2>8 and abs(number1-number2)<=3:
    print('success')
else:
    print('fail')

'''
if...else
格式：
if 条件1:
    条件1True,执行的语句
elif 条件2:
    条件2True,执行的语句
elif 条件3:
    条件3True,执行的语句
...
else:
    1,2,3条件都不符合的情况下

'''
# 输入销售金额，符合哪种奖励的范围
'''
1-5: 1000元
5-10: 奖励笔记本IBM
10-100: 奖励iPhone12
超过了100: 奖励特斯拉
鼓励奖
'''
# money = 100000
# if 10000<money<=50000:
#     print('奖励1000元！恭喜！')
# elif 50000<money<=100000:
#     print('奖励笔记本IBM!恭喜！')
# elif 100000<money<=1000000:
#     print('奖励iPhone12！恭喜！')
# elif money>1000000:
#     print('奖励特斯拉！恭喜！')
# else:
#     print('鼓励奖,毛绒玩具！')
'''
人员管理系统，功能：添加员工 删除员工 查询员工 修改员工信息

'''
print('------------欢迎进入人员管理系统------------')
choice = input('请选择功能:\n 1.添加员工\n 2.删除员工\n 3.查询员工\n 4.修改员工信息\n')
if choice == '1':
    print('添加员工')
elif choice == '2':
    print('删除员工')
elif choice == '3':
    print('查询员工')
elif choice == '4':
    print('修改员工信息')
else:
    print('输入错误！')

    
'''
键盘上输入以下内容并打印输出：
默认（admin，1234）
用户名，username
密码,password
是否记住密码 bool类型 is_remember
如果用户名和密码正确并且is_remember是True表示记住密码 则打印已经记住用户xxx的密码啦
否则打印 没有记住密码需要下次继续输入的

'''
username = input('用户名:')
password = input('密码:')
is_remember = False
if username =='admin' and password == '1234':
    # 判断
    if is_remember:
        print('已经记住用户%s的密码啦' % username)
    else:
        print('没有记住密码需要下次继续输入的')
else:
    print('用户名或者密码有误！')
    
    
'''
模拟超市付款：商品单价 商品数量
键盘上输入商品单价 商品数量
然后计算应付总额
计算总额 float
提示用户可以有4种付款方式
现金没有折扣
微信 0.95折
支付宝 鼓励金付款金额的10% 鼓励金可以直接折算到付款金额中
刷卡 满100-20
'''
print('欢迎光临北科超市')
price = float(input('商品单价:'))
number = int(input('商品数量:'))
# 计算总额
total = price*number
# 选择付款方式
choice = input('选择付款方式:1.现金，2.微信，3.支付宝，4.刷卡\n')
# 判断
if choice == '1':
    print('现金付款没有折扣，应付金额是:%.2f'%total)
elif choice == '2':
    # 微信 0.95折
    print('微信支付....')
    total *= 0.95
    print('支付金额是:%.2f'% total)


elif choice == '3':
    # 支付宝 鼓励金付款金额的10% 鼓励金可以直接折算到付款金额中
    total = total - total * 0.1
    print('支付宝支付...')
    print('支付金额是:%.2f' % total)
elif choice == '4':
    # 刷卡 满100-20
    print('刷卡支付...')
    if total>100:
        total -= 20
        print('支付金额是:%.2f' % total)
    else:
        print('没有折扣，支付金额是::%.2f' % total)
else:
    print('输入错误！')
    
a=1
b=2
# if a<b:
#     c=a
# else:
#     c=b
# print(a,b,c)
c=a if a<b else b
print(a,b,c)
```



### 5.2、 循环语句

#### 5.2.1、 while循环

1. 固定次数的循环
2. 不确定次数的循环

```python
'''
循环：
场景：
1. 用户名和密码，反复输入
2. 计算1-100
3. 游戏 死了重生
4. ...
方式：
1.while
2.for
while格式：
while 条件:
    要循环执行的代码
布尔类型的条件
'''
# 打印1-10之间的数字
# 初始值
# n = 1
# # 结束条件
# while n <= 10:
#     print('------>n=%d' % n)
#     # 变量要有变化
#     n += 1

'''
打印1-50之间能被3整除的数字
'''
n = 1
# while n <= 50:
#     if n % 3 == 0:
#         print('---->', n)
#
#     n += 1
n = 3
while n <= 50:
    print('---->', n)
    n += 3

    n = 1
sum = 0  #  容器 存放和的结果
while n <= 10:
    sum += n

    n += 1
print('累加和：sum=',sum)

# count = 0
# while True:
#     print('11111111')
#     count += 1
#     if count == 5:
#         break

total = 0
while True:
    # 先买
    price = float(input('输入价格:'))
    number = int(input('输入数量:'))
    # 累加
    total += price * number
    # 判断是否继续购买
    answer = input('当前商品总额:%.2f,是否继续添加商品（q表示退出）？' % total)
    if answer == 'q':
        # 跳出while循环
        break
print('所有商品的总额是:%.2f' % total)


total = 0
numbers = 0
while True:
    # 先买
    price = float(input('输入价格:'))
    number = int(input('输入数量:'))
    # 累加
    total += price * number
    # 数量累加
    numbers += number
    # 判断是否继续购买
    answer = input('当前商品总额:%.2f,是否继续添加商品（q表示退出）？' % total)
    if answer == 'q':
        # 跳出while循环
        break
print('商品数量共:%.d,总额是:%.2f' % (numbers, total))


'''
产生随机数
可以猜多次 直到猜对为止 如果猜错了适当给出提示 猜大了还是猜小了
1. 统计猜了几次
2. 如果1次就中 赶快去买彩票吧，运气爆了
   2-5，猜对了，运气还可以哦
   6以上，猜对了，运气一般啊
'''
import random
ran = random.randint(1, 50)
count = 0
# 循环猜多次
while True:
    guess = int(input('猜一个1-50之间的数字:'))
    # count 改变
    count += 1
    # 猜对就结束
    if guess == ran:
        if count == 1:
            print('赶快去买彩票吧，运气爆了!')
        elif 2 <= count <= 5:
            print('猜对了，运气还可以哦!')
        elif count >= 6:
            print('猜对了，运气一般啊！')
        break
    elif guess > ran:
        print('猜大了，再小一点！')
    else:
        print('猜小了，再大一点！')
        
        
'''
猜拳游戏：3局
'''
import random
n = 1
# 计数
p_count = 0
m_count = 0
while n <= 3:
    # 猜拳
    # 机器产生数字 0 1 2
    ran = random.randint(0, 2)
    # 人猜数字
    guess = int(input('请输入:剪刀(0) 石头(1) 布(2)\n'))
    if guess == 0 and ran== 2 or guess==1 and ran==0 or guess==2 and ran==1:
        print('~~~本轮我赢了！~~~')
        p_count += 1
    elif ran == 0 and guess== 2 or ran==1 and guess==0 or ran==2 and guess==1:
        print('~~~本轮机器赢了~~~')
        m_count += 1
    else:
        print('本轮平局！')
    # n的变化
    n += 1
# 比较胜负
if p_count > m_count:
    print('最终人获胜了！')
elif p_count < m_count:
    print('机器最终获胜！')
else:
    print('最终平局！')

```



#### 5.2.2、 for循环

固定次数

```python
'''
for i in range(n):
    循环体中的内容

range(n): 从0开始取值到n-1结束
range(stop) -> range object
range(start, stop[, step]) -> range object
'''
# 1-10 数字打印
# for i in range(10):
#     print(i)
# 1-50 的累加和
sum = 0
for i in range(1, 51):
    sum += i
print(sum)
# 最多输入用户名和密码，如果三次没有登录成功，提示账号被锁定
# break 退出
i = 0
for i in range(3):
    # 提示输入用户名和密码
    username = input('用户名:')
    password = input('密码:')
    # 判断输入的是否正确 admin 1234
    if username == 'admin' and password == '1234':
        print('用户登录成功！')
        break
    else:
        print('用户名或者密码有误！\n')
# i += 1
# print('账户被锁定！已输入错误%s次' % i)
if i == 2:
    print('账户被锁定！')
    
    
    
'''
for...else
if 条件:
    pass
else:
    pass
    
for i in range(n):
    循环体
else:
    如果上面的for循环0~n-1没有出现中断 （break）
'''
# 最多输入用户名和密码，如果三次没有登录成功，提示账号被锁定
# break 退出
# i = 0
for i in range(3):
    # 提示输入用户名和密码
    username = input('用户名:')
    password = input('密码:')
    # 判断输入的是否正确 admin 1234
    if username == 'admin' and password == '1234':
        print('用户登录成功！')
        break
    else:
        print('用户名或者密码有误！\n')
# i += 1
# print('账户被锁定！已输入错误%s次' % i)
# if i == 2:
else:
    print('账户被锁定！')
```





#### 5.2.3、 退出循环

- break

```python
'''
掷骰子
两个 1-6
1. 玩游戏要有金币
2. 玩游戏赠金币1枚，充值获取金币
3. 10元的倍数，20个金币
4. 玩游戏消耗金币5个
5. 猜大小：猜对 鼓励金币2枚 猜错无奖励 超出6点以上是大否则是小
6. 游戏结束：1.主动退出 2.没有金币退出
7. 退出打印金币数，共玩了几局
'''
import random
# 金币数
coins = 0
# 计数器
count = 0
if coins < 5:
    # 提示充值
    print('金币不足请充值再玩！')
    # 多次充值
    while True:
        money = int(input('请输入充值金额:'))
        # 10元的倍数
        if money % 10 == 0:
            coins += money // 10 *20
            print('充值成功！当前金币有%d个' % coins)
            # 开启游戏之旅
            print('~~~~开启游戏之旅~~~')
            answer = input('请问是否开始游戏(y/n)?')
            while coins > 5 and answer == 'y':
                # 扣金币
                coins -= 5
                # 赠送金币
                coins += 1
                # 产生两枚随机的骰子数
                ran1 = random.randint(1,6)
                ran2 = random.randint(1,6)
                # 猜大小
                guess = input('洗牌完毕，请猜大小:')
                # 判断比较
                if guess == '大' and ran1+ran2>6 or guess == '小' and ran1+ran2 <= 6:
                    print('恭喜猜对了，你赢了！')
                    # 鼓励金
                    coins += 2
                else:
                    print('很遗憾！猜错了！')
                # 玩的次数
                count += 1
                answer = input('请问是否继续游戏(y/n)?')
            # 打印 次数 金币数
            print('共玩了%d次，剩余金币:%d' % (count, coins))
            break


        else:
            print('不是10的倍数，充值失败！')
```



- continue

#### 5.2.4、 循环嵌套

- 独立嵌套

```python
'''
break
continue
都是出现在循环中
不同：
break 跳出循环结构
continue 跳出本次循环（后面的语句不执行）继续下一次循环
'''
# 不打印能被3整除的
# for i in range(10):
#     if i % 3 != 0:
#         print(i)
# for i in range(10):
#     if i % 3 == 0:
#         continue
#     print(i)
# 循环嵌套
# n = 1
# while n <= 4:
#     print('*' * n)
#     n += 1
n = 1
while n <= 5:
    m = 0
    while m < n:
        print('*', end='')
        m += 1
    n += 1
    print()
    
    
n = 1
while n <= 5:  # 控制行数
    m = 5
    while m >= n:
        print('*', end='')
        m -= 1
    n += 1
    print()
```



- 关联嵌套



## 六、重要数据类型

6.1、 列表

```python
# 获取列表里面的元素，通过索引下标
list = ['牛奶', '面包', '火腿肠', '辣条', '臭豆腐']
print(list[1])
print(list[3])
print(list[-2])
# 切片
# print(list[:2])
# print(list[2:4])
# print(list[2:-1])
# print(list[-3:-1])
# print(list[::-3])
# print(list[1::3])
# s = 'abcd123'
# for i in s:
#     print(i)
'''
添加 删除 修改 查询
1. 添加元素 append 追加 附加 类似排队
'''
list1 = []
list2 = ['面包']
list1.append('火腿肠')
print(list1)
list1.append('酸奶')
list1.append('辣条')
print(list1)
list2.append('薯条')
print(list2)
# list1 list2
list1 = list1 + list2
print(list1)
'''
数字: n = 1+3
字符串: s = 'aa'+'bb'
列表: list0 = [1,2,3]+['a','b']
'''
list1.extend(list2)
print(list1)
'''
买多件：
商品名称 价格 数量
要用到列表嵌套
'''
container = []  # 存多件商品的容器
flag = True
while flag:
    # 添加商品
    name = input('商品名称:')
    price = input('商品价格:')
    number = input('商品数量:')
    goods = [name, price, number]
    # 将商品添加到container
    container.append(goods)
    answer = input('是否继续添加？(按q或Q退出添加)')
    if answer.lower() == 'q':
        flag = False

print('*_'*20)
# 遍历容器container
print('商品名称\t商品价格\t商品数量')
for goods in container:
    print('{}\t{}\t{}'.format(goods[0], float(goods[1]), int(goods[2])))

    sum = ((float(goods[1]))*(int(goods[2])))

    # print(sum)
    print('共购买%d件，商品总额是:%d' % (int(goods[2]), sum))


print('商品总价:{}'.format(sum))
```



```python
'''
删除 pop remove clear
pop(index) 根据下标删除列表中元素，下标写的时候不要超出范围
pop() 从后往前删除
remove（element） 根据元素名称删除 如果删除的元素列表中不存在则报错
如果列表中存在多个同名元素element，只删除遇到的第一个元素，后面的元素不会被删除
clear（）
'''
list1 = ['牛奶', '面包', '火腿肠', '辣条', '臭豆腐', '酸奶', '酸奶']
# list.pop(-1)
# list.pop(8)
# print(list1)
# list1.pop()
# print(list1)
# remove
# list1.remove('辣条')
# print(list1)
# # 判断是否存在要删除的元素
# if '辣条啊' in list1:
#     print('存在')
# else:
#     print('不存在')
# 删除多个元素
# for i in list1:
#     if i == '酸奶':
#         list1.remove(i)
#
# print(list1)

# n = 0
# while n < len(list1):
#     if list1[n] == '酸奶':
#         list1.remove('酸奶')
#     else:
#         n += 1
# print(list1)

# li = [1, 1, 1, 2, 3]
# elem = 1
# result_li = []
# for i in li:
#     if i != elem:
#         result_li.append(i)
# li = result_li
# print(li)
# li = [2, 1, '酸奶', '酸奶', 1]
# for i in li[::-1]:
#     if i == '酸奶':
#         li.remove(i)
# print(li)
li = [2, 1, '酸奶', '酸奶', 1]
# 插入 insert(位置,元素) 元素占了位置 其他元素只能向后移动
# li.insert(1, 8)
# weizhi = li.index(1)
# li[weizhi] = 8
# # li[1] = 8
# print(li)
# # clear 清空列表元素
# list1.clear()
# print(list1)
'''
查找：
1. 元素 in 列表 返回bool类型   元素 not in 列表
2. 列表.index(元素) 返回元素的下标位置 如果没有报错
3. 列表.count() 返回整数 返回值是0则表示不存在此元素 存在则返回个数
'''
list1 = [1, 5, 4, 6, 5, 9, 5]
# print(list1.count(5))
# del 根据下标进行删除
# # del list1[3]
# list1.pop(3)
# print(list1)
# # del list1
# # list1.clear()
# list1.append(1000)
# print(list1)
'''
sort 
reverse
'''
# 生产8个1-20之间的随机整数，保存到列表中 遍历打印
# import random
# numbers = []
# for i in range(8):
#     ran = random.randint(1, 100)
#     # print(ran)
#     numbers.append(ran)
# # print(numbers)
# # numbers.sort() #  默认升序的，可以通过reverse控制
# # print(numbers)
# # numbers.sort(reverse=True)
# # print(numbers)
# numbers.reverse()  # 单纯的反转
# # print(numbers)
#
# '''
# 生产8个1-100之间的随机整数，保存到列表中 遍历打印
# 键盘输入一个1-100之间的整数，将整数插入到排序后的列表中
# '''
# # num = int(input('输入一个1-100的整数:'))
# numbers = []
# for i in range(8):
#     ran = random.randint(1, 100)
#     # print(ran)
#     numbers.append(ran)
#     numbers.sort()
# # numbers.append(num)
# # numbers.sort()
#
# # print(numbers)
#
#
# num = int(input('请输入一个数字:'))
#
# for i in range(len(numbers)):
#     if (num >= (numbers[i])) and (num < (numbers[i+1])):
#         numbers.insert(i+1, num)
#         break
# print(numbers)
a = 2
b = 3
# c = a
# a = b
# b = c
# print(a, b)
# a, b = b, a
# print(a, b)
a, b, c = b, a, b
print(a, b, c)
nums = [5, 1, 7, 6, 8, 2, 4, 3]
print(len(nums))
for i in range(0, len(nums) - 1):
    for j in range(0, len(nums) - 1 - i):
        if nums[j] > nums[j + 1]:
            a = nums[j]
            nums[j] = nums[j + 1]
            nums[j + 1] = a

```

列表推导式：最终得到的是一个列表

格式：[i for i in 可迭代的]

list1 = [i for i in range(1, 21)]

```python
# 列表推导式
list1 = [i for i in range(1, 21)]
print(list1)
list2 = [i+2 for i in range(1, 10)]
print(list2)

# 1-100 之间的偶数
list3 = [i for i in range(0, 101, 2)]
print(list3)
list4 = [i for i in range(0, 101,) if i % 2 == 0]
print(list4)
list5 = ['62', 'hello', '100', 'world', 'luck', '88']
list = [w for w in list5 if w.isalpha()]
print(list)
# h开头的首字母大写 不是全部转大写
list6 = [word.title() if word.startswith('h') else word.upper() for word in list5]
print(list6)
list7 = [(i, j)for i in range(1, 3) for j in range(1, 3)]
print(list7)
```

格式2 [i for i in 可迭代的 if 条件]

格式3 [结果1 if 条件 else 结果2 for 变量 in 可迭代的 ]

6.2、 元组

python的元组和列表类似，不同之处在于元组的元素不能修改（增加 删除 修改）

元组使用小括号()，列表使用方括号[]

list 列表 tuple 元组

定义：名 = ()

注意：如果元组中只有一个元素，必须添加逗号 ('aaa',)  (1,) (2.8,)

```python
t1 = ()
print(type(t1))  #  <class 'tuple'>
t2 = ('aa',)
print(type(t2))
t3 = ('a', 'b', 'c', 'u', 'e', 'a')
# print(type(t3))
# # 下标和切片 字符串，列表，元组   ---》注意下标越界
# print(t3[2])
# print(t3[1:])
# print(t3[::-1])
# n = t3.count('a')  # 计数
# print(n)
# 根据元素获取下标位置
# index = t3.index('a', 1)
index = t3.index('a', 1, 6)
# index = t3.index('a')
print(index)
# l = len(t3)
# print(l)
# # in, not in
# if 'c' in t3:
#     print('存在')
# else:
#     print('不存在')
# # 支持 for...in循环
# for i in t3:
#     print(i)
# list(tuple)  元组转成列表
# tuple(list)  列表转成元组
t3 = list(t3)
print(t3)
t3.append('x')
print(t3)
t3 = tuple(t3)
print(t3)

'''
王者荣耀角色管理
角色：姓名，性别，职业
添加角色
删除角色
修改角色
查询角色 单个角色
显示所有角色
退出管理系统
'''
import time
all_role = []  # 存放所有角色的容器
print('----------欢迎进入王者荣耀角色管理----------')

while True:
    choice = input('请选择功能:\n 1.添加角色 \n 2.删除角色 \n 3.修改角色 \n 4.查询角色 \n 5.显示所有角色 \n 6.退出系统 \n')
    if choice == '1':
        print('添加角色模块:')
        name = input('\t角色名:')
        sex = input('\t性别:')
        job = input('\t职业:')
        role = [name, sex, job]
        # 添加到all_role
        all_role.append(role)
        print('\t成功添加{}到王者荣耀系统!\n'.format(name))
    elif choice == '2':
        print('进入删除角色模块:')
        role_name = input('输入角色名:')
        # 查找是否存在此角色名称
        for role in all_role:
            if role_name in role:
                # 添加一个是否确定删除的询问 确定删除则再删除
                all_role.remove(role)
                print('成功删除角色{}'.format(role_name))
                break
        else:
            print('本系统不存在角色:{},请检查角色名称'.format(role_name))
    elif choice == '3':
        print('进入修改角色模块:')
        role_name = input('输入角色名:')
        # 查找是否存在此角色名称
        for role in all_role:
            if role_name in role:
                # 不完整缺少步骤
                all_role.extend(role)
                print('成功修改角色{}'.format(role_name))
                break
        else:
            print('本系统不存在角色:{},请检查角色名称'.format(role_name))

    elif choice == '4':
        print('进入查询角色模块:')
        role_name = input('\t输入角色名:')
        # 查找是否存在此角色名称
        for role in all_role:
            if role_name in role:
                print('\t存在此角色信息如下:')
                print('\t{}{}{}'.format(role[0].ljust(10), role[1].ljust(10), role[2].ljust(10)))

                break
        else:
            print('\t本系统不存在角色:{},请检查角色名称!'.format(role_name))
    elif choice == '5':
        print('显示所有角色模块:')
        # print('\t 名称 \t 性别 \t 职业')
        print('{}{}{}'.format('名称'.center(10), '性别'.center(10),'职业'.center(10)))
        for role in all_role:
            print(role[0].center(10), end='')
            print(role[1].center(10), end='')
            print(role[2].center(10), end='')
            print()


    elif choice == '6':
        print('正在退出王者荣耀管理系统~~~')
        time.sleep(3)

        print('成功退出！')
        break
    else:
        print('输入错误，请重新选择！')
```



### 6.3、字典

健 可以添加 删除 不能修改 只能修改健后面的值

字典名[key] = value  不存在健 添加 存在 修改

获取：

dict.get(key) 根据key获取value值

dict[key] 根据key获取value值   报error错误



```python
'''
练习:
book = {}
书名，价格，作者，出版社
促销：价格折扣 8折
打印最终字典中的内容
'''
book = {}
book['书名'] = '《平凡的世界》'
# book['价格'] = 40
# book['作者'] = '路遥'
# book['出版社'] = '人民出版社'
# book['价格'] *= 0.8
# print(book)
# book = {'书名': '《平凡的世界》', '价格': 32.0, '作者': '路遥', '出版社': '人民出版社'}
# book.clear()
# pop删除 根据key实现删除 键值对 返回值key对应的value
# r = book.pop('出版社')
# print(r)
# print(book)
# popitem() 返回值  元组 从后往前删除
# r = book.popitem()
# print(r)
# print(book)
# # del
# del book['价格']
# print(book)
books = [
    {'书名': '《平凡的世界》', '价格': 32, '作者': '路遥', '出版社': '人民出版社'},
    {'书名': '《流浪地球》', '价格': 18, '作者': '刘慈欣', '出版社': '华夏出版社'}
         ]
# for book in books:
#
#     book.pop('出版社')
# print(books)
book = {'书名': '《流浪地球》', '价格': 18, '作者': '刘慈欣', '出版社': '华夏出版社'}
value = book.get('书名')
print(value)
# print(len(book))
value = book['书名']
# print(value)
# for i in book:
#     print(i)  # key
# 获取字典中所有的值value
# print(list(book.values()))
# for i in book.values():
#     print(i)
# 获取字典所有的key
# print(list(book.keys()))
# for i in book.keys():
#     print(i)
# print(list(book.items()))
# for i in book.items():
#     print(i)
# for k, v in book.items():
#     print(k, v)
# # 只能添加键值对使用
# book.setdefault('出版', '人民教育出版社')
# print(book)
dict1 = {'a': 10, 'b': 20}
# update 实现两个字典的合并
# book.update(dict1)
# print(book)
# result = book.fromkeys(['a', 'b'])
# print(result)
'''
books = [] 框 能放多本书
书: {}
书名 作者 价格
1.添加3本书 不能添加同名书籍 
'''
# books = []
# while True:
#     if len(books) == 3:
#         break
#     name = input('输入书名:')
#     for book in books:
#         # if name in book.values():
#         if name == book.get('name'):
#             print('书名重复')
#             break
#     else:
#         author = input('输入作者:')
#         price = float(input('输入价格:'))
#         books.append({
#             'name': name,
#             'author': author,
#             'price': price
#         })
books = []
flags = True
while flags:
    book = input('请输入添加的书名，作者，价格（以空格分割）:').split(' ')
    for i in books:
        if i['书名'] == book[0]:
            print(f'[{book[0]}] 已存在，已跳过此选项')
            break
    else:
        books.append({'书名': book[0], '作者': book[1], '价格': book[2]})
    if len(books) == 3:
        flags = False
print(books)
```



### 6.4、 集合

list: 允许重复 有序 有下标 []

tuple:允许重复 里面的元素不能增加删除修改 只能查看 ()

dict: 键值对存在 健 唯一 值 允许重复 {:}

set: 不允许重复 无序 {}

类型转换：

list   ---> tuple  dict (特殊情况  [(a,b),(c,d)])

tuple ---->list tuple

dict  --->  list[key]  tuple set   只转健

set ---> list tuple

## 七、函数

1、什么是函数 函数就是盛放代码的容器

 	具备某一功能的工具

​	事先准备工具的过程

​	遇到应用场景拿来就用

函数的使用原则  先定义 后调用

2、为何要用函数

- 解决下述问题：

  1、代码组织结构不清晰、可读性差

  2、可扩展性差

3、如何用函数

- 定义的语法：

  def 函数名(参数1,参数2,参数3,...):

  ​	'''函数的文档注释'''

  ​	代码1

  ​	代码2

  ​	代码3

  ​	return 返回值

  

- 调用的语法

- 定义函数

  1、申请内存空间把函数整体代码放进去

  2、将函数内存地址绑定给函数名

  强调：定义函数只检测语法，不执行代码

  - 定义函数的三种形式

    1.无参

    ```python
    def func():
        print('xxx')
    func()
    ```

    

    2.有参

    ```python
    def max2(x,y):
        if x > y:
            print(x)
        else:
            print(y)
            
    max2(10, 20)
    ```

    

    3.空

    ```python
    def login():
        pass
    ```

    

- 调用函数

  函数名()

  1、先通过函数名定位到函数的内存地址

  2、函数内存地址()触发函数整体代码的运行

  强调：调用函数才会执行函数整体代码

  - 调用函数的三种形式

    1.语句

    ```python
    len('hello')
    ```

    

    2.表达式

    ```python
    res = len('hello') * 10
    print(res)
    ```

    

    3.可以当做参数传给另外一个函数

    ```python
    print(len('hello'))
    ```

4.函数参数

- 形参：在函数定义阶段括号内指定的参数，称之为形式参数，简称形参   变量名
- 实参：在函数调用阶段括号内传入的值，称之为实际参数，简称实参  变量的值

形参与实参的关系：在调用函数时，实参值会绑定给形参名，在函数调用完毕后解除绑定

细分：

- 位置形参：在定义函数时，按照从左到右的顺序依次定义的变量名 特点：每次调用必须被赋值

- 位置实参：在调用函数时，按照从左到右的顺序依次传入的值         特点：按照位置为形参赋值 一一对应

- 关键字实参：在调用函数时，按照key=value的形式传值                  特点：可以打乱顺序 但是仍然能够指名道姓的为指定的形参赋值

  注意：可以混用位置实参与关键字实参，但是

  1.位置实参必须在关键字实参的前面

  2.不能为同一形参重复赋值

- 默认形参：在定义函数时，就已经为某个形参赋值了  特点：调用函数时，可以不用为其赋值

  注意：可以混用位置形参与默认形参，但是

  1.位置形参必须在前面

  2.默认形参的值通常应该是不可变类型

  3.默认形参的值是在函数定义阶段赋值的

5、可变长系列

可变长参数指的是在调用函数时，传入的实参个数不固定，对应着必须有特殊形式的形参来接收溢出的实参

实参无非两种形式

​	溢出的位置实参 --> *

​	溢出的位置关键字实参 --> **

1.*在形参中的应用：\*会将溢出的位置实参合并成一个元组 然后赋值给紧跟其后的那个形参名

2.**在形参中的应用：\*\*会将溢出的关键字实参合并成一个字典 然后赋值给紧跟其后的那个形参名

3.*在实参中的应用：\*后可以跟可以被for循环遍历的任意类型 *会将紧跟其后的那个值打散成位置实参

4.*\*在实参中的应用：\*\*只能跟字典类型 \*\*会将字典打散成关键字实参

了解：命名关键字形参：在*与**中间的形参称之为命名关键字形参  

return详解：

1.函数可以有多个return，但只要执行一次return，整个函数就立即结束，并且将return后的值当做本次调用的结果返回

2.return后的返回值有三种情况

- return值：返回的就是这一个值
- return值1，值2，值3：返回的是一个元组
- 函数内可以没有return 或者return None 或者return  返回都是None

函数对象

本质：函数可以当变量用

1.可以赋值

2.可以当做参数传给另外一个函数

3.可以当做函数的返回值

4.可以当做容器类型的元素

函数嵌套

函数的嵌套调用

函数的嵌套定义

名称空间与作用域

名称空间与作用域的关系是在函数定义阶段（扫描语法时）就确立的，与什么时候调用以及调用位置无关

1.namespaces 名称空间：存放名字的地方 

- 内置名称空间： 存放内置的名字

  生命周期：Python解释器启动则产生 关闭则销毁

- 全局名称空间：存放的是顶级的名字

  生命周期：运行Python文件时则产生，Python文件运行完毕则销毁

- 局部名称空间:存放的是函数内的名字

  生命周期：调用函数则产生，函数调用完毕则销毁

核心：名字的房屋优先级

​	基于当前所在的位置向外查找

​	函数内 --> 外层函数-->  全局-->		内置

LEGB

```python
# 错误示例
x = 111
def f1():
    print(x)
    x = 222
f1()

```

作用域

全局作用域：内置名称空间+全局名称空间  特点：全局存活 全局有效

局部作用域：局部名称空间                           特点：临时存活  局部有效

global：声明变量名来自于全局的

nonlocal:声明变量名来自于外层函数的 不是全局

闭包函数

闭：指的是该函数是定义在函数内的函数

包：指的是该函数引用了一个外层函数作用域的名字

```python
def outter():
    x = 111
    def wrapper():
        print(x)
    return wrapper  # 千万别加括号
f = outter() 
print(f)

def foo():
    x = 222
    f()
    
foo()  # 111
```

为函数体代码传参的方案

- 直接用参数传

- 闭包函数 包给它

  ```python
  def outter(x):
      # x = 222
      def wrapper():
          print(x)
      return wrapper
  f = outter(111)
  f()
  ```

装饰器

1.什么是装饰器 

​	装饰器就是一个用来为被装饰对象添加新功能的工具

2.为何要用装饰器

​	开放封闭原则：一旦软件上线运行之后，应该对修改源代码封闭，对扩展功能开放

​	原则：

​	1.不修改函数内的源代码

​	2.不修改函数的调用方式

装饰器就是在遵循原则1和2的前提下，为被装饰对象添加上新功能

3.如何实现装饰器

1.装饰器

无参装饰器 2层

装饰器的语法糖

有参装饰器 3层

2.迭代器

1、什么是迭代器

​	迭代器指的是迭代取值的工具

​	什么是迭代？

​	迭代就是一个重复的过程，但是每一次重复都是在上一次的基础之上进行的

```python
# nums = [111, 222, 333]
nums = "hello"
def get(l)
    i = 0
    while i < len(l):
        print(l[i])
        i += 1
get(nums)
```



2、为何要用迭代器

​	1.迭代器提供了一种不依赖于索引的、通用的迭代取值方案

​	2.节省内存

3、如何用迭代器

​	可迭代对象

​	1.内置有\_iter\_方法的对象都叫可迭代的对象

掉用可迭代对象的iter方法 返回的是它的迭代器

迭代器对象

​	1.内置有\_next_方法

​	2.内置有\_iter_方法

调用迭代器对象的_iter_方法得到的是它自己 就跟没调用一样

调用迭代器对象的_next_方法返回下一个值 不依赖索引

可以一直调用\_next_直到取干净 则抛出异常stopIteration

只要调用可迭代对象的\_iter_方法就会拿到一个返回值 该返回值就是Python帮我们做好的迭代器

内置的类型都是可迭代对象 文件对象本身就是迭代器对象

3.生成器 自定义的迭代器

yield 与 return的异同

相同点：返回值层面用法一样

不同点：return只能返回值一次 yield可以返回多次

当函数内出现yield关键字 再调用函数并不会触发函数体代码的运行 会返回一个生成器

函数的递归调用：

在调用一个函数的内部又调用自己 递归调用的本质就是一个循环的过程

大前提：递归调用一定要在某一层结束

递归的两个阶段：

1.回溯：向下一层一层挖井

2.递推：向上一层一层返回

面向过程编程：

​	核心是“过程”二字 过程就是解决问题的步骤 即先干啥再干啥后干啥

​	所以基于该思想编写程序就好比设计一条一条的流水线

优点：复杂的问题流程化 进而简单化

缺点：牵一发而动全身 扩展性差





## 八、文件操作

## 九、模块与包

1.什么是模块

​	模块就是一系列功能的集合体

py文件与包都是模块 模块都是被导入使用的 不是直接运行

​	模块大致分为四种类别：

​		1.一个py文件就是一个模块 文件名叫test.py 模块名叫 test

​		2.一个包含有 \__init__.py文件的文件夹称之为包 包也是模块

​		3.使用C编写并链接到Python解释器的内置模块

​		4.已被编译为共享库或DLL的C或C++扩展

模块三种来源：

​	1.自带的模块  lib 标准库 包  内置的模块

​	2.第三方模块  pip install

​	3.自定义的模块

2.为何要用模块

​	1.自带的模块以及第三方模块是为了解决拿来主义 提升开发效率

​	2.自定义模块是为了解决代码冗余问题

3.如何用

首次导入模块发生的事情

1.运行Python文件 创建一个模块的名称空间 将Python文件运行过程中产生的名字都丢到模块的名称空间中

2.在当前名称空间中得到一个名字 该名字是指向模块的名称空间

后续的导入直接引用首次导入的成功 不会重复执行Python文件 不会重复创建名称空间

导入规范

通常情况下所有的导入语句都应该写在文件的开头 然后分为三部分

第一部分：先导入自带的模块

第二部分：导入第三方

第三部分：导入自定义

模块的使用

1.模块的分类

2.模块的使用

import  模块名

模块名.名字

import os, sys, re  可以一行建议多行

import spam as sm



from    模块名  import  名字

from spam import money as m

from spam import *    慎用

\__all__ = ["money", "read1"]

循环导入问题

3.模块的搜索路径优先级

1.内存中已经导入好的

2.内置

3.sys.path

sys.path

4.软件开发的目录规范

soft software  软件

​	bin  binary  可执行文件

​	conf  config	配置文件 参数 

​	core  核心代码

​		

​	db  数据

​	lib  库 模块

​	log  日志

Readme  软件说明

区分py文件的两种用途

1.直接运行

2.被导入使用

包的使用与常用模块的使用

1.包的使用

包含有\__init__的文件夹

首次导入包发生的事情

1.运行包下的\__init__.py文件 创建一个包的名称空间 将\___init__.py运行过程中产生的名称空间

2.在当前名称空间中得到一个名字 该名字是指向包名称空间

2.常用模块的使用

日志模块

logging

logger对象 产生不同级别的日志

fitter

handler对象 控制日志输入的地方

formatter对象 控制日志格式

subprocess模块

json与pickle模块

time与datetime

3.其他模块

random

os

configparser

hashlib

re模块

面向对象编程

1.类与对象的基本使用

2.属性查找

3.绑定方法

4.封装

面向对象的特性

1.绑定与非绑定方法

2.继承

3.多态 len()

4.面向对象高级

​	反射

​	内置方法











## 十、常见模块

- time

- datetime

- os

- sys

- shutil

- logging

- suprocess

- hashlib

  hash算法：传入一段内容会得到一串hash值 hash值的三大特点：

  - 如果传入的内容一样 得到的hash值必然一样
  - 只要采用的算法是固定的 hash值的长度就是固定的 不会随着内容的增多而变长
  - hash值不可逆 不能通过hash值反解出内容是什么

  1+2=> 校验文件的完整性

  1+3=> 加密

- re 模块

  - re.findall("abc","abcxxxabcyyyabc")

  ```python
  import re
  print(re.findall("\w", "hello_123 * -+"))  # ['h', 'e', 'l', 'l', 'o', '_', '1', '2', '3']
  print(re.findall("\W", "hello_123 * -+"))  # [' ', '*', ' ', '-', '+']
  print(re.findall("\s", "hello_123 * -+"))  # [' ', ' ']
  print(re.findall("\S", "hello_123 * -+"))  # ['h', 'e', 'l', 'l', 'o', '_', '1', '2', '3', '*', '-', '+']
  print(re.findall("\d", "hello_123 * -+"))  # ['1', '2', '3']
  print(re.findall("\D", "hello_123 * -+"))  # ['h', 'e', 'l', 'l', 'o', '_', ' ', '*', ' ', '-', '+']
  ```

  ?: 左边那一个字符出现0次或者1次

  *: 左边哪一个字符出现0次或者无穷次

  +: 左边哪一个字符出现1次或者无穷次

  {n,m}: 左边哪一个字符出现n次或m次

  序列化与反序列化

- json&pickle模块

  eval 把字符串里面的代码拿出来运行

## 十一、面向对象

面向对象编程  面向对象就是把相关的东西放在一起

- 核心是对象二字 对象就是一个用来盛放相关的数据与功能的容器
- 类存放的是对象相似的数据与功能的容器
- 基于该思想编写程序就创造一个个的容器
- 优点：扩展性强  封装  继承与之相悖
- 缺点：编程的复杂度提升
- 对象对类只有使用权 没有所有权 修改权
- 对象本身-类-报错
- 对象属性只能由对象来调用 类不能调用对象属性

定义类发送的事

- 立刻运行类体代码
- 将运行过程中产生的名字都丢到类的名称空间中

调用类发生的事情

- 创造一个对象的字典 用来存放对象独有的数据
- 将对象与类建立好关联

调用类的过程  调用类是为了产生对象的 这个过程又叫做实例化  实际存在的一个案例  调用类 类的实例化  对象又叫做实例

- 先创造一个空对象
- 自动触发类内的\__init__函数的运行 将空对象当做第一个参数自动传入
- 返回一个初始化好的对象

对象.属性的查找顺序：先从对象的字典里找 再从类的字典里找

类.属性：从类自己的字典里找

类中的数据属性是直接共享给所有对象用的

类中的函数类可以用 如果类来调用就是一个普通函数 该怎么传参就怎么传

但其实类中的函数是给对象用的 对象来调用就是一个绑定方法 绑定方法的特点是会将调用当做第一个参数自动传入

__开头的属性的特点：

- 并没有真的藏起来 只是变形
- 该变形只在定义阶段 扫描语法的时候执行 此后__开头的属性都不会变形
- 该隐藏对外不对内

为何要隐藏属性

- 隐藏数据属性
- 隐藏函数属性是为了隔离复杂度



Python3统一了类与类型的概念

绑定方法

- 特点：绑定给谁就应该由谁来调用 谁来调用就会将自己当做第一个参数传入

非绑定方法

- 特点：不与类和对象绑定 意味着谁都可以来调用 但无论谁来调用就是一个普通函数 没有自动传参的功能

但凡在类中定义一个函数 默认就是绑定给对象的 应该由对象来调用

会将对象当做第一个参数自动传入

类中定义的函数被classmethod装饰过 就绑定给类 应该由类来调用

类来调用会将类本身当做第一个参数自动传入

类中定义的函数被staticmethod装饰过 就成了一个非绑定的方法即一个普通函数 谁都可以调用

但无论谁来调用就是一个普通函数 没有自动传参的效果

只有绑定方法会自动传参

绑定给类的方法 应该由类来调用 但是对象也可以调用 传进去的参数还是类

绑定给对象的方法 应该由对象来调 但类也可以调 但只是普通函数

继承是为了解决类的冗余问题 类是为了解决对象的冗余问题 

继承 is-a

继承是创建新类的一种方式

新建的类称之为子类

被继承的类称之为父类 基类 超类

类是用来解决对象之间的冗余问题

继承是用来解决类与类之间冗余问题的

Python的继承特点：支持多继承

继承的特点：子类可以遗传父类的属性

在Python2中新式类与经典类之分  在Python3中全都是新式类

但凡是继承了object类的子类以及该子类的子子孙孙类都是新式类

反之就是经典类

继承与抽象

派生

继承的实现原理

菱形继承/死亡钻石：一个子类继承的多条分支最终汇聚一个非object的类上

经典类：深度优先

新式类：广度优先

在子类派生的新方法中如何重用父类的功能的两种方式

- 指名道姓地访问某一个类的函数 不依赖继承

- super()返回一个特殊的对象 该对象会参考发起属性查找的哪一个类的mro列表继续往后找 去当前类的父类中找属性 严格依赖继承

mixins机制  多继承 

组合：一个对象的属性是 另外一个 has-a

多态与多态性

多态：同一种事物有多种形态

import abc

@abc.abstractmethod

抽象基类  

鸭子类型

面向对象高级

内置方法

isinstance  

issubclass  判断一个类是不是另一个类的子类

\__开头并且__结尾的属性会在满足某种条件下自动触发

\__str__

\__del__

```Python
def Single(cls):
    # 存储对象的字典
    __my_class = {}

    # 单例执行的主要代码
    def __innerSingle(*args, **kwargs):
        if cls not in __my_class:
            __my_class[cls] = cls(*args, **kwargs)
        return __my_class[cls]
    return __innerSingle


@Single
class A:
    def __init__(self, x):
        self.x = x


# __innerSingle = Single(A)
# print(__innerSingle)
# print(__innerSingle.__name__)
# print(__innerSingle(1))
# print(__innerSingle(2))
a1 = A(1)
a2 = A(2)
print(a1)  # <__main__.A object at 0x000001925EA13D30>
print(a2)  # <__main__.A object at 0x000001925EA13D30>
```

反射

- hasattr
- getattr
- setattr
- delattr

metaclass 元类

异常处理

1.什么是异常

- 异常是错误发生的信号
- 程序一旦出错就会产生一个异常
- 如果该异常没有被处理 该异常就会抛出来 程序的运行也随即终止

错误分为两种

- 语法错误
- 逻辑错误

2.为何要处理异常

3.如何处理异常

- 语法错误  程序运行前就必须改正确
- 逻辑错误  
  - 针对可以控制的逻辑错误，应该直接在代码层面解决
  - 针对不可以控制的逻辑错误，应该采用try...except...

try...except... 一种异常产生之后的补救措施

try:

​	被监测的代码块1

​	被监测的代码块2

except 异常的类型 as e:

​	处理异常的代码

except (异常的类型1，异常的类型2， 异常的类型3)  as e:

​	处理异常的代码

except Exception:

​	处理异常的代码

else:

​	没有发生异常时要执行的代码

finally:

​	无论异常与否 都会执行该代码 通常用来进行回收资源的操作

断言

raise

自定义异常

```python
# 类中方法:动作
# 种类：普通方法 类方法  静态方法 魔术方法
'''
普通方法格式:
def 方法名(self, [参数, ...]):
    pass
'''
class Phone:
    brand = 'xiaomi'
    price = 4999
    type = 'mate 80'

    # Phone类里面方法:call
    def call(self):
        print('self的值', self)
        print("正在打电话...")


phone1 = Phone()
print(phone1)  # <__main__.Phone object at 0x0000023EF435DE20>
# print(phone1.brand)
phone1.call()  # self的值 <__main__.Phone object at 0x0000023EF435DE20>
phone2 = Phone()
phone2.call()
print(phone2)
```

## 十二、网络编程

学习套接字编程的目的是为了开发一个C/S或者B/S架构的软件

client   客户端         server  服务端

browser  浏览器      server  服务端

互联网 =  物理连接介质 + 通信协议

应用层  表示层  会话层  传输层  网络层  数据链路层  物理层

- 应用层  http ftp mail  CS架构软件可以自定义协议 
- 传输层  tcp/udp  端口 1024-65535
- 网络层  IP 数据包
- 数据链路层  分组  ethernet 以太网协议  数据帧
- 物理层  电信号 高低电压 10

网卡 网线 交换机（局域网  mac地址）  IP地址 （IPV4  IPV6）数据包  路由器

选择大于努力

mac地址用来标识这台计算机在局域网的那个位置

IP+MAC可以用来标识全世界一台独一无二的计算机

IP+MAC+端口（port）标识全世界范围内独一无二的一个基于网络通信的软件

orp协议：ip解析成mac地址

ip+port  标识全世界范围内独一无二的一个基于网络通信的软件

套接字  传输层 网络层  数据链路层

基础知识掌握的越多越好

mac 网关 公网 ip 路由器 ip

套接字

解决粘包问题

基于Udp协议的套接字

socketserver模块

并发编程

进程:  进程指的是程序的运行过程，是一个动态的概念  

​		也可以说是操作系统干活的过程 ，

​		说白了就是操作系统控制硬件来运行应用程序的过程

进程是操作系统最核心的概念，没有之一，研究进程就是在研究操作系统

程序：程序就是一系列的代码文件，是一个静态的概念

操作系统复习

第一代计算机（1940-1955）：真空管和穿孔卡片  

特点：

- 没有操作系统概念
- 所有的程序设计都是直接操控硬件

缺点：

- 浪费了计算机资源
- 串行

优点：

- 独享计算机资源

第二代计算机（1955-1965）批处理操作系统

优点：批处理 节省了机时

缺点：

- 整个流程需要人参与控制
- 串行
- 不能独享 要等待同批次运行完成

第三代计算机（1965-1980）：集成电路芯片和多道程序设计

IBM  system/360  服务器

多道技术

- 空间上的复用

- 时间上的复用

  cpu在多个进程/任务之间快速切换

  ​	1.遇到io切换

  ​	2.占用时间过长切换

多个进程的数据可以共享资源

SPOOLING

分时操作系统

MULTICS

UNIX

posix 可移植的操作系统接口Portable Operating System Interface

GNU

UNIX -  minix  - Linux

第四代计算机（1980-至今）：个人计算机

cpu  多核 多线程

并发  同一时间只能运行一个 速度快 感觉 时间空间上的复用 多个任务看起来是同时运行的

并行  多个任务是真正意义上的同时运行

串行  一个任务运行完毕后才能开启下一个任务然后运行

逻辑删除与物理删除

一个任务运行的三种状态：

运行态

阻塞态

就绪态

提交任务的两种方式

同步  调完一个结果后再调一个

异步  同时调

多进程

多线程

IO

1.僵尸进程与孤儿进程

- 僵尸进程：是Linux操作系统中国一种特殊的数据结构
- 僵尸进程指的是将该进程的资源释放掉（CPU、内存、打开的文件），但是会保留该进程的状态信息，比如pid
- 在linux系统中所有的子进程在死后都会进入僵尸进程的状态
- PID 进程在操作系统中的唯一编号
- kill -CHLD  父进程的pid
- kill -9  父进程的pid 
- 孤儿进程：父进程先死了，子进程就会被pid为1的进程接管，成为孤儿进程
- top
- ps aux | grep Z  僵尸
- ps -elf | grep Z

2.守护进程

守护进程守护的是主进程的代码，而不是生命周期

主进程代码运行完毕，守护进程就会结束

3.互斥锁

4.IPC机制

- 队列  管道+互斥锁实现
- 管道

5.信号量

能从本地拿尽量不要从网络拿  能从内存拿尽量不要从硬盘拿

python 运行程序的原理 .py内容丢给解释器 运行解释器

Python性能的优劣很大程度上取决于解释器的优劣

taskkill /F /PID  1764  杀进程 win

ps  aux | grep 18472

- 生成者消费者模型  编程套路
- 线程理论
  - 线程：一个流水线的运行过程（代码）  进程是一个程序的运行过程（资源）
  - 线程：进程内代码的运行过程
  - 线程是一个执行单位，CPU执行的就是线程
  - 进程是一个资源单位
  - 同一进程下的多个线程共享数据
  - 《现代操作系统》
- 线程 vs j进程
  - 同一进程下的多个线程共享该进程的内存资源
  - 开启子线程的开销要远远小于开启子进程
- 多线程
  - 多线程共享一个进程的地址空间
  - 线程为轻量级的进程
- 开启线程的两种方式
- 线程对象的相关属性和方法
- 主线程要等子线程结束它才结束
- 守护线程守护的是生命周期
- 互斥锁
- 信号量
- 死锁与递归锁
- 事件Event  线程之间配合工作
- 定时器

用户态：应用程序

内核态：操作系统

混合实现：用户级与内核级的多路复用，内核同一调度内核线程，每个内核线程对应n个用户线程

Python用的是操作系统的调度模式  内核

- 线程queue

- GIL全局解释器锁（\*\*\**\*\*）
- 进程池与线程池
- 协程  一个任务阻所有都阻  应用程序  轻量级  速度快
- nginx

操作系统并发的单位是线程

- IO模型

Python多线程可以并发

在cpython解释器中，同一个进程下开启的多线程，同一时刻只能有一个线程执行，无法利用多核优势

Python利用多核优势开启多进程

纯计算  多进程

IO密集型  多线程



























