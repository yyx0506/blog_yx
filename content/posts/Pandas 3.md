---
title: "pandas 3 "
date: 2023-03-02
categories:
- tranquilpeak
- features
tags:
- pandas
# keywords:
# - pandas
# - numpy

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### Pandas 3

中文文档地址：  https://www.pypandas.cn/ 

#### 1: 常用的基本函数

```
接下来的介绍将会使用一份learn_pandas.csv的虚拟数据集，它记录了四所学校学生的体测个人信息

print(df.columns)
Index(['School', 'Grade', 'Name', 'Gender', 'Height', 'Weight', 'Transfer',
       'Test_Number', 'Test_Date', 'Time_Record'],
      dtype='object')
上述列名依次代表学校、年级、姓名、性别、身高、体重、是否为转系生、体测场次、测试时间、1000米成绩，本章只需使用其中的前七列。

df = df[df.columns[:7]]


-----------------------------------汇总函数----------------------------------
head, tail函数分别表示返回表或者序列的前n行和后n行，其中n默认为5  这个跟日志时一样的

print(df.head(2))
print(df.tail(3))
分别获取 前两行 后三行 
df.info()
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 200 entries, 0 to 199
Data columns (total 7 columns):
 #   Column    Non-Null Count  Dtype  
---  ------    --------------  -----  
 0   School    200 non-null    object 
 1   Grade     200 non-null    object 
 2   Name      200 non-null    object 
 3   Gender    200 non-null    object 
 4   Height    183 non-null    float64
 5   Weight    189 non-null    float64
 6   Transfer  188 non-null    object 
dtypes: float64(2), object(5)
memory usage: 11.1+ KB

print(df.describe())
           Height      Weight
count  183.000000  189.000000
mean   163.218033   55.015873
std      8.608879   12.824294
min    145.400000   34.000000
25%    157.150000   46.000000
50%    161.900000   51.000000
75%    167.500000   65.000000
max    193.900000   89.000000

info, describe只能实现较少信息的展示，如果想要对一份数据集进行全面且有效的观察，特别是在列较多的情况下，推荐使用pandas-profiling包。


-----------------------------------特征函数----------------------------------
在Series和DataFrame上定义了许多统计函数，最常见的是sum, mean, median, var, std, max, min。例如，选出身高和体重列进行演示：
分别获取 数据综合 平均值 中位数 方差 标准差 最大值以及最小值等
df_demo = df[['Height', 'Weight']]
print(df_demo.mean())
df.count()          #非空元素计算
df.min()            #最小值
df.max()            #最大值
df.idxmin()         #最小值的位置，类似于R中的which.min函数
df.idxmax()         #最大值的位置，类似于R中的which.max函数
df.quantile(0.1)    #10%分位数
df.sum()            #求和
df.mean()           #均值
df.median()         #中位数
df.mode()           #众数
df.var()            #方差
df.std()            #标准差
df.mad()            #平均绝对偏差
df.skew()           #偏度
df.kurt()           #峰度
df.describe()       #一次性输出多个描述性统计指标
df.abs()            #求绝对值
df.prod             #元素乘积
df.cumsum           #累计和
df.cumprod          #累计乘积

此外，需要介绍的是quantile, count, idxmax这三个函数，它们分别返回的是分位数、非缺失值个数、最大值对应的索引
为了更一般化，在计算的过程中，我们考虑p分位。当p=0.25 0.5 0.75 时，就是在计算四分位数
print(df_demo.quantile(0.5))
print(df_demo.count())
print(df_demo.idxmax())

上面这些所有的函数，由于操作后返回的是标量，所以又称为聚合函数，它们有一个公共参数`axis`，默认为0代表逐列聚合，如果设置为1则表示逐行聚合：
print(df_demo.mean(axis=1).head())
0    102.45
1    118.25
2    138.95
3     41.00
4    124.00
dtype: float64

-----------------------------------唯一值函数-----------------------------------
对序列使用unique和nunique可以分别得到其唯一值组成的列表和唯一值的个数：
print(df['School'].unique())
print(df['School'].nunique())
['Shanghai Jiao Tong University' 'Peking University' 'Fudan University'
 'Tsinghua University']
4
我们可以获取到 school 去重后的唯一列表 和 唯一值的个数

value_counts可以得到唯一值和其对应出现的频数：
print(df['School'].value_counts())

如果想要观察多个列组合的唯一值，可以使用drop_duplicates。其中的关键参数是keep，默认值first表示每个组合保留第一次出现的所在行，last表示保留最后一次出现的所在行，False表示把所有重复组合所在的行剔除。
df_demo = df[['Gender','Transfer','Name']]
print(df_demo.drop_duplicates(['Gender', 'Transfer']))

    Gender Transfer            Name
0   Female        N    Gaopeng Yang
1     Male        N  Changqiang You
12  Female      NaN        Peng You
21    Male      NaN   Xiaopeng Shen
36    Male        Y    Xiaojuan Qin
43  Female        Y      Gaoli Feng

print(df_demo.drop_duplicates(['Gender', 'Transfer'], keep='last'))
     Gender Transfer            Name
147    Male      NaN        Juan You
150    Male        Y   Chengpeng You
169  Female        Y   Chengquan Qin
194  Female      NaN     Yanmei Qian
197  Female        N  Chengqiang Chu
199    Male        N     Chunpeng Lv

print(df_demo.drop_duplicates(['Name', 'Gender'], keep=False).head() )# 保留只出现过一次的性别和姓名组合
   Gender Transfer            Name
0  Female        N    Gaopeng Yang
1    Male        N  Changqiang You
2    Male        N         Mei Sun
4    Male        N     Gaojuan You
5  Female        N     Xiaoli Qian

print(df['School'].drop_duplicates()) # 在Series上也可以使用)
0    Shanghai Jiao Tong University
1                Peking University
3                 Fudan University
5              Tsinghua University
Name: School, dtype: object

此外，duplicated和drop_duplicates的功能类似，但前者返回了是否为唯一值的布尔列表，其keep参数与后者一致。其返回的序列，把重复元素设为True，否则为False。 drop_duplicates等价于把duplicated为True的对应行剔除。
print(df['School'].duplicated()) # 在Series上也可以使用)

-----------------------------------替换函数-----------------------------------

一般而言，替换操作是针对某一个列进行的，因此下面的例子都以Series举例。pandas中的替换函数可以归纳为三类：映射替换、逻辑替换、数值替换。其中映射替换包含replace方法、第八章中的str.replace方法以及第九章中的cat.codes方法，此处介绍replace的用法。
在replace中，可以通过字典构造，或者传入两个列表来进行替换：

print(df['Gender'].replace({'Female':0, 'Male':1}).head())
print(df['Gender'].replace(['Female', 'Male'], [0, 1]).head())
两者都是实现了一个功能 就是将 Female替换成0 Male 替换成 1

另外，replace还有一种特殊的方向替换，指定method参数为ffill则为用前面一个最近的未被替换的值进行替换，bfill则使用后面最近的未被替换的值进行替换。从下面的例子可以看到，它们的结果是不同的：
0    a
1    a
2    b
3    b
4    b
5    b
6    a

0    a
1    b
2    b
3    a
4    a
5    a
6    a

虽然对于replace而言可以使用正则替换，但是当前版本下对于string类型的正则替换还存在bug，因此如有此需求，请选择str.replace进行替换操作

逻辑替换包括了where和mask，这两个函数是完全对称的：where函数在传入条件为False的对应行进行替换，而mask在传入条件为True的对应行进行替换，当不指定替换值时，替换为缺失值。
s = pd.Series([-1, 1.2345, 100, -50])
print(s.where(s<0))
0    -1.0
1     NaN
2     NaN
3   -50.0
dtype: float64

print(s.where(s<0, 100))
0     -1.0
1    100.0
2    100.0
3    -50.0
dtype: float64

print(s.mask(s<0))
0         NaN
1      1.2345
2    100.0000
3         NaN

mask 与where 对比
需要注意的是，传入的条件只需是与被调用的Series索引一致的布尔序列即可：
s = pd.Series([-1, 1.2345, 100, -50])
s_condition= pd.Series([True,False,False,True],index=s.index)
print(s.mask(s_condition, -100))
0   -100.0000
1      1.2345
2    100.0000
3   -100.0000
代表着传入的布尔序列的为TRue的会被替换成对应的数据

数值替换包含了round, abs, clip方法，它们分别表示按照给定精度四舍五入、取绝对值和截断：
s = pd.Series([-1, 1.2345, 100, -50])
print(s.round(2))
0     -1.00
1      1.23
2    100.00
3    -50.00
全部数据保留两位小数 
s.abs() 绝对值
s.clip(0, 2) # 前两个数分别表示上下截断边界  代表低于第一个值的都设置为0 高于2 的都设置为2 在此范围内的不做任何处理

在 clip 中，超过边界的只能截断为边界值，如果要把超出边界的替换为自定义的值，应当如何做？
print(s.clip(0,4).replace(4,5))

-----------------------------------排序函数-----------------------------------

排序共有两种方式，其一为值排序，其二为索引排序，对应的函数是sort_values和sort_index。
为了演示排序函数，下面先利用set_index方法把年级和姓名两列作为索引，多级索引的内容和索引设置的方法将在后续文章中进行详细讲解。

对身高进行排序，默认参数ascending=True为升序：
print(df_demo.sort_values('Height').head())
参数ascending=False 为降序
print(df_demo.sort_values('Height', ascending=False).head())

在排序中，经常遇到多列排序的问题，比如在体重相同的情况下，对身高进行排序，并且保持身高降序排列，体重升序排列：
print(df_demo.sort_values(['Weight','Height'],ascending=[True,False]).head())

索引排序的用法和值排序完全一致，只不过元素的值在索引中，此时需要指定索引层的名字或者层号，用参数level表示。另外，需要注意的是字符串的排列顺序由字母顺序决定。

print(df_demo.sort_index(level=['Grade','Name'],ascending=[True,False]).head())

apply方法常用于DataFrame的行迭代或者列迭代，它的axis含义与第2小节中的统计聚合函数一致，apply的参数往往是一个以序列为输入的函数。例如对于.mean()，使用apply可以如下地写出：
df_demo = df[['Height', 'Weight']]
def my_mean(x):
    res = x.mean()
    return res
dfsa = df_demo.apply(my_mean)
print(dfsa)
类似于python 的map 函数 对可迭代 做方法上的处理
同样的，可以利用lambda表达式使得书写简洁，这里的x就指代被调用的df_demo表中逐个输入的序列：
df_demo.apply(lambda x:x.mean())
若指定axis=1，那么每次传入函数的就是行元素组成的Series，其结果与之前的逐行均值结果一致。
df_demo.apply(lambda x:x.mean(), axis=1).head()


得益于传入自定义函数的处理，apply的自由度很高，但这是以性能为代价的。一般而言，使用pandas的内置函数处理和apply来处理同一个任务，其速度会相差较多，因此只有在确实存在自定义需求的情境下才考虑使用apply。
```

#### 2  : 窗口对象      

```
pandas中有3类窗口，分别是滑动窗口rolling、扩张窗口expanding以及指数加权窗口ewm。需要说明的是，以日期偏置为窗口大小的滑动窗口将在后续文章中讨论，指数加权窗口见本章练习。
1. 滑窗对象
要使用滑窗函数，就必须先要对一个序列使用.rolling得到滑窗对象，其最重要的参数为窗口大小window。

s = pd.Series([1,2,3,4,5])
roller = s.rolling(window = 3)
print(roller)
Rolling [window=3,center=False,axis=0,method=single]
可以看到这是一个滑动窗口对象
在得到了滑窗对象后，能够使用相应的聚合函数进行计算，需要注意的是窗口包含当前行所在的元素，例如在第四个位置进行均值运算时，应当计算(2+3+4)/3，而不是(1+2+3)/3：
0    NaN    
1    NaN
2    2.0
3    3.0
4    4.0
对于滑动相关系数或滑动协方差的计算，可以如下写出：
s2 = pd.Series([1,2,6,16,30])
print(roller.cov(s2))

此外，还支持使用apply传入自定义函数，其传入值是对应窗口的Series，例如上述的均值函数可以等效表示：
print(roller.apply(lambda x:x.mean()))

shift, diff, pct_change是一组类滑窗函数，它们的公共参数为periods=n，默认为1，分别表示取向前第n个元素的值、与向前第n个元素做差（与Numpy中不同，后者表示n阶差分）、与向前第n个元素相比计算增长率。这里的n可以为负，表示反方向的类似操作。

将其视作类滑窗函数的原因是，它们的功能可以用窗口大小为n+1的rolling方法等价代替：
s.rolling(3).apply(lambda x:list(x)[0]) # s.shift(2)
s.rolling(4).apply(lambda x:list(x)[-1]-list(x)[0]) # s.diff(3)
def my_pct(x):
     L = list(x)
     return L[-1]/L[0]-1
s.rolling(2).apply(my_pct) # s.pct_change()

rolling对象的默认窗口方向都是向前的，某些情况下用户需要向后的窗口，例如对1,2,3设定向后窗口为2的`sum`操作，结果为3,5,NaN，此时应该如何实现向后的滑窗操作？
c= pd.Series([1,2,3])
c.rolling(window=2).sum().shift(-1)



 扩张窗口
扩张窗口又称累计窗口，可以理解为一个动态长度的窗口，其窗口的大小就是从序列开始处到具体操作的对应位置，其使用的聚合函数会作用于这些逐步扩张的窗口上。具体地说，设序列为a1, a2, a3, a4，则其每个位置对应的窗口即\[a1\]、\[a1, a2\]、\[a1, a2, a3\]、\[a1, a2, a3, a4\]。
s = pd.Series([1, 3, 6, 10])
print(s.expanding().mean())
0    1.000000
1    2.000000
2    3.333333
3    5.000000





cummax, cumsum, cumprod函数是典型的类扩张窗口函数，请使用expanding对象依次实现它们
输出：
0	 1.000000 
1	 2.000000 
2	 3.333333
3    5.000000
dtype: float64

sa.cummax()
输出：
0     1
1     3
2     6
3    10
dtype: int64

sa.cumsum()  
输出：
0     1
1     4
2    10
3    20
dtype: int64

sa.cumprod()
输出：
0      1
1      3
2     18
3    180
dtype: int64

我们可以参考一个世纪的例子  我们有三个数据 对其做一些处理 首先是分组 然后利用扩张函数进行计算
当 inplace = False 时，返回为修改过的数据，原数据不变。
当 inplace = True 时，返回值为 None，直接在原数据上进行操作。

import pandas as pd


def get_values(x, num):
    '''_type: 类型 0.全部 剩余的都是近几日
    '''
    if num == 0:
        print(sum(x))
        return sum(x)
    else:
        print(sum(x[::-1][0:num]))
        return sum(x[::-1][0:num])


df = pd.DataFrame([
    {'stats': [74, 5], 'country_id':86,
                    'app_id':'yr8u9uv20qqd2bo', 'market_id':1, 'time':1597248000},
    {'stats': [70, 7], 'country_id':86,
     'app_id':'yr8u9uv20qqd2bo', 'market_id':1, 'time':1597246000},
    {'stats': [72, 9], 'country_id':86,
     'app_id':'yr8u9uv20qqd2bo', 'market_id':1, 'time':1597244000},
],

                  )

gps = df.groupby(['country_id', 'app_id', 'market_id'])
data = []
for gp in gps:
    gp[1].sort_values(['time'], ascending=[True], inplace=True) 
    gp[1]['download'] = gp[1]['stats'].apply(lambda x: x[0])
    gp[1]['income'] = gp[1]['stats'].apply(lambda x: x[1])
    gp[1]['download_total'] = gp[1]['download'].expanding().apply(
        lambda x: get_values(x, 0))
    gp[1]['income_total'] = gp[1]['income'].expanding().apply(
        lambda x: get_values(x, 0))
    gp[1]['rpd'] = round(gp[1]['income_total']/gp[1]['download_total'], 2)
    print(gp[1]['rpd'])
    for index, row in gp[1].iterrows():
        stats = [row['download'], row['income'], row['rpd']]
        data.append(
            {'country_id': row['country_id'], 'app_id': row['app_id'], 'market_id': row['market_id'], 'stats': stats})
    print(data)
```

