# 类与继承

## 22. 尽量用辅助类来维护程序的状态，而不要用字典和元组

- 不要使用包含其他字典的字典，也不要使用过长的元组
- 如果容器中包含简单而又不可改变的数据，那么可以先使用`namedtuple`来表示，待稍后有需要时，再修改为完整的类（可以参考[namedtuple](http://www.zhangdongshengtech.com/article-detials/214)）
- 保存内部状态的字典如果变得非常复杂， 那就应该把这些代码拆解为多个辅助类

```python
# 下述一个例子---比较好的写法：各个对象之间剥离开来
class Grade(object):
    def __init__(self, score, weight):
        self.score = score
        self.weight = weight


class Subject(object):
    def __init__(self):
        self._grades = []

    def report_grade(self, score, weight):
        self._grades.append(Grade(score, weight))

    def average_grade(self):
        total, total_weight = 0, 0
        for grade in self._grades:
            total += grade.score * grade.weight
            total_weight += grade.weight
        return total / total_weight


class Student(object):
    def __init__(self):
        self._subjects = {}

    def subject(self, name):
        if name not in self._subjects:
            self._subjects[name] = Subject()
        return self._subjects[name]

    def average_grade(self):
        total, count = 0, 0
        for subject in self._subjects.values():
            total += subject.average_grade()
            count += 1
        return total / count


class Gradebook(object):
    def __init__(self):
        self._students = {}

    def student(self, name):
        if name not in self._students:
            self._students[name] = Student()
        return self._students[name]


book = Gradebook()
albert = book.student('Albert Einstein')
math = albert.subject('Math')
math.report_grade(80, 0.1)
math.report_grade(90, 0.5)

print(albert.average_grade())

# 比较差的写法：差的理由
# 1. 比如无法返回某个人某门课的成绩
# 2. 无法知道grades里面具体内容是什么
class Gradebook(object):
    def __init__(self):
        self._grades = {}

    def add_student(self, name):
        self._grades[name] = {}

    def report_grade(self, name, subject, score, weight):
        by_subject = self._grades[name]  # 还是字典
        grade_list = by_subject.setdefault(subject, [])
        grade_list.append((score, weight))

    def average_grade(self, name):
        by_subject = self._grades[name]
        score_sum, score_cnt = 0, 0
        for subject, scores in by_subject.items():
            subject_avg, total_weight = 0, 0
            for score, weight in scores:
                subject_avg += score * weight
                total_weight += weight
            score_sum += subject_avg / total_weight
            score_cnt += 1
        return score_sum / score_cnt


book = Gradebook()
book.add_student('Albert Einstein')
book.report_grade('Albert Einstein', 'Math', 80, 0.1)
book.report_grade('Albert Einstein', 'Math', 90, 0.5)
print(book.average_grade('Albert Einstein'))
```

## 23. 简单的接口应该接受函数，而不是类的实例

- 对于连接各种Python组件的简单接口来说，通常应该给其直接传入函数，而不是先定义某个类，然后再传入该类的实例

  ```python
  names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
  names.sort(key=lambda x: len(x))  # 这种简单的接口直接采用函数即可
  print(names)
  ```

- Python中的函数和方法都可以像一级类那样引用，因此，它们与其他类型的对象一样，也可能够放在表达式里面

  ```python
  class SortLen(object):
      def sort_len(self, x):
          return len(x)
  
  names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
  
  sort_class = SortLen()
  names.sort(key=sort_class.sort_len)
  print(names)
  ```

- 通过名为`__call__`的特殊方法，可以使类的实例能够像普通的Python函数那样得到调用

  ```python
  class SortLen(object):
      def __call__(self, x):
          return len(x)
  
  names = ['Socrates', 'Archimedes', 'Plato', 'Aristotle']
  
  sort_class = SortLen()
  names.sort(key=sort_class)  # 注意: 这里并没有显示指定方法
  print(names)
  ```

- 如果要用函数来保存状态，那就应该定义新的类，并令其实现`__call__`方法，而不要定义带状态的闭包

  ```python
  class SortLen(object):
      def __init__(self):
          self.cnt = 0
  
      def __call__(self, x):
          self.cnt += 1
          return len(x)
  
  n1 = ['aaaa', 'aa', 'aaa', 'a']
  n2 = ['bbbb', 'bb', 'bbb', 'b']
  sort_class = SortLen()
  n1.sort(key=sort_class)
  print(sort_class.cnt)   # 输出4
  n2.sort(key=sort_class)
  print(sort_class.cnt)   # 输出8
  ```

## 24. 以@classmethod形式的多态去通用地构建对象

- 在Python程序中，每个类只能有一个构造器，也就是`__init__`方法

- 通过`@classmethod`机制，可以用一种与构造器相仿的方式来构造类的对象

  ```python
  class Base(object):
      def print_info(self):
          raise NotImplementedError
  
      @classmethod
      def generate_cls(cls):  # 可以根据类来判别返回的对象
          return cls()
  
  class A(Base):
      def print_info(self):
          print('A')
  
  class B(Base):
      def print_info(self):
          print('B')
  
  class Worker(object):
      @classmethod
      def create_worker(cls, input_cls):  # 可以认为该类可以处理任意Base的子类
          input_cls.print_info()
  
  Worker.create_worker(A.generate_cls())
  Worker.create_worker(B.generate_cls())
  ```

- 通过类方法多态机制，我们能够以更加通用的方式来构建并拼接具体的子类

  > 多态：使得继承体系中的多个类都能以各种所独有的方式来实现某个方法：即接口相同，但实现不同

## 25. 用super初始化父类

- Python采用标准的方法解析顺序来解决超类初始化次序及钻石继承问题

  ```python
  class MyBaseClass(object):
      def __init__(self, value):
          self.value = value
  
  # -----------------------超类初始化顺序问题------------------------
  class TimesTwo(object):
      def __init__(self):
          self.value *= 2
  
  class PlusFive(object):
      def __init__(self):
          self.value += 5
  
  class OneWay(MyBaseClass, TimesTwo, PlusFive):  # 多重继承情况
      def __init__(self, value):
          MyBaseClass.__init__(self, value)
          TimesTwo.__init__(self)
          PlusFive.__init__(self)
  
  foo = OneWay(5)
  print(foo.value)  # 输出15： 先MyBaseClass, 再TimesTwo, 再PlusFive
  
  class AnotherWay(MyBaseClass, PlusFive, TimesTwo):  # 多重继承情况
      def __init__(self, value):
          MyBaseClass.__init__(self, value)
          TimesTwo.__init__(self)
          PlusFive.__init__(self)
  
  foo = AnotherWay(5)
  print(foo.value)  # 输出15, 因此顺序取决于内部初始化__init__调用的顺序
  
  # -------------------------钻石型继承问题--------------------------
  class TimesFive(MyBaseClass):
      def __init__(self, value):
          MyBaseClass.__init__(self, value)
          self.value *= 5
  
  class PlusTwo(MyBaseClass):
      def __init__(self, value):
          MyBaseClass.__init__(self, value)
          self.value += 2
  
  class ThisWay(TimesFive, PlusTwo):
      def __init__(self, value):
          TimesFive.__init__(self, value)  # MyBaseClass
          PlusTwo.__init__(self, value)  # 再次调用BaseClass
  
  foo = ThisWay(5)
  print(foo.value)  # 输出7; 第一次调用TimesFive, self.value变为25; 但调用PlusTwo的时候self.value=value执行后变为5, 因此最终的self.value变为7        
  ```

- 总是应该使用内置的super函数来初始化父类

  ```python
  class TimesFive(MyBaseClass):
      def __init__(self, value):
          super().__init__(value)
          self.value *= 5
  
  class PlusTwo(MyBaseClass):
      def __init__(self, value):
          super().__init__(value) # 等价于super(__class__, self).__init(value)
          self.value += 2
  
  class GoodWay(TimesFive, PlusTwo):
      def __init__(self, value):
          super().__init__(value) 
  
  foo = GoodWay(5) # 输出35
  print(foo.value)
  # 说明：此时调用的顺序会采用MRO标准的流程来安排超类之间的初始化顺序，也保证钻石顶部的公共基类的__init__方法只会运行一次
  # 可以通过GoodWay.mro()查看具体初始化的顺序
  ```

## 26. 只在使用Mix-in组件制作工具类时进行多重继承

- 能用`mix-in`组件实现的效果，就不要用多重继承来做

  ```python
  # 这个例子可能不太合适
  class MixIn(object):
      def count_dict(self):  # 统计属性个数的插件
          return len(self.__dict__)
  
  class A(MixIn):
      def __init__(self):
          self.a = 10
          self.b = 20
          self.c = 30
  
  class B(MixIn):
      def __init__(self):
          self.b = 20
          self.a = 10
  
  a = A()
  print(a.count_dict())
  b = B()
  print(b.count_dict())
  ```

  > `mix-in`是一种小型的类，它只定义了其他类可能需要提供的一套附加方法，而不定义自己的实例属性，此外，它也不要求使用者调用自己的`__init__`构造器

- 将各种功能实现为可插拔的`mix-in`组件，然后令相关的类继承自己需要的那些组件，即可定制该类实例所具备的行为
- 把简单的行为封装到`mix-in`组件里，然后就可以用多个`mix-in`组合出复杂的行为了

## 27. 多用public属性，少用private属性

- Python编译器无法严格保证private字段的私密性

  ```python
  class MyObject(object):
      def __init__(self):
          self.public_field = 5
          self.__private_field = 10
  
  foo = MyObject()
  print(foo.public_field)
  # print(foo.__private_field)  # 报错
  print(foo._MyObject__private_field)  # 可以访问
  ```

  > 在Python中，只是将私有属性`__private_field`变换为`_MyObject__private_field` （即这个私有属性名字被改了）来保证私有性，因此其实外部还是有方法可以访问的。可以通过`print(foo.__dict__)`来看到所有的属性

- 不要盲目地将属性设为`private`，而是应该从一开始就做好规划，并允许子类更多地访问超类的内部`API`（因为子类无法访问父类的private字段）

  ```python
  class MyParentObj(object):
      def __init__(self):
          self.__private_field = 71
  
  class MyChildObj(MyParentObj):
      def get_private_field(self):
          return self.__private_field  # 错误, 无法范问
  ```

  > 因为只存在`_MyParentObj__private_field`

- 应该多用`protected`属性，并在文档中把这些字段的合理用法告诉子类的开发者，而不要试图用`private`属性来限制子类访问这些字段

  > 所谓的`protected`属性，其实就是一种"约定俗成的习惯"，将`self._xxx`认为是`protected`属性，其实就是`public`属性

- 只有当子类不受自己控制时，才可以考虑用`private`属性来避免名称冲突

  ```python
  class ApiClass(object):
      def __init__(self):
          self._value = 5
  
      def get(self):
          return self._value
  
  class Child(ApiClass):
      def __init__(self):
          super().__init__()
          self._value = 'hello'
  
  a = Child()
  print(a.get())  # 返回的是hello
  # 如果你希望a.get()不被子类内容覆盖掉, 可以如下
  class ApiClass(object):
      def __init__(self):
          self.__value = 5
  
      def get(self):
          return self.__value
  
  class Child(ApiClass):
      def __init__(self):
          super().__init__()
          self._value = 'hello'
  
  a = Child()
  print(a.get())  # 返回的是5
  ```

## 28. 继承collections.abc以实现自定义的容器类型

- 如果要定制的子类比较简单，那就可以直接从Python的容器类型（如`list`或`dict`）中继承

  ```python
  # 比如希望一个类能够统计每个元素出现频率的list类
  class FrequenceList(list):  # 具备list所有的方法
      def __init__(self, members):
          super().__init__(members)
  
      def frequency(self):
          counts = {}
          for item in self:
              counts.setdefault(item, 0)
              counts[item] += 1
          return counts
  
  foo = FrequenceList([1, 2, 3, 4, 1, 2])
  print(len(foo))
  print(foo.frequency())
  ```

- 想正确实现自定义的容器类型，可能需要编写大量的特殊方法

- 编写自制的容器类型时，可以从`collections.abc`模块的抽象基类中继承，那些基类能够确保我们的子类具备适当的接口及行为

  ```python
  from collections.abc import Sequence
  
  class SeqCls(Sequence): # 继承Sequence必须实现__getitem__和__len__方法
      def __init__(self, members):
          super().__init__()
          self.members = members
  
      def __getitem__(self, item): 
          return self.members[item]
  
      def __len__(self):
          return len(self.members)
  
  seq_cls = SeqCls([1, 2, 3, 4])
  for e in seq_cls:
      print(e)
  ```

  > 核心就在于这个类定义了一系列抽象基类，提供了每一种容器类型所具备的常用方法。此时，基类继承该子类之后，如果忘记实现某个方法，`collections.abc`就会指出这个错误
  >
  > [collections.abc](https://docs.python.org/3/library/collections.abc.html)

