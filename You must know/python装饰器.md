# python基础之装饰器

## 定义
本质就是函数，功能是装饰其他函数，其实就是给其他函数添加附加功能，


## 原则
1、不能修改被装饰的函数的源代码

2、不能修改被装饰的函数的调用方式

其实就是当你用装饰器函数装饰其他函数的时候，被装饰的函数被装饰之前的调用方式不会有任何区别


## 实现装饰器的知识储备
 
1、函数即“变量”

2、高阶函数

3、嵌套函数

**高阶函数+嵌套函数===装饰器**


## 关于函数即变量的理解
![关于函数即变量的理解](http://www.pythonsite.com/ueditor/php/upload/image/20170208/1486535209561090.png)

上图中描述了函数在内存中存放的原理图，其实test函数名就相当于函数在内存中存放的一个地址，也就是图中所说的房间门牌号

高阶函数
什么是高阶函数？

1、把一个函数名当做实参传给另外一个函数

2、返回值中包含函数名

 

写一个高阶函数的例子：
```
import time
  
def bar():
    time.sleep(3)
    print("in the bar")
def test1(func):
    start_time = time.time()
    func()    #这里的func()就相当于bar()
    stop_time = time.time()
    # 这个打印的时间就是func函数运行的时间，也就是bar函数的运行时间
    print("the fun runn time is %s" %(stop_time-start_time)) 
test1(bar)
```

上面的代码的例子就是一个高阶函数，满足了把函数名当做实参传给另外一个函数，上述代码中是把bar函数名传递给了test1作为实参，这里就起到了一个装饰的作用，相当于给bar函数增加了一个计算运行时间的功能，但是不能称为装饰器。因为上述我们的原则中说了，装饰器不能修改被装饰器函数的调用方式，我们这里的bar函数调用方式变了，变成了test1(bar),而不是bar(),并且我们的高阶函数还有一个条件没有满足，就是“返回值中包含函数名”

我们接着再写一个高阶函数的例子：

```
import time
def bar():
    time.sleep(3)
    print("in the bar")
def test2(func):
    print(func)
    return func
  
bar = test2(bar)
bar()
```

关于上述代码的一个分析：

当执行test2(bar)的时候，最后返回的其实就是bar函数内存地址，也就是我们所说的门牌号，所以当我们再次给他赋值给bar的时候，执行bar()就会执行bar函数

嵌套函数
 

嵌套函数就是在一个函数体内容再用def 定义一个函数，例子如下：
```
def foo():
    print("in the foo")
    def bar():
        print("int the bar")
    bar()
  
foo()
```

上述讲完了关于装饰器的知识储备：

函数即变量

高阶函数

嵌套函数

正式开始理解装饰器
 

高阶函数+嵌套函数===装饰器

 

我们写一个例子：
```
import time
def timer(func):
    def deco():
        start_time = time.time()
        func()  #这个时候的func()就是test1()
        stop_time = time.time()
        print("the func run time is %s" %(stop_time-start_time))
    return deco
  
def test1():
    time.sleep(3)
    print("in the test1")
  
test1 = timer(test1)
test1()
```

上述代码就是一个简单的装饰器，

通过下面图解对装饰器工作的原理的一个理解：
![图解装饰器工作的原理](http://www.pythonsite.com/ueditor/php/upload/image/20170208/1486535659136609.png)

但是上面代码中的使用装饰器的方法并不是最好的，因为我们是执行了test1=timer(test1)然后执行test1()，其实python中给我们提供了一种方法，即要装饰的函数上面添加@timer,执行@timer其实就是还行test1 = timer(test1)

,并且我们的这种方式，还存在一个问题，就是当被装饰的函数有参数需要传递的时候，就不能用了，所以我们修改后的代码为：
```
import time
def timer(func):
    def deco(*args,**kwargs):
        start_time = time.time()
        func(*args,**kwargs)  #这个时候的func()就是test1()
        stop_time = time.time()
        print("the func run time is %s" %(stop_time-start_time))
    return deco
  
@timer  #就相当于test1 = timer(teest1)
def test1():
    time.sleep(3)
    print("in the test1")
@timer
def test2(name):
    time.sleep(2)
    print("in the test2 %s" %name)
  
test1()
test2("zhaofan")
```

但是上面这种代码还不是最完整的，有一种情况上述的代码无法使用，就是在做用户认证的时候，如果有多个系统，需要不同的认证方式，注意：上述代码还有一个小问题就是如果被装饰的函数存在return返回值得时候，也是无法显示返回的数据，即代码应该为：

```
import time
user,pwd = "zhaofan","123"
def auth(auth_type):
    def outer_wapper(func):
        def wrapper(*args,**kwargs):
            print("认证方式为：\033[31;1m%s\033[0m" %auth_type)
            username = input("输入用户名：").strip()
            password = input("输入密码：").strip()
            if user == username and pwd == password:
                print("\033[32;1m 认证成功\033[0m")
                res=func(*args,**kwargs)
                return res
            else:
                exit("\033[31;1m 认证失败\033[0m")
        return wrapper
    return outer_wapper
@auth(auth_type="local")
def bbs():
    print("welcome to bbs page")
@auth(auth_type="ldap")
def blog(name):
    print("welcome to blog page")
    print(name)
    return name
bbs()
blog("zhaofan")
```

通过图解理解上面的代码：
![http://www.pythonsite.com](http://www.pythonsite.com/ueditor/php/upload/image/20170208/1486535912346342.png)

所以整个程序执行的过程如下：

@auth(auth_type=”local”)----->auth(auth_type=”local”) ----->因为auth(auth_type=”local”)就相当执行auth函数，所以执行结果为outer_wapper,即转换为

@outer_wapper----->@outer_wapper相当于bbs=outer_wapper(bbs)----->执行outer_wapper(bbs)的结果为wapper，所以bbs=wapper,也就是说bbs这个时候相当于wapper函数名的内存地址----->当执行bbs() 就相当于执行了wrapper()