# Python设计模式

设计模式的定义:为了解决面向对象系统中重要和重复的设计封装在一起的一种代码实现框架,可以使得代码更加易于扩展和调用

四个基本要素:模式名称,问题,解决方案,效果

六大原则:

　　1.开闭原则:一个软件实体,如类,模块和函数应该对扩展开发,对修改关闭.既软件实体应尽量在不修改原有代码的情况下进行扩展.

       2.里氏替换原则:所有引用父类的方法必须能透明的使用其子类的对象

       3.依赖倒置原则:高层模块不应该依赖底层模块,二者都应该依赖其抽象,抽象不应该依赖于细节,细节应该依赖抽象,换而言之,要针对接口编程而不是针对实现编程

       4.接口隔离原则:使用多个专门的接口,而不是使用单一的总接口,即客户端不应该依赖那些并不需要的接口

       5.迪米特法则:一个软件实体应该尽可能的少与其他实体相互作用

       6.单一直责原则:不要存在多个导致类变更的原因.即一个类只负责一项职责

零:接口

　 定义:一种特殊的类,声明了若干方法,要求继承该接口的类必须实现这种方法

    作用:限制继承接口的类的方法的名称及调用方式,隐藏了类的内部实现

```
 from abc import ABCMeta,abstractmethod
 
 class Payment(metaclass=ABCMeta):
     @abstractmethod#定义抽象方法的关键字
     def pay(self,money):
         pass
 
     # @abstractmethod
     # def pay(self,money):
     #     raise NotImplementedError
 
 class AiliPay(Payment):
     #子类继承接口,必须实现接口中定义的抽象方法,否则不能实例化对象
     def pay(self,money):
         print('使用支付宝支付%s元'%money)
 
 class ApplePay(Payment):
     def pay(self,money):
         print('使用苹果支付支付%s元'%money)
```

一:单例模式

     定义:保证一个类只有一个实例,并提供一个访问它的全局访问点

     适用场景:当一个类只能有一个实例而客户可以从一个众所周知的访问点访问它时

     优点:对唯一实例的受控访问,相当于全局变量,但是又可以防止此变量被篡改


```
 class Singleton(object):
     #如果该类已经有了一个实例则直接返回,否则创建一个全局唯一的实例
     def __new__(cls, *args, **kwargs):
         if not hasattr(cls,'_instance'):
             cls._instance = super(Singleton,cls).__new__(cls)
         return cls._instance
 
 class MyClass(Singleton):
     def __init__(self,name):
         if name:
             self.name = name
 
 a = MyClass('a')
 print(a)
 print(a.name)
 
 b = MyClass('b')
 print(b)
 print(b.name)
 
 print(a)
 print(a.name)
```

二:简单工厂模式

     定义:不直接向客户暴露对象创建的实现细节,而是通过一个工厂类来负责创建产品类的实例

     角色:工厂角色,抽象产品角色,具体产品角色

     优点:隐藏了对象创建代码的细节,客户端不需要修改代码

     缺点:违反了单一职责原则,将创建逻辑集中到一个工厂里面,当要添加新产品时,违背了开闭原则


```
 from abc import ABCMeta,abstractmethod
 
 class Payment(metaclass=ABCMeta):
     #抽象产品角色
     @abstractmethod
     def pay(self,money):
         pass
 
 class AiliPay(Payment):
     #具体产品角色
     def __init__(self,enable_yuebao=False):
         self.enable_yuebao = enable_yuebao
 
     def pay(self,money):
         if self.enable_yuebao:
             print('使用余额宝支付%s元'%money)
         else:
             print('使用支付宝支付%s元'%money)
 
 class ApplePay(Payment):
     # 具体产品角色
     def pay(self,money):
         print('使用苹果支付支付%s元'%money)
 
 class PaymentFactory:
     #工厂角色
     def create_payment(self,method):
         if method == 'alipay':
             return AiliPay()
         elif method == 'yuebao':
             return AiliPay(True)
         elif method == 'applepay':
             return ApplePay()
         else:
             return NameError
 
 p = PaymentFactory()
 f = p.create_payment('yuebao')
 f.pay(100)
```

三:工厂方法模式

     定义:定义一个创建对象的接口(工厂接口),让子类决定实例化哪个接口

     角色:抽象工厂角色,具体工厂角色,抽象产品角色,具体产品角色

     适用场景:需要生产多种,大量复杂对象的时候,需要降低代码耦合度的时候,当系统中的产品类经常需要扩展的时候

     优点:每个具体的产品都对应一个具体工厂,不需要修改工厂类的代码,工厂类可以不知道它所创建的具体的类,隐藏了对象创建的实现细节

     缺点:每增加一个具体的产品类,就必须增加一个相应的工厂类

```
 #!/usr/bin/env python
 # -*- coding: utf-8 -*-
 # __author__= 'luhj'
 
 from abc import ABCMeta,abstractmethod
 
 class Payment(metaclass=ABCMeta):
     #抽象产品
     @abstractmethod
     def pay(self,money):
         pass
 
 
 class AliPay(Payment):
     #具体产品
     def pay(self,money):
         print('使用支付宝支付%s元'%money)
 
 class ApplePay(Payment):
     def pay(self,money):
         print('使用苹果支付支付%s元'%money)
 
 class PaymentFactory(metaclass=ABCMeta):
     #抽象工厂
     @abstractmethod
     def create_payment(self):
         pass
 
 class AliPayFactory(PaymentFactory):
     #具体工厂
     def create_payment(self):
         return AliPay()
 
 class ApplePayFactory(PaymentFactory):
     def create_payment(self):
         return ApplePay()
 
 af = AliPayFactory()
 ali = af.create_payment()
 ali.pay(100)
 
 #如果要新增支付方式
 class WechatPay(Payment):
     def pay(self,money):
         print('使用微信支付%s元'%money)
         
 class WechatPayFactory(PaymentFactory):
     def create_payment(self):
         return WechatPay()
 
 w = WechatPayFactory()
 wc = w.create_payment()
 wc.pay(200)
```

四:抽象工厂模式

　  定义:定义一个工厂类接口,让工厂子类来创建一系列相关或相互依赖的对象

     角色:抽象工厂角色,具体工厂角色,抽象产品角色,具体产品角色,客户端

     适用场景:系统要独立于产品的创建和组合时,强调一系列相关产品的对象设计以便进行联合调试时,提供一个产品类库,想隐藏产品的具体实现时

     优点:将客户端与类的具体实现相分离,每个工厂创建了一个完整的产品系列,易于交换产品.有利于产品的一致性

     缺点:难以支持新种类的产品

```
from abc import abstractmethod, ABCMeta  # ------抽象产品------ 
class PhoneShell(metaclass=ABCMeta):      
@abstractmethod     
def show_shell(self):         
    pass 
class CPU(metaclass=ABCMeta):
    @abstractmethod
    def show_cpu(self):
        pass

class OS(metaclass=ABCMeta):
    @abstractmethod
    def show_os(self):
        pass

# ------抽象工厂------
class PhoneFactory(metaclass=ABCMeta):

    @abstractmethod
    def make_shell(self):
        pass

    @abstractmethod
    def make_cpu(self):
        pass

    @abstractmethod
    def make_os(self):
        pass

# ------具体产品------
class SmallShell(PhoneShell):
    def show_shell(self):
        print('小手机壳')

class BigShell(PhoneShell):
    def show_shell(self):
        print('大手机壳')

class AppleShell(PhoneShell):
    def show_shell(self):
        print('苹果机壳')

class SnapDragonCPU(CPU):
    def show_cpu(self):
        print('骁龙CPU')

class MediaTekCPU(CPU):
    def show_cpu(self):
        print('联发科CPU')

class AppleCPU(CPU):
    def show_cpu(self):
        print('苹果CPU')

class Andriod(OS):
    def show_os(self):
        print('安卓系统')

class IOS(OS):
    def show_os(self):
        print('iOS系统')

# ------具体工厂------
class MiFactory(PhoneFactory):
    def make_shell(self):
        return BigShell()

    def make_os(self):
        return Andriod()

    def make_cpu(self):
        return SnapDragonCPU()

class HuaweiFactory(PhoneFactory):
    def make_shell(self):
        return SmallShell()

    def make_os(self):
        return Andriod()

    def make_cpu(self):
        return MediaTekCPU()

class AppleFactory(PhoneFactory):
    def make_shell(self):
        return AppleShell()

    def make_os(self):
        return IOS()

    def make_cpu(self):
        return AppleCPU()

# ------客户端------
class Phone:
    def __init__(self,shell,os,cpu):
        self.shell=shell
        self.os=os
        self.cpu=cpu

    def show_info(self):
        print('手机信息')
        self.cpu.show_cpu()
        self.shell.show_shell()
        self.os.show_os()

def make_phone(factory):
    cpu = factory.make_cpu()
    os = factory.make_os()
    shell = factory.make_shell()
    return Phone(shell,os,cpu)
 
 p1 = make_phone(AppleFactory())
 p1.show_info()
```

 五:建造者模式

     定义:将一个复杂对象的构建与它的表示分离,使得同样的构建过程可以创建不同的表示

     角色:抽象建造者,具体建造者,指挥者,产品

     适用场景:当创建复杂对象的算法应该独立于对象的组成部分以及它的装配方式,当构造过程允许被构造的对象有不同的表示

     优点:隐藏了一个产品的内部结构和装配过程,将构造代码与表示代码分开,可以对构造过程进行更精确的控制


```
 from abc import abstractmethod, ABCMeta
 
 #------产品------
 class Player:
     def __init__(self,face=None, body=None, arm=None, leg=None):
         self.face =face
         self.body=body
         self.arm=arm
         self.leg=leg
 
     def __str__(self):
         return '%s,%s,%s,%s'%(self.face,self.body,self.arm,self.leg)
 
 #------建造者------
 class PlayerBuilder(metaclass=ABCMeta):
     @abstractmethod
     def build_face(self):
         pass
 
     @abstractmethod
     def build_body(self):
         pass
 
     @abstractmethod
     def build_arm(self):
         pass
 
     @abstractmethod
     def build_leg(self):
         pass
 
     @abstractmethod
     def get_player(self):
         pass
 
 #------具体建造者------
 class BeautifulWoman(PlayerBuilder):
     def __init__(self):
         self.player=Player()
 
     def build_face(self):
         self.player.face = '白脸蛋'
 
     def build_body(self):
         self.player.body = '好身材'
 
     def build_arm(self):
         self.player.arm = '细胳膊'
 
     def build_leg(self):
         self.player.leg = '大长腿'
 
     def get_player(self):
         return self.player
 
 #------指挥者------
 class PlayerDirecter:
     def build_player(self,builder):
         builder.build_face()
         builder.build_body()
         builder.build_arm()
         builder.build_leg()
         return builder.get_player()
 
 director = PlayerDirecter()
 builder = BeautifulWoman()
 p = director.build_player(builder)
 print(p)
```

 六:适配器模式

     定义:将一个接口转换为客户希望的另一个接口,该模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作

     角色:目标接口,待适配的类,适配器

     适用场景:想使一个已经存在的类,但其接口不符合你的要求.想对一些已经存在的子类.不可能每一个都是用子类来进行适配,对象适配器可以适配其父类接口

```
 from abc import abstractmethod, ABCMeta
 
 
 class Payment(metaclass=ABCMeta):
     @abstractmethod
     def pay(self, money):
         raise NotImplementedError
 
 
 class Alipay(Payment):
     def pay(self, money):
         print("支付宝支付%s元"%money)
 
 
 class ApplePay(Payment):
     def pay(self, money):
         print("苹果支付%s元"%money)
 
 #------待适配的类-----
 class WeChatPay:
     def fuqian(self,money):
         print('微信支付%s元'%money)
 
 #------类适配器------
 class RealWeChatPay(Payment,WeChatPay):
     def pay(self, money):
         return self.fuqian(money)
 
 #-----对象适配器-----
 class PayAdapter(Payment):
     def __init__(self,payment):
         self.payment=payment
     def pay(self, money):
         return self.payment.fuqian(money)
 
 #RealWeChatPay().pay(100)
 
 p=PayAdapter(WeChatPay())
 p.pay(200)
```

 七:组合模式

     定义:将对象组合成树形结构以表示'部分-整体'的层次结构.组合模式使得用户对单个对象和组合对象的使用具有一致性

     角色:抽象组件,叶子组件,复合组件,客户端

     适用场景:表示对象的'部分-整体'层次结构,希望用户忽略组合对象与单个对象的不同,用户统一使用组合结构中的所有对象

     优点:定义了包含基本对象和组合对象的类层次结构,简化客户端代码,即客户端可以一致的使用组合对象和单个对象,更容易新增新类型的组件

     缺点:很难限制组合中的组件

```
 from abc import abstractmethod, ABCMeta
 
 #-------抽象组件--------
 class Graph(metaclass=ABCMeta):
 
     @abstractmethod
     def draw(self):
         pass
 
     @abstractmethod
     def add(self,graph):
         pass
 
     def get_children(self):
         pass
 
 #---------叶子组件--------
 class Point(Graph):
     def __init__(self,x,y):
         self.x = x
         self.y = y
 
     def draw(self):
         print(self)
 
     def add(self,graph):
         raise TypeError
 
     def get_children(self):
         raise TypeError
 
     def __str__(self):
         return '点(%s,%s)'%(self.x,self.y)
 
 
 class Line(Graph):
 
     def __init__(self,p1,p2):
         self.p1 = p1
         self.p2 = p2
 
     def draw(self):
         print(self)
 
     def add(self,graph):
         raise TypeError
 
     def get_children(self):
         raise TypeError
 
     def __str__(self):
         return '线段(%s,%s)'%(self.p1,self.p2)
 
 #--------复合组件---------
 class Picture(Graph):
     def __init__(self):
         self.children = []
 
     def add(self,graph):
         self.children.append(graph)
 
     def get_children(self):
         return self.children
 
     def draw(self):
         print('-----复合图形-----')
         for g in self.children:
             g.draw()
         print('结束')
 
 
 #---------客户端---------
 pic1 = Picture()
 point = Point(2,3)
 pic1.add(point)
 pic1.add(Line(Point(1,2),Point(4,5)))
 pic1.add(Line(Point(0,1),Point(2,1)))
 
 pic2 = Picture()
 pic2.add(Point(-2,-1))
 pic2.add(Line(Point(0,0),Point(1,1)))
 
 pic = Picture()
 pic.add(pic1)
 pic.add(pic2)
 
 pic.draw()
```

八:代理模式

　  定义:为其他对象提供一种代理以控制对特定对象的访问

     角色:抽象实体,实体,代理

     适用场景:远程代理(为远程的对象提供代理),虚代理(根据需要创建很大的对象,即懒加载),保护代理(控制对原始对象的访问,用于具有不同访问权限的对象)

     优点:远程代理(可以隐藏对象位于远程地址空间的事实),虚代理(可对大对象的加载进行优化),保护代理(允许在访问一个对象时有一些附加的处理逻辑,例如权限控制)


```
 from abc import ABCMeta, abstractmethod
 
 #抽象实体
 class Subject(metaclass=ABCMeta):
     @abstractmethod
     def get_content(self):
         pass
 
 #实体
 class RealSubject(Subject):
     def __init__(self,filename):
         print('读取文件%s内容'%filename)
         f = open(filename)
         self.content = f.read()
         f.close()
 
     def get_content(self):
         return self.content
 
 #远程代理
 class ProxyA(Subject):
     def __init__(self,filename):
         self.subj =RealSubject(filename)
     def get_content(self):
         return self.subj.get_content()
 
 #虚代理
 class ProxyB(Subject):
     def __init__(self,filename):
         self.filename = filename
         self.subj = None
     def get_content(self):
         if not self.subj:
             self.subj = RealSubject(self.filename)
         return self.subj.get_content()
 
 #保护代理
 class ProxyC(Subject):
     def __init__(self,filename):
         self.subj = RealSubject(filename)
     def get_content(self):
         return '???'
 
 
 #客户端
 filename = 'abc.txt'
 username = input('>>')
 if username!='alex':
     p=ProxyC(filename)
 else:
     p=ProxyB(filename)
 
 print(p.get_content())
```

 九:观察者模式

     定义:定义对象间的一种一对多的依赖关系,当一个对象的状态发生改变时,所有依赖它的对象都会得到通知并被自动更新.观察者模式又称为'发布订阅'模式

     角色:抽象主题,具体主题(发布者),抽象观察者,具体观察者(订阅者)

     适用场景:当一个抽象模型有两个方面,其中一个方面依赖于另一个方面.将两者封装在独立的对象中以使它们各自独立的改变和复用

                    当一个对象的改变需要同时改变其他对象,而且不知道具体有多少对象以待改变

                    当一个对象必须通知其他对象,而又不知道其他对象是谁,即这些对象之间是解耦的

     优点:目标和观察者之间的耦合最小,支持广播通信

     缺点:多个观察者之间互不知道对方的存在,因此一个观察者对主题的修改可能造成错误的更新


```
 from abc import ABCMeta, abstractmethod
 
 #抽象主题
 class Oberserver(metaclass=ABCMeta):
     @abstractmethod
     def update(self):
         pass
 
 #具体主题
 class Notice:
     def __init__(self):
         self.observers = []
 
     def attach(self,obs):
         self.observers.append(obs)
 
     def detach(self,obs):
         self.observers.remove(obs)
 
     def notify(self):
         for obj in self.observers:
             obj.update(self)
 
 #抽象观察者
 class ManagerNotice(Notice):
     def __init__(self,company_info=None):
         super().__init__()
         self.__company_info = company_info
 
     @property
     def company_info(self):
         return self.__company_info
 
     @company_info.setter
     def company_info(self,info):
         self.__company_info = info
         self.notify()
 
 #具体观察者
 class Manager(Oberserver):
     def __init__(self):
         self.company_info = None
     def update(self,noti):
         self.company_info = noti.company_info
 
 #消息订阅-发送
 notice = ManagerNotice()
 
 alex=Manager()
 tony=Manager()
 
 notice.attach(alex)
 notice.attach(tony)
 notice.company_info="公司运行良好"
 print(alex.company_info)
 print(tony.company_info)
 
 notice.company_info="公司将要上市"
 print(alex.company_info)
 print(tony.company_info)
 
 notice.detach(tony)
 notice.company_info="公司要破产了，赶快跑路"
 print(alex.company_info)
 print(tony.company_info)
```

十:策略模式

     定义:定义一系列的算法把它们一个个封装起来,并且使它们可相互替换.该模式使得算法可独立于使用它的客户而变化

     角色:抽象策略,具体策略,上下文

     适用场景:许多相关的类仅仅是行为有异,需使用一个算法的不同变体,算法使用了客户端无需知道的数据,一个类中的多个行为以多个条件语句存在可以将其封装在不同的策略类中

     优点:定义了一系列可重用的算法和行为,消除了一些条件语句,可提供相同行为的不同实现

     缺点:客户必须了解不同的策略,策略与上下文之间的通信开销,增加了对象的数目

```
 from abc import ABCMeta, abstractmethod
 import random
 
 #抽象策略
 class Sort(metaclass=ABCMeta):
     @abstractmethod
     def sort(self,data):
         pass
 
 #具体策略
 class QuickSort(Sort):
 
     def quick_sort(self,data,left,right):
         if left<right:
             mid = self.partation(data,left,right)
             self.quick_sort(data,left,mid-1)
             self.quick_sort(data,mid+1,right)
 
     def partation(self,data,left,right):
         tmp = data[left]
         while left < right:
             while left<right and data[right]>=tmp:
                 right -= 1
             data[left] = data[right]
 
             while left<right and data[left]<=tmp:
                 left += 1
             data[right] = data[left]
         data[left] = tmp
         return left
 
     def sort(self,data):
         print("快速排序")
         return self.quick_sort(data,0,len(data)-1)
 
 class MergeSort(Sort):
     def merge(self,data,low,mid,high):
         i = low
         j = mid+1
         ltmp = []
         while i <= mid and j <= high:
             if data[i] <= data[j]:
                 ltmp.append(data[i])
                 i+=1
             else:
                 ltmp.append(data[j])
                 j+=1
         while i <= mid:
             ltmp.append(data[i])
             i+=1
         while j <= high:
             ltmp.append(data[j])
             j+=1
         data[low:high+1]=ltmp
 
     def merge_sort(self,data,low,high):
         if low<high:
             mid = (low+high)//2
             self.merge_sort(data,low,mid)
             self.merge_sort(data,mid+1,high)
             self.merge(data,low,mid,high)
     def sort(self,data):
         print("归并排序")
         return self.merge_sort(data,0,len(data)-1)
 
 #上下文
 class Context:
     def __init__(self,data,strategy=None):
         self.data=data
         self.strategy=strategy
 
     def set_strategy(self,strategy):
         self.strategy=strategy
 
     def do_strategy(self):
         if self.strategy:
             self.strategy.sort(self.data)
         else:
             raise TypeError
 
 
 li = list(range(100000))
 random.shuffle(li)
 context = Context(li,MergeSort())
 context.do_strategy()
 
 random.shuffle(context.data)
 context.set_strategy(QuickSort())
 context.do_strategy()
```

 十一:责任链模式

     定义:使多个对象有机会处理请求,从而避免请求的发布者和接收者之间的耦合关系,将这些对象连成一条链,并沿着这条链传递该请求,直到有一个对象能处理它为止

     角色:抽象处理者,具体处理者,客户端

     适用场景:有多个对象可以处理一个请求,哪个对象处理由运行时决定

     优点:降低耦合度,一个对象无需知道是其他哪一个对象处理其请求

     缺点:请求不保证被接收,链的末端没有处理或链配置错误

```
 from abc import ABCMeta, abstractmethod
 
 class Handler(metaclass=ABCMeta):
     @abstractmethod
     def handel_leave(self,day):
         pass
 
 class GeneralManagerHandler(Handler):
     def handel_leave(self,day):
         if day < 10:
             print('总经理批准请假%s天'%day)
 
         else:
             print('不能请假')
 
 class DepartmentManagerHandler(Handler):
     def __init__(self):
         self.successor = GeneralManagerHandler()
     def handel_leave(self,day):
         if day < 7:
             print('部门经理批准请假%s天' % day)
         else:
             print('部门经理无权批假')
             self.successor.handel_leave(day)
 
 class ProjectDirectorHandler(Handler):
     def __init__(self):
         self.successor = DepartmentManagerHandler()
     def handel_leave(self,day):
         if day < 3:
             print('项目经理批准请假%s天' % day)
         else:
             print('项目经理无权批假')
             self.successor.handel_leave(day)
 
 day = 6
 h = ProjectDirectorHandler()
 h.handel_leave(day)
```

```
 from abc import ABCMeta, abstractmethod
 #--模仿js事件处理
 class Handler(metaclass=ABCMeta):
     @abstractmethod
     def add_event(self,func):
         pass
 
     @abstractmethod
     def handler(self):
         pass
 
 class BodyHandler(Handler):
     def __init__(self):
         self.func = None
 
     def add_event(self,func):
         self.func = func
 
     def handler(self):
         if self.func:
             return self.func()
         else:
             print('已经是最后一级,无法处理')
 
 
 class ElementHandler(Handler):
 
     def __init__(self,successor):
         self.func = None
         self.successor = successor
 
     def add_event(self,func):
         self.func = func
 
     def handler(self):
         if self.func:
             return self.func()
         else:
             return self.successor.handler()
 
 
 #客户端
 body = {'type': 'body', 'name': 'body', 'children': [], 'father': None}
 
 div = {'type': 'div', 'name': 'div', 'children': [], 'father': body}
 
 a = {'type': 'a', 'name': 'a', 'children': [], 'father': div}
 
 body['children'] = div
 div['children'] = a
 
 body['event_handler'] = BodyHandler()
 div['event_handler'] = ElementHandler(div['father']['event_handler'])
 a['event_handler'] = ElementHandler(a['father']['event_handler'])
 
 def attach_event(element,func):
     element['event_handler'].add_event(func)
 
 #测试
 def func_div():
     print("这是给div的函数")
 
 def func_a():
     print("这是给a的函数")
 
 def func_body():
     print("这是给body的函数")
 
 attach_event(div,func_div)
 #attach_event(a,func_a)
 attach_event(body,func_body)
 
 a['event_handler'].handler()
```

 十二:迭代器模式

     定义:提供一种方法可顺序访问一个聚合对象中的各个元素,而又不需要暴露该对象的内部指示

     适用场景:实现方法__iter__,__next__


```
 class LinkedList:
     class Node:
         def __init__(self,item=None):
             self.item=item
             self.next=None
     class LinkedListIterator:
         def __init__(self,node):
             self.node = node
         #实现next方法,返回下一个元素
         def __next__(self):
             if self.node:
                 cur_node = self.node
                 self.node = cur_node.next
                 return cur_node.item
 
         def __iter__(self):
             return self
     def __init__(self,iterable=None):
         self.head = LinkedList.Node(0)
         self.tail = self.head
         self.extend(iterable)
 
     #链表尾部追加元素
     def append(self,obj):
         s = LinkedList.Node(obj)
         self.tail.next = s
         self.tail = s
     #链表自动增加长度
     def extend(self,iterable):
         for obj in iterable:
             self.append(obj)
         self.head.item += len(iterable)
 
     def __iter__(self):
         return self.LinkedListIterator(self.head.next)
 
     def __len__(self):
         return self.head.item
 
     def __str__(self):
         return '<<'+', '.join(map(str,self)) + '>>'
 
 li = [i for i in range(100)]
 lk = LinkedList(li)
 print(lk)
```

十三:模板方法模式

     定义:定义一个操作中算法的骨架,将一些步骤延迟到子类中,模板方法使得子类可以不改变一个算法的结构即可重定义该算法某些特定的步骤

     角色:抽象类(定义抽象的原子操作,实现一个模板方法作为算法的骨架),具体类(实现原子操作)

     适用场景:一次性实现一个算法不变的部分,各个子类的公共行为,应该被提取出来集中到公共的父类中以避免代码重复,控制子类扩展


```
 from abc import ABCMeta, abstractmethod
 
 #----抽象类-----
 class IOHandler(metaclass=ABCMeta):
     @abstractmethod
     def open(self,name):
         pass
 
     @abstractmethod
     def deal(self,change):
         pass
 
     @abstractmethod
     def close(self):
         pass
     #在父类中定义了子类的行为
     def process(self,name,change):
         self.open(name)
         self.deal(change)
         self.close()
 
 #子类中只需要实现部分算法,而不需要实现所有的逻辑
 #-----具体类--------
 class FileHandler(IOHandler):
     def open(self,name):
         self.file = open(name,'w')
 
     def deal(self,change):
         self.file.write(change)
 
     def close(self):
         self.file.close()
 
 f = FileHandler()
 f.process('abc.txt','hello')
```