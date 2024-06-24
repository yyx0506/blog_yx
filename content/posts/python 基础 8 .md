---
title: "python 继承多态"
date: 2022-10-08
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - collections
# - python
# - calendar
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->
 

Python 类继承 的相关问题



### 1 python 类方法 和实例方法 静态方法

```
----------------------类的方法类别----------------------
1: 类方法，通过装饰器@calssmethod进行修饰。
2: 静态方法，通过装饰器@staticmethod进行修饰。
3: 实例方法，属于方法类型的函数。
----------------------方法之间的功能差异------------------
1: 类方法：不能获取构造函数定义的变量，可以获取类的属性。
2: 静态方法：不能获取构造函数定义的变量，也不可以获取类的属性。
3: 实例方法：既可以获取构造函数定义的变量，也可以获取类的属性值。
----------------------方法的调用格式----------------------
1: 类方法：有两种调用方式，a  类名.类方法名     b  实例化调用
2: 静态方法：有两种调用方式，a  类名.静态方法名  b 实例化调用
3: 实例方法：见名知意，也许命名就是告诉大家，它只能通过实例化进行调用，事实也是。

一般来说 用的最多的就是 实例方法 和 类方法 一般来说 我们 某个类的方法 会经常被其他 外部类调用的话 建议使用类方法 只是内部使用的建议是 实例方法

下面我们聚一个例子
class FunctionTest:
    fun = "test"

    @classmethod
    def execute_class(cls):
        print("this is class method!")

    @staticmethod
    def execute_static(x):
        print("this is static method!")
        print(f"{x} is a num.")

    def execute_normal(self):
        print("This is normal method!")


if __name__ == '__main__':
    # 实例化调用
    FT = FunctionTest()
    # 只能通过实例化调用
    FT.execute_normal()
    # 实例化调用
    FT.execute_static(111)
    FT.execute_class()
    # 类.方法名 调用
    FunctionTest.execute_static(2222)
    FunctionTest.execute_class()
    
    FunctionTest.execute_normal()  # 会失败 
    FunctionTest().execute_normal()  # 加了括号代表声明了一个变量 
```

### 2 继承

```
python 作为一门面向对象的语言 继承 和多态是一定有的  关于继承：

1: 子类可以继承父类（基类）的属性和方法。
2: 子类不能继承和调用父类的私有（private）方法。
3: 子类中定义和父类中同名的方法或属性，父类的方法或属性将被覆盖。（多态）
4: 多继承：所以的类都可以追溯到根类（object）.

例如 
class Animals(object):
    def eat(self):
        print('i can eat')

    def call(self):
        print('i can call phone ')

class Dog(Animals):
    
    def eat(self):
        print(' i like eat gutouts')
当子类去继承了父类时就拥有了父类的方法。如果需要修改父类的方法 也可以直接重写 ，不重写的话就是直接继承

如果 你 已经重写了 父类的方法 但是还是想要去调用父类的 方法怎们办 那就需要 super()

class CarInfors(object):
    def __init__(self, brand, model, color):
        self.brand = brand
        self.model = model
        self.color = color

    def run(self):
        print('每秒 我能跑 20公里')


class GasolineCar(Car):
    def __init__(self, brand, model, color):
        super().__init__(brand, model, color)

    def run(self):
        print('每秒 我能跑 80公里')


class ElectricCar(Car):
    def __init__(self, brand, model, color):
        super().__init__(brand, model, color)
        # 代表电池电量。
        self.battery = 70

    def run(self):
        print(f'每秒 我能跑 60公里}')


bmw = GasolineCar('保时捷', '911', '白色')
bmw.run()

tesla = ElectricCar('比亚迪', '唐', '红色')
tesla.run()


继承时 可以重写 父类的方法 但是 要记住的是 继承时 方法 和参数都要是一样。
python中出现函数名相同但参数不同的函数时，后面的同名不同参的函数会覆盖掉前面的同名不同参函数，这与java中是完全不一样的 python 并不能实现 重载的功能 


---------------各种方法继承问题---------------
1  构造 方法的继承。
都没有定义'构造方法',默认'使用子类'的 
父类'自定义'构造方法,子类'没有自定义'构造方法,子类会'继承父类'的构造方法,使用'父类'的构造方法
父类和子类同时'自定义'构造方法,默认会'覆盖'父类的构造方法
简单来说。子类优先 

__ 开头的方法无法被继承 静态方法也无法被继承
```

### 3: 多态

```
class Fruit(object):
    def makejuice(self):
        print('i can make juice ')


class Apple(Fruit):
    def makejuice(self):
        print('i can make  apple juice ')


class Banana(Fruit):
    def makejuice(self):
        print('i can make Banana juice ')


class Orange(Fruit):
    def makejuice(self):
        print('i can make Orange juice ')


# 利用多态。  定义一个service  公共方法接口
def service(obj):
    obj.makejuice()


# apple = Apple()
# apple.makejuice()  # 以往的方式，需要用一个调用一个
# banana = Banana()
# banana.makejuice()  # 橘子也是 一样
# orange = Orange()
# orange.makejuice()

# 利用接口
apple = Apple()
banana = Banana()
orange = Orange()
for i in (apple,banana,orange):
    service(i)

```

