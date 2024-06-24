---
title: "pandas  2 "
date: 2023-03-01
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


### Pandas2

之前关于pandas 只是简单介绍了一下在项目中关于分词聚合的一个实际使用场景还是相对简单一点的接下来会介绍padnas 的实际使用

官方文档 https://github.com/pandas-dev/pandas

#### 1 pandas 文件的读取与写入

```
总所周知 pandas 关于文件的读写是一个基础功能 它是如何做到的呢，我们来看一下

---------------------------------------文件读取----------------------------------

pandas可以读取的文件格式有很多，这里主要介绍读取  csv, excel, txt文件 

有一个需要注意的地方是 似乎是read_csv 不支持相对路径
df_csv = pd.read_csv('/Users/......./my_csv.csv')
print(df_csv)

df_txt = pd.read_table('/Users/......./my_table.txt')
print(df_txt）

df_excel = pd.read_excel('/Users/......./my_excel.xlsx')
print(df_excel)

还有一些公用的参数，header=None表示第一行不作为列名，index_col表示把某一列或几列作为索引，usecols表示读取列的集合，默认读取所有的列，parse_dates表示需要转化为时间的列，关于时间序列的有关内容将在第十章讲解，nrows表示读取的数据行数。上面这些参数在上述的三个函数里都可以使用。

我们读取同一份txt文件 然后进行header=None的 对比（上面的加了heade=None） 可以看出 第一行不再是列名 新增一个0代表第一列

                            0
0      col1\tcol2\tcol3\tcol4
1   2\ta\t1.4\tapple 2020/1/1
2  3\tb\t3.4\tbanana 2020/1/2
3  6\tc\t2.5\torange 2020/1/5
4   5\td\t3.2\tlemon 2020/1/7
--------------------------------------------------
       col1\tcol2\tcol3\tcol4
0   2\ta\t1.4\tapple 2020/1/1
1  3\tb\t3.4\tbanana 2020/1/2
2  6\tc\t2.5\torange 2020/1/5
3   5\td\t3.2\tlemon 2020/1/7

我们读取同一份txt文件 然后加入 usecols 可以看到筛选出了我们想要的数据
   col1 col2  col3             col4
0     2    a   1.4   apple 2020/1/1
1     3    b   3.4  banana 2020/1/2
2     6    c   2.5  orange 2020/1/5
3     5    d   3.2   lemon 2020/1/7
--------------------------------------------------
   col1 col2
0     2    a
1     3    b
2     6    c
3     5    d


我们读取同一份csv文件 然后进行index_col参数对比的 (上面的加了index_col)可以发现 col1 和col2作为索引放到了前面
   col1 col2  col3    col4      col5
0     2    a   1.4   apple  2020/1/1
1     3    b   3.4  banana  2020/1/2
2     6    c   2.5  orange  2020/1/5
3     5    d   3.2   lemon  2020/1/7
--------------------------------------------------
           col3    col4      col5
col1 col2                        
2    a      1.4   apple  2020/1/1
3    b      3.4  banana  2020/1/2
6    c      2.5  orange  2020/1/5
5    d      3.2   lemon  2020/1/7

我们读取同一份excel文件 然后进行parse_dates参数对比 第二个的是加了parse_dates 第三个是 加入 nrows ==2 指定获取的行数

   col1 col2  col3    col4      col5
0     2    a   1.4   apple  2020/1/1
1     3    b   3.4  banana  2020/1/2
2     6    c   2.5  orange  2020/1/5
3     5    d   3.2   lemon  2020/1/7
--------------------------------------------------
   col1 col2  col3    col4       col5
0     2    a   1.4   apple 2020-01-01
1     3    b   3.4  banana 2020-01-02
2     6    c   2.5  orange 2020-01-05
3     5    d   3.2   lemon 2020-01-07

--------------------------------------------------
   col1 col2  col3    col4      col5
0     2    a   1.4   apple  2020/1/1
1     3    b   3.4  banana  2020/1/2

在读取txt文件时，经常遇到分隔符非空格的情况，read_table有一个分割参数sep，它使得用户可以自定义分割符号，进行txt数据的读取。例如，下面的读取的表以||||为分割：
增加两个参数 一个分割函数  一个语言引擎即可  sep=' \|\|\|\| ', engine='python'
在使用read_table的时候需要注意，参数sep中使用的是正则表达式，因此需要对|进行转义变成\|，否则无法读取到正确的结果


数据写入：
一般在数据写入中，最常用的操作是把index设置为False，特别当索引没有特殊意义的时候，这样的行为能把索引在保存的时候去除。
df_csv.to_csv('/my_csv_saved.csv', index=False)
df_excel.to_excel('/my_excel_saved.xlsx', index=False)
pandas中没有定义to_table函数，但是to_csv可以保存为txt文件，并且允许自定义分隔符，常用制表符\t分割：
pandas 还可以指定 to_markdown print(df_csv.to_markdown())

```

### 2: 基本的数据结构

```
pandas中具有两种基本的数据存储结构，存储一维values的Series和存储二维values的DataFrame，在这两种结构上定义了很多的属性和方法。
当然也有其他类型的数据结构 但是一般来说就是这两种数据结构为主  

---------------------------------series---------------------------------
series 一般由四个部分组成 分别是 序列的值data 索引index 存储类型dtype 序列的名字name ,其中索引可以指定它的名字 默认为空
s = pd.Series(data = [100, 'a', {'dic1':5}],
              index = pd.Index(['id1', 20, 'third'], name='my_idx'),
              dtype = 'object',
              name = 'my_name')
print(s)
我们生成一个简单的series 数据是一个列表有int str 和dict index 分别设置为 id1 20 和third 指定索引名称为 my_idx
存储的类型定为 object series 的name 为 my_name

object代表了一种混合类型，正如上面的例子中存储了整数、字符串以及Python的字典数据结构。此外，目前pandas把纯字符串序列也默认认为是一种object类型的序列，但它也可以用string类型存储
对于这些属性，可以通过 . 的方式来获取：
print(s.values,s.name,s.dtype,s.index)
还可以使用 .shape 来获取长度 
print(s.shape)
打印结果是  (3,) 可以看出这个是一个元组 此属性可以用于dataframe 所以第一个是行 第二个是列

---------------------------------DataFrame---------------------------------
dataframe 在 series 的基础上增加了列索引 一个数据矿可以由二维的data与行列索引来构造：
我们创建一个简单的 datafram 
data = [[1, 'a', 1.2], [2, 'b', 2.2], [3, 'c', 3.2]]
df = pd.DataFrame(data = data,
                  index = ['row_%d'%i for i in range(3)],
                  columns=['col_0', 'col_1', 'col_2'])
print(df)  查看打印结果
       col_0 col_1  col_2
row_0      1     a    1.2
row_1      2     b    2.2
row_2      3     c    3.2
是一个三行三列的dataframe 数据
但一般而言，更多的时候会采用从列索引名到数据的映射来构造数据框，同时再加上行索引
df = pd.DataFrame(data = {'col_0': [1,2,3],
                          'col_1':list('abc'),
                          'col_2': [1.2, 2.2, 3.2]},
                  index = ['row_%d'%i for i in range(3)])
由于这种映射关系，在DataFrame中可以用[col_name]与[col_list]来取出相应的列与由多个列组成的表，结果分别为Series和DataFrame：                  
print(df['col_0'])
print(df[['col_0', 'col_1']])
与Series类似，在数据框中同样可以取出相应的属性：
print(df.dtypes,df.values,df.columns,df.index)
通过.T 可以将 dataframe 进行转置

```



