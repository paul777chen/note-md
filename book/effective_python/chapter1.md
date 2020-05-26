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





