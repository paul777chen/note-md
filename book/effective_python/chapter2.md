# 第2章：函数

## 14. 尽量用异常来表示特殊情况，而不要返回None

- 用None这个返回值来表示特殊意义的函数，很容易使调用者犯错，因为None和0及空字符串之类的值，在条件表达式里都会评估为False

  ```python
  a = ''  # None, False, '' 输出都是False
  if a:
      print('True')
  else:
      print('False')
  ```

- 函数在遇到特殊情况时，应该抛出异常，而不要返回None。调用者看到该函数的文档中所描述的异常之后，应该就会编写相应的代码来处理它们了

## 15. 了解如何在闭包里使用外围作用域中的变量

- 对于定义在某作用域内的闭包来说，它可以引用这些作用域中的变量

  ```python
  def sort_priority(values, group):
      def helper(x):  # 这个就叫闭包: 定义在某个作用域中的函数(能够访问这个作用域里的变量)
          if x in group:  # 访问了group这个变量
              return (0, x)
          else:
              return (1, x)
  
      values.sort(key=helper)
  ```

- 使用默认方式对闭包内的变量赋值，不会影响外围作用域中的同名变量

  ```python
  def outer():
      flag = False
      def inner():
          flag = True  # 并不改变外围的flag---相当于重新定义了一个flag
      inner()
      print(flag)  # 依旧是False
  ```

- 在Python3中，程序可以在闭包内使用nonlocal语句来修饰某个名称，使该闭包能够修改外围作用域中的同名变量

  ```python
  def outer():
      flag = False
      def inner():
          nonlocal flag  # 此时相当于使用外围的flag
          flag = True
      inner()
      print(flag)  # 输出True
  ```

- 在Python2中（Python3中也可以），程序可以使用可变量（例如，包含单个元素的列表）来实现与nonlocal语句相仿的机制

  ```python
  def outer():
      flag = [False]
      def inner():
          flag[0] = True
      inner()
      print(flag[0])  # 输出True
  ```

- 除了那种比较简单的函数，尽量不要用nonlocal语句

## 16. 考虑用生成器来改写直接返回列表的函数

- 使用生成器比把收集到的结果放入列表里放回给调用者更加清晰

  ```python
  # 1. 采用列表存放所有结果
  def index_words(text):  # 获取一句话每个单词开头的位置
      result = []
      if text:
          result.append(0)
      for index, letter in enumerate(text):
          if letter == ' ':
              result.append(index + 1)
      return result
  
  print(index_words('hello world')) # 返回的是[0, 6]
  # 2. 采用生成器来返回结果
  def index_words_iter(text):
      if text:
          yield 0
      for index, letter in enumerate(text):
          if letter == ' ':
              yield index + 1
  
  print(index_words_iter('hello world'))  # 返回generator对象
  print(list(index_words_iter('hello world'))) # 返回[0, 6]
  ```

- 由生成器函数所返回的那个迭代器，可以把生成器函数体中，传给yield表达式的那些值，逐次产生出来

- 无论输入量有多大，生成器都能产生一系列输出，因为这些输入量和输出量都不会影响它在执行时所耗的内存

  ```python
  def index_file(handle):
      offset = 0
      for line in handle:
          if line:
              yield offset
          for letter in line:
              offset += 1
              if letter == ' ':
                  yield offset
  
  with open(txt_file, 'r') as f: # 此时不管输入的文件有多大, 都可以执行
      it = index_file(f)  # 返回的还是一个generator对象, 可以各种不同方式去执行
  ```

## 17. 在参数上面迭代时，要多加小心

- 函数在输入的参数上面多次迭代时要小心：如果参数是迭代器，那么可能会导致奇怪的行为并错失某些值

  ```python
  def normalize(numbers):
      total = sum(numbers)  # 第一次迭代
      result = []
      for value in numbers:  # 第二次迭代
          percent = 100 * value / total
          result.append(percent)
      return result
  
  # 输入list类型
  visits = [15, 35, 80]
  perc = normalize(visits) # 正常执行
  
  # 输入iterator类型
  visits = iter([15, 35, 80])
  perc = normalize(visits) # 不正常: 第二次迭代为空
  ```

- Python的迭代器协议，描述了容器和迭代器应该如何与`iter`和`next`内置函数、`for`循环及相关表达式相互配合

  > 原理：在执行`for x in foo`这样的语句时，Python实际上会调用`iter(foo)`。内置的`iter`函数又会调用`foo.__iter__`这个特殊方法。该方法必须返回迭代器对象，而那个迭代器本身，则实现了名为`__next__`的特殊方法。此后，`for`循环会在迭代器对象上面反复调用内置的`next`函数，直至其耗尽并产生`StopIteration`异常（真实实现时：只需将自己的类把`__iter__`方法实现为生成器—一般生成器都是自带`next`）

- 把`__iter__`方法实现为生成器，即可定义自己的容器类型

  ```python
  class ReadVisits(object):
      def __init__(self, data_path):
          self.data_path = data_path
  
      def __iter__(self):  # 实现生成器：即iter(ReadVisits对象)
          with open(self.data_path) as f:
              for line in f:
                  yield int(line)
  visits = ReadVisits(path) 
  perc = normalize(visits)   # 此时就正常了
  ```

- 想判断某个值是迭代器还是容器，可以拿该值为参数，两次调用`iter`函数，若结果相同，则是迭代器；调用内置的`next`函数，即可令该迭代器前进一步

  ```python
  a = [1, 2, 3]
  b = iter(a)
  print(iter(a) is iter(a))  # False --- a是容器类型
  print(iter(b) is iter(b))  # True --- b是迭代器类型
  ```

## 18. 用数量可变的位置参数减少视觉杂讯

- 在`def`语句中使用`*args`，即可令函数接受数量可变的位置参数

  ```python
  def log(*values):
      for v in values:
          print(v)
  # demo1
  log(10, 20)
  # demo2
  a = [10, 20] 
  log(*a) # 这里注意*必须有
  ```

- 调用函数时，可以采用`*`操作符，把序列中的元素当成位置参数，传给该函数（如上述的`demo2`）

- 对生成器使用`*`操作符，可能导致程序耗尽内存并奔溃（因为变长参数在传给函数时，总是要先转化为元组tuple）

  ```python
  def log(*values):
      print(type(values))  # 输出tuple
      for v in values:
          print(v)
  ```

- 在已经接受`*args`参数的函数上面继续添加位置参数，可能会产生难以排查的bug（所以可以采用关键字的形式）

  ```python
  def log(*values):
      for v in values:
          print(v)
  
  def log2(new_value, *values):
      for v in values:
          print(v)
  
  def log3(*values, new_value=None):
      for v in values:
          print(v)
  
  log(10, 20)  # 输出10, 20
  log2(10, 20) # 输出20
  log3(10, 20) # 输出10, 20
  ```

## 19. 用关键字参数来表达可选的行为

- 函数参数可以按位置或关键字来指定
- 只是用位置参数来调用参数，可能会导致这些参数值的含义不够明确，而关键字参数则能够阐明每个参数的意图
- 给函数添加新的行为时，可以使用带默认值的关键字参数，以便与原有的函数调用代码保持兼容
- 可选的关键字参数，总是应该以关键字形式来指定，而不应该以未知参数的形式来指定

## 20. 用None和文档字符串来描述具有动态默认值的参数

- 参数的默认值，只会在程序加载模块并读到本函数的定义时评估一次（调用时并不会再次定义）。对于`{}`或`[]`等动态的值，这可能会导致奇怪的行为

  ```python
  # 第一种常见情况
  def log(message, when=datetime.now()):  # 这里的when默认是采用定义时的时间
      print('%s: %s' % (when, message))
  
  log('first')   # 2020-06-03 23:40:32.254045: first
  time.sleep(1)
  log('second')  # 2020-06-03 23:40:32.254045: second
  
  # 可以修改为
  def log_new(message, when=None):
      when = datetime.now() if when is None else when
      print('%s: %s' % (when, message))
      
  log_new('first')  # 2020-06-03 23:39:47.298356: first
  time.sleep(1)
  log_new('second') # 2020-06-03 23:39:48.298446: second
  ```

  ```python
  # 第二种常见情况
  def decode(data, default={}):
      try:
          return json.loads(data)
      except  ValueError:
          return default
  
  foo = decode('bad data')
  foo['stuff'] = 5
  bar = decode('also bad')
  bar['meep'] = 1
  print('Foo', foo)  # Foo {'stuff': 5, 'meep': 1}
  print('Bar:', bar) # Bar: {'stuff': 5, 'meep': 1}
  
  # 可以修改为
  def decode_new(data, default=None):
      if default is None:
          default = {}
      try:
          return json.loads(data)
      except  ValueError:
          return default
  
  foo = decode_new('bad data')
  foo['stuff'] = 5
  bar = decode_new('also bad')
  bar['meep'] = 1
  print('Foo', foo)   # Foo {'stuff': 5}
  print('Bar:', bar)  # Bar: {'meep': 1}
  ```

- 对于以动态值作为实际默认值的关键字参数来说，应该把形式上的默认值写为None，并在函数的文档字符串里面描述该默认值所对应的实际行为

## 21. 用只能以关键字形式指定的参数来确保代码清晰

- 关键字参数能够使函数调用的意图更加明确

- 对于各参数之间很容易混淆的函数，可以声明只能以关键字形式指定的参数，以确保调用者必须通过关键字来指定它们。对于接受多个Boolean标志的函数，更应该这样做

- 在编写函数时，Python3有明确的语法来定义这种只能以关键字形式指定的参数

- Python2的函数可以接受`**kwargs`参数，并手工抛出TypeError异常，以便模拟只能以关键字形式来指定的参数

  ```python
  # Python3
  # *标志着位置参数就此终结，之后的那些参数，都只能以关键字形式来指定
  def division(number, divisor, *, ignore_overflow=False):
      try:
          return number / divisor
      except OverflowError:
          if ignore_overflow:
              return 0
          else:
              raise ValueError('bad')
  
  division(10, 20, True)  # 错误！
  division(10, 20, ignore_overflow=True) # 正确
  
  # Python2
  def division(number, divisor, **kwargs):
      ignore_overflow = kwargs.pop('ignore_overflow', False)
      if kwargs:
          raise TypeError('Unexpected **kwargs: %r' % kwargs)
      try:
          return number / divisor
      except OverflowError:
          if ignore_overflow:
              return 0
          else:
              raise ValueError('bad')
  ```

  