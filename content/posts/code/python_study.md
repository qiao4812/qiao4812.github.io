---
title: "Python笔记"
date: 2023-02-12T18:40:06+08:00
draft: false
tags: ["Python"]
categories: ["Python"]

---

## 1.今日内容大纲

- pycharm的安装以及简单使用
  - 辅助开发软件，代码逐行调试，设置高端，书写代码时会提示、debug模式，最好用的pycharm
- 格式化输出
- while循环
- 运算符
- 编码的初识

## 2.昨日内容回顾

- 编译型与解释型
  - 编译型：一次性编译成二进制，再执行
    - 优点：执行效率高
    - 缺点：不能跨平台，开发效率低
    - 代表语言：C
  - 解释型：逐行解释成二进制，再执行
    - 优点：可以跨平台，开发效率高
    - 缺点：执行效率低
    - 代表语言：python
- 变量
  - 数字，字母，下划线任意组合
  - 不能以数字开头
  - 不能用python的关键字：print if ...
  - 不能使用中文
  - 描述性
  - 区分变量与数据类型的区别
- 常量
  - 一直不变的量，与变量几乎一样
- 注释：解释说明
- 基础数据类型
  - int  +-*/%**...  数字整型
  - str str + *int  字符串
  - bool  True  False  布尔值
  - float   浮点数
- 用户输入input
  - name = input('>>>')  print(type(name))
- if
  - if 条件
  - if else
  - if elif elif ...
  - if elif elif ... else
  - if 嵌套

### 3.今日内容

1.while循环

- why：大气循环 吃饭 上课 睡觉 日复一日 歌曲列表循环 程序中：输入用户名密码

- what：while 无限循环

- how:

  1. 基本结构

     ```python
     while 条件:
         循环体
     ```

  2. 初识循环

     ```python
     while True:
         print('狼的诱惑')
         print('我们不一样')
         print('月亮之上')
         print('庐州月')
         print('人间')
     ```

  3. 基本原理

     1.先判断条件是否为True

     2.如果是True进入循环体

     3.执行到循环体的底部

     4.继续判断条件，条件成立，进入循环体

     ......

  4. 循环如何终止？

     1. 改变条件

        ```python
        flag = True
        while flag:
            print('狼的诱惑')
            print('我们不一样')
            print('月亮之上')
            flag = False
            print('庐州月')
            print('人间')
        ```

     2. break

     ```python
     # break：循环中遇到break直接退出循环
     # while True:
     #     print('狼的诱惑')
     #     print('我们不一样')
     #     print('月亮之上')
     #     break
     #     print('庐州月')
     #     print('人间')
     ```

     3. 系统命令(今天不讲)

     4. continue

        ```python
        # continue  退出本次循环 继续下一次循环
        # flag = True
        # while flag:
        # # while True:
        #     print(111)
        #     print(222)
        #     flag = False
        #     continue
        #     print(333)
        ```

        ```python
        # while else: while 循环如果被break打断，则不执行else语句
        count = 1
        while count < 5:
            print(count)
            if count == 2:
                break
            count = count + 1
        else:
            print(666)
        ```

- where：你需要重复之前的动作，输入用户名密码，考虑到while循环

2.格式化输出

```python
# 制作一个公共的模板
# 让一个字符串的某些位置变成动态可传入的
# 格式化输出
name = input('请输入你的姓名：')
age = input('请输入你的年龄：')
job = input('请输入你的工作：')
hobby = input('请输入你的爱好：')
# % 占位符 s --> str
msg = '''----------- info of %s ----------
Name : %s
Age : %s
Job : %s
Hobbie : %s
-------------------- end ----------------''' % (name,name,age,job,hobby)
print(msg)
```

- 当你遇到这样的需求：字符串中想让某些位置变成动态可传入的，首先要考虑到格式化输出

  ```python
  # 坑:在格式化输出中，%只想表示一个百分号，而不是作为占位符使用
  # mag = '我叫%s,今年%s,学习进度%1' % ('太白金星',18)
  msg = '我叫%s,今年%s,学习进度%%1' % ('太白金星',18)
  print(msg)
  # ValueError: incomplete format
  ```

3.运算符：

- 算术运算符  + - * /  % **  //

- 比较运算符  > < >= <= !=

- 赋值运算符 = += -= *= /= **=  %= //=

- 逻辑运算符  and 与  or 或  not 非

  - 在没有()的情况下 优先级：not > and > or  
  - 同一优先级从左至右依次计算

- 成员运算符

  ```python
  i1 = 2
  i2 = 3
  print(i1 ** i2)
  print(10 // 3)
  print(10 % 3)
  ```

  非0即True, 0为False

  ```python
  # str ---> int : 只能是纯数字组成的字符串
  s1 = '00100'
  print(int(s1))
  # int ---> str
  i1 = 100
  print(str(i1),type(str(i1)))
  
  # int ---> bool : 非零即True, 0为False
  i = 0
  print(bool(i))
  # bool ---> int
  print(int(True)) # 1
  print(int(False)) # 0
  ```

4. 编码的初识

计算机存储文件，存储数据，以及将一些数据信息通过网络发送出去，存储发送数据什么内容？底层都是01010101

ASCII码：只包含英文字母，数字，特殊字符  7位  128位

8bit == 1byte

gbk：只包含英文字母，数字，特殊字符 和中文 国标

一个英文字母：1byte

一个中文：2byte

Unicode：万国码：把世界上所有的文字都记录到这个密码本

起初一个字符用2个字节表示

后来为了涵盖全部文字：4个字节 浪费空间 浪费资源

Utf-8升级：最少用8bit1个字节表示一个字符

a 1字节  欧洲 2 字节  中  3个字节

8bit = 1byte

1024byte = 1KB

1024KB = 1MB

1024MB = 1GB

1024GB = 1TB

明日内容：

1. 二进制与十进制之间的转换
2. str bool int 转换
3. str具体操作方法：索引切片步长，常用操作方法
4. for循环

### day03课程大纲

一、今日内容大纲

1. 基础数据类型总览
2. int
3. bool
4. str
   - 索引、切片
   - 常用操作方法
5. for循环

二、

`

## 类

> ### 面向对象编程

- 通过类获取一个对象的过程 - 实例化

- 类名()会自动调用类中的\__init__方法

- 类和对象之间的关系？

  - 类  是一个大范围 是一个模子 它约束了事务有哪些属性  但是不能约束具体的值
  - 对象 是一个具体的内容 是模子的产物 它遵循了类的约束 同时给属性赋上具体的值

- 类有一个空间 存储的是定义在class中的所有名字

- 每一个对象又拥有自己的空间 通过对象名.\__dict__就可以查看这个对象的属性和值

  ```python
  print(alex.name)  # print(alex.__dict__['name'])  属性的查看
  alex.name = 'alexsb'  # 属性的修改
  alex.money = 100  # 属性的增加
  del alex.money  # 属性的删除
  ```

- 练习类的创建和实例化

  ```python
  d = ['k':'v']
  print(d,id(d))
  d['k'] = 'vvvv'
  print(d,id(d))
  ```

- 修改列表\字典中的某个值，或者是对象的某一个属性 都不会影响这个对象\字典\列表所在的内存空间

```python
# 实例化所经历的步骤
 1.类名() 之后的第一个事儿：开辟一块儿内存空间
 2.调用__init__把空间的内存地址作为self参数传递到函数内部
 3.所有的这一个对象需要使用的属性都需要和self关联起来
 4.执行完init中的逻辑之后，self变量会自动的被返回到调用处(发生实例化的地方)
```

- dog类 实现狗的属性 名字 品种 血量 攻击力 都是可以被通过实例化被定制的

  ```python
  class Dog():
      def __init__(self,name,blood,aggr,kind):
          self.dog_name = name
          self.hp = blood
          self.ad = aggr
          self.kind = kind
          
  小白 = Dog('小白',5000,249,'柴犬')
  print(小白.dog_name)
  print(小白.__dict__)
  ```

- 对象/实例 = 类名()    =>  实例化的过程

- 对象的属性/实例变量

- 类中的方法(函数) 有一个必须传的参数    self 对象

- hasattr  getattr setattr delattr 反射

  - 实例对象
  - 类
  - 本模块 sys.modules[\__name__]
  - 其他模块

  ```python
  if hasattr(obj,'name'):
      getattr(obj,'name')
  ```

- 找到tbjx对象 的c类 实例化一个对象

  ```pyhton
  obj = getattr(tbjx,'C')()
  ```

- 找到tbjx对象 的c类 通过对c类这个对象使用反射取到area

  ```python
  print(getattr(tbjx.C,'area'))
  ```

- 找到tbjx对象 的c类 实例化一个对象 对对象进行反射取值

  ```python
  obj = getattr(tbjx,'C')('赵海狗')
  print(obj.name)
  print(getattr(obj,'name'))
  ```

- 一次执行多个函数

  ```python
  def func1():
      print('in func1')
      
  func_lst = [f'func{i}' for i in range(1,5)]
  for func in func_lst:
      getattr(sys.modules[__name__],func)()
  ```

- 反射应用

  ```python
  class User:
      user_list = [('login','登录'),('register','注册'),('save','存储')]
  
      def login(self):
          print('欢迎来到登录页面')
  
      def register(self):
          print('欢迎来到注册页面')
  
      def save(self):
          print('欢迎来到存储页面')
  
  
  # choose_dic = {
  #     1: User.login,
  #     2: User.register,
  #     3: User.save,
  # }
  
  while 1:
      choose = input('请输入序号:\n1:登录\n2:注册\n3:存储\n').strip()
      obj = User()
      # choose_dic[int(choose)](obj)
      getattr(obj,obj.user_list[int(choose)-1][0])()
  # getattr(obj,obj.user_list[int(1)-1][0])()
  ```

> ###函数与方法的区别

```python
def func1():
    pass

class A:
    def func(self):
        pass

# 1.通过打印函数名的方式区别什么是方法，什么是函数 (了解)
# print(func1)  # <function func1 at 0x000001EE93C3F040>
# # 通过类名调用的类中的实例方法叫做函数
# print(A.func)  # <function A.func at 0x000001EEAE73D670>
# # 通过对象调用的类中的实例方法叫做方法
# obj = A()
# print(obj.func)  # <bound method A.func of <__main__.A object at 0x000001A98AE13C10>>

# 2.可以借助模块判断是方法还是函数
from types import FunctionType
from types import MethodType


# 总结: 如何判断类中的是方法还是函数
# 函数都是显性传参 方法都是隐性传参
```

- python中一切皆对象，类在某种意义上也是一个对象，python中自己定义的类，以及大部分内置类，都是由type元类(构建类)实例化得来的
- python中一切皆对象，函数在某种意义上也是一个对象，函数这个对象是从FunctionType这个类实例化出来的
- python中一切皆对象，方法在某种意义上也是一个对象，方法 这个对象是从MethodType这个类实例化出来的

```python
class A:
    
    company_name = '老男孩教育'  # 静态变量（静态字段）
    __iphone = '135'  # 私有静态变量（私有静态字段）
    
    def __init__(self,name,age):  # 特殊方法
        self.name = name  # 对象属性（普通字段）
        self.__age = age  # 私有对象属性（私有普通字段）
        
    def func1(self):  # 普通方法
        pass
    
    def __func(self):  # 私有方法
        print(666)
        
    @classmethod  # 类方法
    def class_func(cls):
        """定义类方法 至少有一个cls参数"""
        print('类方法')
        
    @staticmethod  # 静态方法
    def static_func():
        """定义静态方法 无默认参数"""
        print('静态方法')
        
    @property  # 属性
    def prop(self):
        pass
```

> ### 特殊的双下方法

- 特殊的双下方法：原本是开发python这个语言的程序员用的，源码中使用的，我们不能轻易使用。慎用！

- 双下方法：你不知道你干了什么就触发了某个双下方法

  ```python
  # __len__  使用len() 触发
  class B:
      def __init__(self,name,age):
          self.name = name
          self.age = age
  
      def __len__(self):
          return len(self.__dict__)
  
  b = B('liye',28)
  print(len(b))
  
  # __hash__
  # __str__  必须return一个字符串类型   打印时触发print  打印对象触发  直接str转化也可以触发 
  
  # __repr__  str优先级高于repr
  print('我叫%s' % ('alex'))
  print('我叫%r' % ('alex'))
  print('我叫{}'.format('alex'))
  a = 'alex'
  print(f'我叫{a}')
  print(repr('ting'))
  
  # __call__  对象后面加括号，触发执行  这个对象从属于类(父类)的__call__方法
  # __eq__ 
  # __del__ 析构方法 当对象在内存中被释放时 自动触发执行
  # __new__  new一个对象  对象是object类的__new__方法 产生了一个对象  构造方法
  # 类名加括号先要触发object的__new__方法，object的__new__产生一个对象空间并返回给类名加括号，再触发__init__给对象封装属性 
  # 类名() 
  # 1. 先触发 object的__new__方法，此方法在内存中开辟一个对象空间
  # 2. 执行__init__方法，给对象封装属性
  # __init__ 不允许有return
  
  # 单例模式  python中的设计模式  方便对实例个数的控制
  # 一个类只允许实例化一个对象
  class A:
      __instance = None
  
      def __new__(cls, *args, **kwargs):
          if not cls.__instance:
              cls.__instance = object.__new__(cls)
              # return cls.__instance
          # else:
          #     return cls.__instance
          return cls.__instance
  
  obj = A()
  print(obj)
  obj1 = A()
  print(obj1)
  ```
