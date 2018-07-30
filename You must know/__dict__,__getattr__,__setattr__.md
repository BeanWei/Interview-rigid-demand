# python中的__dict__,__getattr__,__setattr__

_python class 通过内置成员__dict__存储成员信息(字典)

首先用一个简单的例子看一下__dict__ 的用法
```
class A():
    def __init__(self,ax,bx):
        self.a = ax
        self.b = bx
    def f(self):
        print (self.__dict__)
a = A(1,2)
a.f()
``` 
> 输出结果：{‘b’: 2, ‘a’: 1}

利用__dict__ 可以达到一些简化代码的目的，参考下面的例子

```
class Person:
    def __init__(self,_obj):
        self.name = _obj['name']
        self.age = _obj['age']
        self.energy = _obj['energy']
        self.gender = _obj['gender']
        self.email = _obj['email']
        self.phone = _obj['phone']
        self.country = _obj['country']
```
简化成下面的代码

```
class Person:
    def __init__(self,_obj):
        self.__dict__.update(_obj)
```

我们可以通过重载__getattr__和__setattr__来拦截对成员的访问或者作出一些自己希望的行为 
__getattr__ 在访问对象访问类中不存在的成员时会自动调用
```
class A():
    def __init__(self,ax,bx):
        self.a = ax
        self.b = bx
    def f(self):
        print (self.__dict__)
    def __getattr__(self,name):
        print ("__getattr__")
a = A(1,2)
a.f()
a.x
```
> 输出结果
> {'b': 2, 'a': 1}
> __getattr__
 
__setattr__ 方法用于在初始化对象成员的时候调用，即在设置__dict__的item时就会调用__setattr__方法。 具体看下面的例子
```
class A():
    def __init__(self,ax,bx):
        self.a = ax
        self.b = bx
    def f(self):
        print (self.__dict__)
    def __getattr__(self,name):
        print ("__getattr__")
    def __setattr__(self,name,value):
        print ("__setattr__")
        self.__dict__[name] = value

a = A(1,2)
a.f()
a.x
a.x = 3
a.f()
```
> 输出结果
> __setattr__
> __setattr__
> {'a': 1, 'b': 2}
> __getattr__
> __setattr__
> {'a': 1, 'x': 3, 'b': 2}
