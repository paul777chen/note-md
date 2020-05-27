# 第1章：用Pythonic方式来思考

## 1. 确认自己所用的Python版本

- 命令行形式：`python --version`

- 代码形式：

```python
import sys
# 详细信息
print(sys.version_info)
# 缩略信息
print(sys.version)
```

## 2. 遵循PEP8风格指南

- 遵循PEP风格指南，具体请参考：[PEP8指南](https://legacy.python.org/dev/peps/pep-0008/)

## 3. 了解bytes, str与unicode的区别

- Python3中字符序列以`str`（Unicode字符序列）或者`bytes`（8位值序列---可以先理解为二进制）表示，两者之间的转换：

  ```python
  # bytes --> str
  bytes_var.decode('utf-8')
  # str ---> bytes
  str_var.encode('utf-8')
  ```

- Python2中字符序列以`str`（8位值得序列）或者`unicode`（Unicode字符序列）表示（注意，这里的str相当于python3中的bytes）

- 在Python3中读写文件，有个比较坑的地方就在于windows下默认的字符格式和linux下不同（一般需要人为的在environment variables中添加）

  ```python
  # windows默认open(xxx, 'r', encoding='gbk')
  # linux 默认open(xxx, 'r', encoding='utf-8')
  ```

> 注：
>
> 1. 在pycharm等IDE里面涉及中文可以在Environment里添加：`NAME:LANG, Value:zh_CN.UTF-8`
> 2. 而在shell等可以采用`export LANG=zh_CN.UTF-8`

## 4. 用辅助函数来取代复杂的表达式

- 开发者很容易过度运用Python的语法特性，从而写出那种特别复杂并且难以理解的单行表达式
- 请把复杂的表达式移入辅助函数之中，如果要反复使用相同的逻辑，那就更应该这么做
- 使用if/else表达式，要比用or或and这样的Boolean操作符写成的表达式更加清晰

## 5. 了解切割序列的方法

> 所谓切片就是类似：`list_var[start:end:step]`（step默认为1，一般也称为stride）

- 不要些多余的代码，当start索引为0，或end索引为序列长度时，应该将其省略（这一点个人感觉看情况）
- 切片操作不会计较start与end索引是否越界（比如`a=[1,2,3,4]`，写`a[1:6]`也不会报错，返回的是`[2,3,4]`）（感觉这个功能也没有什么大用途吧？反而容易引起误解）
- 对list赋值的时候，如果使用切片操作，就会把原列表中处于相关范围内的值替换成新值，即便他们的长度不同也依然可以替换！

```python
a=[1,2,3,4,5]
b=a[2:]  # 此时b是一个全新的列表, 修改b并不会对a产生什么影响
a[1:2]=[6,7,8] # 此时的a为[1,6,7,8,3,4,5], 即替换和长度无关
b=a[:]   # 此时b是一个全新的列表，改变b不会对a产生影响
b=a      # 此时b和a是指向同一个列表, 改变b会改变a
```

## 6. 在单次切片操作内，不要同时指定start,end和stride

- 既有start和end，又有stride的切割操作，可能会令人费解（这点不太认同）

- 尽量使用stride为正数，且不带start和end索引的切割操作。尽量避免用负数做stride

  ```
  a=[1,2,3,4,5,6,7,8]
  a[::2]     # 输出[1, 3, 5, 7]
  a[::-2]    # 输出[8, 6, 4, 2]
  a[-2::-2]  # 输出[7, 5, 3, 1]
  ```

  > 其实大致规律就是stride有负号代表往左走，有正号代表往右走（不受到start和end的影响，除非start本身是0那么往左走就从最后一个开始）

- 在同一个切片操作内，不要同时使用start，end和stride。如果确实需要执行这种操作，那就考虑将其拆解为两条赋值语句，其中一条做范围切割，另一条做步进切割，或者考虑使用内置itertools模块中的islice

  ```python
  # 比如a[-2::-2]就有些晦涩了，可以采用
  b = a[::-1] # [8,7,6,5,4,3,2,1]
  c = b[::2]  # [7,5,3,1]
  # 关于itertools的islice模块, 如果只提供一个参数代表的是stop, 返回的是一个迭代器
  # 最大的好处其实是省内存
  islice(iterable, [start,], stop, [,step])
  ```

## 7. 用列表推导来取代map和filter

- 列表推导要比内置的map和filter函数清晰，因为它无需额外编写lambda表达式
- 列表推导可以跳过输入列表中的某些元素，如果改用map来做，那就必须辅以filter方能实现
- 字典与集合也支持列表推导

```python
# 一些等价表达式
a = [1, 2, 3, 4]
b = [x ** 2 for x in a]
c = map(lambda x: x ** 2, a)   # 其实c还是个map类型, 需要list(c)才真正等价
# filter+map
a = [1, 2, 3, 4]
b = [x ** 2 for x in a if x % 2 == 0]
c = map(lambda x: x ** 2, filter(lambda x: x % 2 == 0, a))
```

## 8. 不要使用含有两个以上表达式的列表推导

- 列表推导支持多级循环，每一级循环也支持多项条件
- 超过两个表达式的列表推导是很难理解的，应该尽量避免

## 9. 用生成器表达式来改写数据量较大的列表推导

- 当输入的数据量较大时，列表推导可能会因为占用太多内存而出问题

- 由生成器表达式所返回的迭代器，可以逐次产生输出值，从而避免了内存用量问题

  ```python
  a = [1, 2, 3, 4, 5, 6]  # 假设这是一个非常大的列表推导, a=(x for x in open(xxx))
  b = (t for t in a)  # 获得的是一个generator
  next(b)  # 获得1
  ```

- 把某个生成器表达式所返回的迭代器，放在另一个生成器表达式的for子表达式中，即可将两者组合起来

  ```python
  c=((x, x*2) for x in b)  # 此时c也是一个generator
  next(c)  # 获得1,2
  ```

- 串在一起的生成表达式执行速度很快

## 10. 尽量用enumerate取代range

- enumerate函数提供了一种精简的写法，可以在遍历迭代器时获知每个元素的索引
- 尽量用enumerate来改写那种将range与下标访问结合的序列遍历代码
- 可以给enumerate提供第二个参数，以指定开始计数时所用的值（默认为0）

```python
a = ['a', 'b', 'c']
for i, v in enumerate(a):
    print(i, v)  # 输出0,a; 1,b; 2,c

for i, v in enumerate(a, 2):
    print(i, v)  # 输出2,a; 3,b; 4,c 
```

## 11. 用zip函数同时遍历两个迭代器

- 内置的zip函数可以平行地遍历多个迭代器

  ```python
  a = [1, 2, 3]
  b = ['a', 'b', 'c']
  for ea, eb in zip(a, b):
      print(ea, eb)  # 输出1,a; 2,b; 3,c
  ```

- Python3中的zip相当于生成器，会在遍历过程中逐次产生元组，而Python2中的zip则是直接把这些元组完全生成好，并一次性地返回整份列表

- 如果提供的迭代器长度不等，那么zip就会自动提前终止

  ```python
  a = [1, 2, 3, 4]
  b = ['a', 'b', 'c']
  for ea, eb in zip(a, b):
      print(ea, eb)  # 输出1,a; 2,b; 3,c --- 而没有4
  ```

- itertools内置模块中的zip_longest函数可以平行地遍历多个迭代器，而不用在乎它们的长度是否相等

  ```python
  a = [1, 2, 3, 4]
  b = ['a', 'b', 'c']
  for ea, eb in itertools.zip_longest(a, b):
      print(ea, eb)  # 输出1,a; 2,b; 3,c; 4,None
  ```

## 12. 不要在for和while循环后面写else块

- Python有种特殊语法，可在for及while循环的内部语句块之后紧跟一个else块
- 只有当整个循环主体都没遇到break语句时，循环后面的else块才会执行
- 不要在循环后面使用else块，因为这种语法既不直观，又容易引入误解

## 13. 合理使用try/except/else/finally结构中的每个代码块

- 无论try块是否发生异常，都可以利用try/finally复合语句中的finally块来执行清理工作

  ```python
  try:
  	xxxx
  finally:
  	xxx   # 无论如何该语句都会被执行
  ```

- else块可以用来缩减try块中的代码，并把没有发生异常时所要执行的语句与try/except代码块隔开

  ```python
  try:
  	xxx
  except SOMEERROR:
  	xxx  # 发生特定异常进入此处
  else:
  	xxx  # 没有发生异常进入此处
  ```

- 顺利运行try块后，若想使某些操作能在finally块的清理代码之前执行，则可将这些操作写到else块中





