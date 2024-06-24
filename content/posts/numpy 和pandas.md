---
title: "numpy 和pandas "
date: 2022-06-28
categories:
- tranquilpeak
- features
tags:
- numpy
- pandas
# keywords:
# - pandas
# - numpy

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### numpy 和pandas 

#### 1: numpy

```
numpy是 Python 科学计算的基本包，几乎所有用 Python 工作的科学家都利用了的强大功能。
（https://docs.scipy.org/doc/numpy/user/quickstart.html）  官方文档
numpy.array() 函数可以用来创建函数 
numpy.array(object, dtype =None, copy =True, order =None, subok =False, ndmin =0)

object  :  数组或嵌套的数列  
dtype   :  数组元素的数据类型，可选 
copy    :  对象能否复制，可选 
order   :  创建数组的样式，C为行方向，F为列方向，A为任意方向，默认为A
subok   :  默认返回一个与基类类型一致的数组  
ndmin   :  指定生成数组的最小维度


Numpy数组和Python列表性能对比：
比如我们想要对一个Numpy数组和Python列表中的每个素进行求平方。那么代码如下：

import time

t1 = time.time()
a = []
for x in range(100000):
    a.append(x**2)
t2 = time.time()
t = t2 - t1
print(t)
#### 0.03940296173095703


t3 = time.time()
b = np.arange(100000)**2
t4 = time.time()
print(t4-t3)

###0.0006630420684814453
```



#### 2: pandas

```
https://github.com/pandas-dev/pandas 官方文档

-----------------------pandas 概念-----------------------

① pandas一般解决表格型的数据、二维的。

② pandas是专门为处理表格和混杂数据设计的，而Numpy更适合处理统一数值数据。

③ pandas主要数据结构：Series 和 DataFrame

先说一说 dataframe  目前 我有一个需求 是将 一个分词 结果数据 列表 进行聚合 去重 并将 长度小于1 的单词 进行过滤

def pandas_clean_count(datas):
  """
  :param datas:  原始 分词数据
  :return:  去掉长度小于1的 词 然后聚合结果
  """
  df = pd.DataFrame({'datas': datas})
  df = df[df['datas'].str.len() > 1]
  df = df.groupby(by=['datas']).size()
  data_dict = df.to_dict()
  return data_dict
##### 测试
data_list = ['1','知乎','知乎','上海','知乎','知乎','知乎','知乎','知乎','测试']
##### 结果
{'上海': 1, '测试': 1, '知乎': 7}

pandas 还可以 通过 time 时间字段来进行排序
 df = pd.DataFrame([int(k) for k in keys], columns=['time']).sort_values(
            by='time', ascending=False)
另外pandas 还可以通过读取excel 进行操作 
results = pd.read_excel('xxxxx.xlsx', usecols=[0])
```

