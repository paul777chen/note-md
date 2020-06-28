# 元类及属性



## 29. 用纯属性取代get和set方法

- 编写新类时，应该采用简单的public属性来定义其接口，而不要手工实现`set`和`get`方法

  ```python
  # 推荐的写法
  class Resistor(object):
      def __init__(self, ohms):
          self.ohms = ohms
  
  resistor = Resistor(10)
  resistor.ohms = 20
  print(resistor.ohms)
  # 不推荐的写法
  class BadResistor(object):
      def __init__(self, ohms):
          self._ohms = ohms
  
      def get_ohms(self):
          return self._ohms
  
      def set_ohms(self, ohms):
          self._ohms = ohms
  
  resistor_bad = BadResistor(10)
  resistor_bad.set_ohms(20)
  print(resistor_bad.get_ohms())
  ```

- 如果访问对象的某个属性时，需要表现出特殊的行为，那就用`@property`来定义这种行为

  ```python
  class Demo(object):
      def __init__(self, ohms):
          super().__init__()
          self.ohms = ohms
          self.current = 0
          self._voltage = 0
  
      @property
      def voltage(self):
          return self._voltage
  
      @voltage.setter
      def voltage(self, voltage):
          self._voltage = voltage
          self.current = self._voltage / self.ohms
  
  demo = Demo(10)
  demo.voltage = 20
  print(demo.current)
  ```

  > `setter`和`getter`方法的名称必须与相关属性相符（即都命名为和`@property`指定的内容相符），方能正常运行

- `@property`方法应该遵循最小惊讶原则，而不应该产生奇怪的副作用
- `@property`方法需要执行得迅速一些，缓慢或复杂的工作，应该放在普通的方法里面

> `@prperty`的最大缺点在于：和属性相关的方法，只能在子类里面共享，而与之无关的其他类，则无法复用同一份实现代码

## 30. 考虑用@property来代替属性重构

- `@property`可以为现有的实例属性添加新的功能
- 可以使用`@property`来逐步完善数据模型
- 如果`@property`用得太过频繁，那就应该考虑彻底重构该类并修改相关的调用代码

```

```

