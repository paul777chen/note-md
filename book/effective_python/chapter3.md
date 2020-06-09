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

  



