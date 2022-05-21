---
title: argparse的基本用法
categories:
  - - 工作技能
    - python笔记
date: 2020-01-06 16:35:08
tags:
  - python
  - argparse
---
### 用途及原理
为python脚本运行时所添加额外命令行参数提供解释。大致原理是为一个parser对象添加一些属性，然后这个parser对象在接受一系列参数（默认是命令行参数）之后，就拥有了一系列属性，然后程序就可以用这些属性来做事了。就是说，信息从命令行传递到了脚本内。

### 示例
```
# 创建对象
import argparse
parser = argparse.ArgumentParser()

# 添加属性
parser.add_argument('integers', //位置型属性，与添加顺序有关
		    metavar='N',  //帮助文档中的示例显示内容
		    type=int,  //这个属性的类型
		    nargs='+', //这个属性的值为自适应长度的列表
                    help='an integer for the accumulator') //与这个属性相关的帮助信息
parser.add_argument('--sum',  //flag型属性，位置无关
                    dest='accumulate', //属性的正式名称，有了这个之后sum这个名称就不能用了，没有的话sum是正式名称
		    action='store_const', //与下面的const配合使用，定义--sum在命令行中出现时，sum属性的赋值行为，这里是accumulate = sum
                    const=sum, 
		    default=max, //定义当--sum未出现时sum属性的赋值行为，这里是accumulate = max。
                    help='sum the integers (default: find the max)')

# 接受参数
args = parser.parse_args()

# 使用参数
print(args.accumulate(args.integers)) //解析后等价于print(sum([1,2,3,4]))
```

### 参考链接
[1] https://towardsdatascience.com/a-simple-guide-to-command-line-arguments-with-argparse-6824c30ab1c3
[2] https://www.huaweicloud.com/articles/208c06ca1f4aba8dfd9219c6e2c72b23.html
[3] https://blog.csdn.net/liuweiyuxiang/article/details/82918911
[4] https://docs.python.org/zh-cn/3/library/argparse.html
