# Interview questions collection

1.*args和**kwargs是什么意思?
```
答：*args表示可变参数(variadic arguments)，它允许你传入0个或任意个无名参数，这些参数在函数调用时自动组装为一个tuple; **kwargs表示关键字参数(keyword arguments)，它允许你传入0个或任意个含参数名的参数，这些关键字参数在函数内部自动组装为一个dict。同时使用*args和**kwargs的时候，必须保证*args在**kwargs之前。
```
2.python里面如何拷贝一个对象?
```
答：

(1) 赋值(=)，就是创建了对象的一个新的引用，修改其中任意一个变量都会影响到另一个;

(2)浅拷贝(copy.copy())，创建一个新的对象，但它包含的是对原始对象中包含项的引用(如果用引用的方式修改其中一个对象，另一个也会被改变);

(3)深拷贝(copy.deepcopy())，创建一个新的对象，并且递归的复制它所包含的对象(修改其中一个，另一个不会改变)

注意：并不是所有的对象都可以拷贝
```

3.简要描述python的垃圾回收机制
```

答：python中的垃圾回收是以引用计数为主，标记-清除和分代收集为辅。

引用计数：python在内存中存储每个对象的引用计数，如果计数变成0，该对象就会消失，分配给该对象的内存就会释放出来。

标记-清除：一些容器对象，比如说list、dict、tuple、instance等可能会出现引用循环，对于这些循环，垃圾回收器会定时回收这些循环(对象之间通过引用(指针)连在一起，构成一个有向图，对象构成这个有向图的节点，而引用关系构成这个有向图的边)。

分代收集：python把内存根据对象存活时间划分为三代，对象创建之后，垃圾回收器会分配它们所属的代。每个对象都会被分配一个代，而被分配更年轻的代是被优先处理的，因此越晚创建的对象越容易被回收。
```

4.什么是lambda函数?它有什么好处?
```
答：lambda表达式，通常是在需要一个函数，但是又不想费神去命名一个函数的场合下使用，也就是指匿名函数。

Python允许你定义一种单行的小函数。定义lambda函数的形式如下(lambda参数：表达式)lambda函数默认返回表达式的值。你也可以将其赋值给一个变量。lambda函数可以接受任意个参数，包括可选参数，但是表达式只有一个。
```
5.python如何实现单例模式?
```
答：单例模式是一种常用的软件设计模式。在它的核心结构中只包含一个被称为单例类的特殊类。通过单例模式可以保证系统中一个类只有一个单例而且该单例易于外界访问，从而方便对实例个数的控制并节约系统资源。如果希望在系统中某个类的对象只能存在一个，单例模式是最好的解决方案。

__new__()在__init__()之前被调用，用于生成实例对象。利用这个方法和累的属性的特点可以实现设计模式的单例模式。单例模式是指创建唯一对象，单例模式设计的类只能实例。

(1)使用__new__方法

class Singleton(object):
def __new__(cls, *args, **kw):
if not hasattr(cls, '_instance'):
orig = super(Singleton, cls)
cls._instance = orig.__new__(cls, *args, **kw)
return cls._instance
class MyClass(Singleton):
a = 1
(2)共享属性

class Borg(object):
_state = {}
def __new__(cls, *args, **kw):
ob = super(Borg, cls).__new__(cls, *args, **kw)
ob.__dict__ = cls._state
return ob
class MyClass2(Borg):
a = 1
(3)装饰器版本

def singleton(cls, *args, **kw):
instances = {}
def getinstance():
if cls not in instances:
instances[cls] = cls(*args, **kw)
return instances[cls]
return getinstance
@singleton
class MyClass:
...
(4)import方法

class My_Singleton(object):
def foo(self):
pass
my_singleton = My_Singleton()
# to use
from mysingleton import my_singleton
my_singleton.foo()
```

6.python自省
```
答：自省就是面向对象的语言所写的程序在运行时，所能知道对象的类型，简单一句就是运行时能够获得对象的类型，比如type(),dir(),getattr(),hasattr(),isinstance().

a = [1,2,3]
b = {'a':1,'b':2,'c':3}
c = True
print type(a),type(b),type(c) # 
print isinstance(a,list) # True
```

7.谈一谈python的装饰器
```
答：装饰器本质上是一个python函数，它可以让其他函数在不作任何变动的情况下增加额外功能，装饰器的返回值也是一个函数对象。它经常用于有切面需求的场景。比如：插入日志、性能测试，事务处理、缓存、权限验证等。有了装饰器我们就可以抽离出大量的与函数功能无关的雷同代码进行重用。
```

8.什么是鸭子类型?
```
答：在鸭子类型中，关注的不是对象的类型本身，而是他如何使用的。例如，在不适用鸭子类型的语言中，我们可以编写一个函数，它接受一个类型为鸭的对象，并调用它的走和叫方法。在使用鸭子类型的语言中，这样的一个函数可以接受一个任意类型的对象，并调用它的走和叫方法。

class duck():
def walk(self):
print('I am duck,I can walk...')
def swim(self):
print('I am duck,I can swim...')
def call(self):
print('I am duck,I can call...')
duck1=duck()
duck1.walk()
# I am duck,I can walk...
duck1.call() # I am duck,I can call...
```

9.@classmethod和@staticmethod

答：@classmethod修饰符对应的函数不需要实例化，不需要self参数，第一个参数需要是表示自身类的cls参数，cls参数可以用来调用类的属性，类的方法，实例化对象等。@staticmethod返回函数的静态方法，该方法不强制要求传递参数，如下声明一个静态方法：

Class C(object):
@staticmethod
Def f(arg1, arg2,…):
…
以上实例声明了静态方法f，类可以不用实例化就可以调用该方法C.f()，也可以实例化后调用C().f()。
```

10.谈一谈python中的元类
```
答：一般来说，我们都是在代码里定义类，用定义的类来创建实例。而使用元类，步骤又是同，定义元类，用元类创建类，再使用创建出来的类来创建实例。元类的主要目的就是为了当创建类时能够自动地改变类。
```