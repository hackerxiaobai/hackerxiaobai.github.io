---
title: unittest单元测试
date: 2019-09-02 15:23:23
tags: 
  - unittest
  - 单元测试
---

# 为什么需要单元测试

[转载该博客，记录主要是自己日后查看](https://www.cnblogs.com/mapu/p/8549824.html)

> 1. 单元测试有没有必要？为什么需要单元测试？...... 理由简直多到不要不要的。总而言之，单元测试肯定是需要的。
> 2. 对于算法工程师需要有单元测试吗？答案是：需要
> 3. 最近老大要求我去对我负责的某个算法模块给出详细的单元报告，所以就写了一下这个报告

<!--more-->
# 单元测试模块unittest
> 1. unittest中最核心的四个概念是：test case, test suite, test runner, test fixture。
> 2. unittest的静态类图

{% asset_img 1.png 1.png  %}

> 一个TestCase的实例就是一个测试用例
> 多个测试用例集合在一起，就是TestSuite，而且TestSuite也可以嵌套TestSuite
> TestLoader是用来加载TestCase到TestSuite中
> TextTestRunner是来执行测试用例的，其中的run(test)会执行TestSuite/TestCase中的run(result)方法
> 对一个测试用例环境的搭建和销毁，是一个fixture


**写好TestCase，然后由TestLoader加载TestCase到TestSuite，然后由TextTestRunner来运行TestSuite，运行的结果保存在TextTestResult中，我们通过命令行或者unittest.main()执行时，main会调用TextTestRunner中的run来执行，或者我们可以直接通过TextTestRunner来执行用例。这里加个说明，在Runner执行时，默认将执行结果输出到控制台，我们可以设置其输出到文件，在文件中查看结果（你可能听说过HTMLTestRunner，是的，通过它可以将结果输出到HTML中，生成漂亮的报告，它跟TextTestRunner是一样的，从名字就能看出来，这个我们后面再说）**

# 代码样例

```python

# mathfunc.py

def add(a, b):
    return a+b

def minus(a, b):
    return a-b

def multi(a, b):
    return a*b

def divide(a, b):
    return a/b
```

```python 
# test_mathfunc.py

import unittest
from mathfunc import *


class TestMathFunc(unittest.TestCase):
    """Test mathfuc.py"""

    def test_add(self):
        """Test method add(a, b)"""
        self.assertEqual(3, add(1, 2))
        self.assertNotEqual(3, add(2, 2))

    def test_minus(self):
        """Test method minus(a, b)"""
        self.assertEqual(1, minus(3, 2))

    def test_multi(self):
        """Test method multi(a, b)"""
        self.assertEqual(6, multi(2, 3))

    def test_divide(self):
        """Test method divide(a, b)"""
        self.assertEqual(2, divide(6, 3))
        self.assertEqual(2.5, divide(5, 2))

if __name__ == '__main__':
    unittest.main()

```

```python 
# test_suite.py

import unittest
from test_mathfunc import TestMathFunc

if __name__ == '__main__':
    suite = unittest.TestSuite()

    tests = [TestMathFunc("test_add"), TestMathFunc("test_minus"), TestMathFunc("test_divide")]
    suite.addTests(tests)

    runner = unittest.TextTestRunner(verbosity=2)
    runner.run(suite)

'''
输出到文件
if __name__ == '__main__':
    suite = unittest.TestSuite()
    suite.addTests(unittest.TestLoader().loadTestsFromTestCase(TestMathFunc))

    with open('UnittestTextReport.txt', 'a') as f:
        runner = unittest.TextTestRunner(stream=f, verbosity=2)
        runner.run(suite)

'''
```

# 依赖关系
> 1. 如下场景：支付是一个独立的接口，由其它开发提供，根据支付的接口返回状态去显示失败，还是成功，这个是你需要实现的功能
> 2. 也就是说你写一个b功能，你的同事写一个a功能，你的b功能需要根据a功能的结果去判断，然后实现对应的功能。这就是存在依赖关系，你同事开发的进度你是无法控制的
你要是等他开发完了，你再开发，那你就坐等加班吧.
> 3. 以下是自己写的 zhifu_statues()函数功能，大概设计如下

```python 
def zhifu():
    '''假设这里是一个支付的功能,未开发完
    支付成功返回：{"result": "success", "reason":"null"}
    支付失败返回：{"result": "fail", "reason":"余额不足"}
    reason返回失败原因
    '''
    pass

def zhifu_statues():
    '''根据支付的结果success or fail，判断跳转到对应页面'''
    result = zhifu()
    print(result)
    try:
        if result["result"] == "success":
            return "支付成功"
        elif result["result"] == "fail":
            print("失败原因：%s" % result["reason"])
            return "支付失败"
        else:
            return "未知错误异常"
    except:
        return "Error, 服务端返回异常!"
```

### 单元测试用例设计
```python 
from unittest import mock
import unittest
import temple
# 作者：上海-悠悠 QQ交流群：588402570

class Test_zhifu_statues(unittest.TestCase):
    '''单元测试用例'''
    def test_01(self):
        '''测试支付成功场景'''
        # mock一个支付成功的数据
        temple.zhifu = mock.Mock(return_value={"result": "success", "reason":"null"})
        # 根据支付结果测试页面跳转
        statues = temple.zhifu_statues()
        print(statues)
        self.assertEqual(statues, "支付成功")

    def test_02(self):
        '''测试支付失败场景'''
        # mock一个支付成功的数据
        temple.zhifu = mock.Mock(return_value={"result": "fail", "reason": "余额不足"})
        # 根据支付结果测试页面跳转
        statues = temple.zhifu_statues()
        self.assertEqual(statues, "支付失败")

if __name__ == "__main__":
    unittest.main()
```
