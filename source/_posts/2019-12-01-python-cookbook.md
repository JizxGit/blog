---
title: python cookbook
tags:
  - python
  - time
  - ing
categories:
  - python
date: 2019-12-01 22:52:58
---
<Excerpt in index | 首页摘要> 

[python cookbook](https://python3-cookbook.readthedocs.io/zh_CN/latest/index.html)的阅读笔记，对内容进行了简练。将常用的知识点记录下来。

<!-- more -->
<The rest of contents | 余下全文>

## 数据结构

### 字典

#### 字典的构造方式

```python
# a,b,c是常用的方式
a = dict(one=1, two=2, three=3) # 注意 one、two、three表示字符串，不用加引号
b = {'one': 1, 'two': 2, 'three': 3}
c = dict(zip(['one', 'two', 'three'], [1, 2, 3]))
d = dict([('two', 2), ('one', 1), ('three', 3)])
e = dict({'three': 3, 'one': 1, 'two': 2})
a == b == c == d == e # True
```

#### 字典键值的获取

```python
one = a['four'] # 键不存在时，会报错KeyError，因此推荐get方法，设置默认值
one = a.get('four')
# None
one=a.get('four',4) # get(key [,default])
# 4
```

#### 字典键值的添加

```python
a['key'] = value
a.setdefault("key"[,default])   # 不要被这个名字给骗了，注意不会使用default修改原来的值
# 如果key在字典中，返回他的value. If not, 插入该键，设置为default 并返回 default。default 默认是 None。注意不会使用default修改原来的值。
```

个人觉得与get的差别只有 不存在给定的键时会不会在字典中添加该键。

例子

```python
strings = ('puppy', 'kitten', 'puppy', 'puppy','weasel', 'puppy', 'kitten', 'puppy')
counts = {}
for kw in strings:
    # counts[kw] += 1 会抛出KeyError异常
    counts[kw] = counts.setdefault(kw, 0) + 1
    # counts[kw] = counts.get(kw, 0) + 1 效果相同
```

#### 设置默认值

有没有一种字典它本身提供了默认值的功能呢？答案是肯定的，那就是 `collections.defaultdict` 。

定义：`defaultdict`([*default_factory*[, *...*]])

> `default_factory`：必须是`callable`类型，因此可以是类型（如int、list）、函数。在 [`__missing__()`](https://docs.python.org/3/library/collections.html#collections.defaultdict.__missing__) 方法中会使用default_factory ; default_factory使用第一个参数初始化，否则为None。
> 
> `__missing__`：在调用 [`__getitem__()`](https://docs.python.org/3/reference/datamodel.html#object.__getitem__) 方法或者`dict[key]`未找到key时，会调用`__missing__`(*key*)方法。 如果default_factory 是 None，则抛出异常。如果不是None，则为给定键提供默认值，插入到字典中，并返回。

注意：这种形式的默认值只有在通过 `dict[key] `或者 `dict.__getitem__(key)` 访问的时候才有效。

**default_factory：类型**

```python
from collections import defaultdict
s = 'mississippi'
d = defaultdict(int) # 默认为0
for k in s:
     d[k] += 1

sorted(d.items())
# [('i', 4), ('m', 1), ('p', 2), ('s', 4)]
```

```python
from collections import defaultdict
s = [('yellow', 1), ('blue', 2), ('yellow', 3), ('blue', 4), ('red', 1)]
d = defaultdict(list)
for k, v in s:
     d[k].append(v)

sorted(d.items())
[('blue', [2, 4]), ('red', [1]), ('yellow', [1, 3])]
```

**default_factory：函数、lambda 函数**

```python
# 提供函数
from collections import defaultdict
def constant_factory(value):
    return lambda: value

d = defaultdict(constant_factory('<missing>')) # 
d.update(name='John', action='ran')
print('%(name)s %(action)s to %(object)s' % d)
# 'John ran to <missing>'
```

```python
# 提供lambda 函数
from collections import defaultdict
s = 'mississippi'
d = defaultdict(lambda : 0) # 默认为0
for k in s:
     d[k] += 1

sorted(d.items())
# [('i', 4), ('m', 1), ('p', 2), ('s', 4)]
```



### 队列与堆

#### deque队列

- 在队列**两端插入或删除**元素时间复杂度都是 `O(1)` ，区别于列表，在列表的开头插入或删除元素的时间复杂度为 `O(N)` 。

```python
from collections import deque
>>> q = deque()
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3])
>>> q.appendleft(4)
```



- 使用 `deque(maxlen=N)` 构造函数会新建一个固定大小的队列。当新的元素加入并且这个队列已满的时候， 最老的元素会自动被移除掉。

```
>>> q = deque(maxlen=3)
>>> q.append(1)
>>> q.append(2)
>>> q.append(3)
>>> q
deque([1, 2, 3], maxlen=3)
>>> q.append(4)
>>> q
deque([2, 3, 4], maxlen=3)
>>> q.append(5)
>>> q
deque([3, 4, 5], maxlen=3)
```



#### heapq堆(TODO )

在底层实现里面，首先会先将集合数据**进行堆排序后放入一个列表中**



### 解压赋值、星号表达式

#### 解压赋值给多个变量

- 任何的序列（或者是可迭代对象）可以通过一个简单的赋值语句解压并赋值给多个变量。 唯一的前提就是**变量的数量必须跟序列元素的数量是一样的。**

- 解压赋值可以用在**任何可迭代对象**上面，而不仅仅是列表或者元组。 包括字符串，文件对象，迭代器和生成器。

```
s = 'Hello'
a, b, c, d, e = s
```

- 你可能只想解压一部分，丢弃其他的值。可以使用任意变量名去占位，到时候丢掉这些变量就行了，最常用的占位符就`_`。

```
data = [ 'ACME', 50, 91.1, (2012, 12, 21) ]
_, shares, price, _ = data
```



#### 变量与星号

扩展的迭代解压语法是专门**为解压不确定个数或任意个数元素的可迭代对象**而设计的。

假设你现在有一些用户的记录列表，每条记录包含一个名字、邮件，接着就是<u>不确定数量</u>的电话号码。 你可以像下面这样分解这些记录：

```python
# 可以解压元组
record = ('Dave', 'dave@example.com', '773-555-1212', '847-555-1212')
name, email, *phone_numbers = record
print(name)             #'Dave'
print(email)            #'dave@example.com'
print(phone_numbers)    # ['773-555-1212', '847-555-1212']
```

你有一个公司前 8 个月销售数据的序列， 但是你想看下最近一个月数据和前面 7 个月的平均值的对比。

```python
# 可以解压列表
*trailing, current = [10, 8, 7, 1, 9, 5, 10, 3]
print(trailing)            # [10, 8, 7, 1, 9, 5, 10]
print(current)            `# 3
trailing_avg = sum(trailing_qtrs) / len(trailing_qtrs)
```

在学期末的时候， 统计家庭作业的平均成绩，但是排除掉第一个和最后一个分数。

```
first, *middle, last = grades
avg(middle)
```



**星号`*`解压出的变量永远都是list列表类型** ，不管解压的数量是多少(包括 0 个)。试着理解下面这些容易让人困惑的点：

- `*elements = iterable` 会让 `elements` 变为 list 类型；

- 但是 `elements = *iterable,` 会让 `elements` 变成 tuple类型

  > 因为这等价于`elements= e1,e2...en,`(e是iterable对象中的元素)，`e1,e2...en,`就是tuple类型，试试这个`a=1,`，a就成了一个tuple对象，等价于`a=(1,)`

理解上面的以后，下面的就能理解了吧。

- `first, *elements = *iterable,` ，elements是list类型

- `*elements, = *iterabl,` ，elements是list类型。

  > 注意：`*elements = *iterabl,`
  > 
  > SyntaxError: starred assignment target must be in a list or tuple
  > 
  > 报错提示：星号赋值的目标element必须在list、tuple中，因此像上面的写法（加个逗号）才是对的。

#### 函数与星号

一、函数定义中的星号

函数定义中的星号，`def func(a,*b)` 表示接受可变参数，分配给位置参数后剩下的参数(关键字参数、`**`解压的字典参数除外)会被封装为tuple。

整体的参数定义的**顺序必须是：必选参数、默认参数、可变参数、命名关键字参数和关键字参数**。

在一个函数的接收参数中，同时出现"非关键字参数（位置参数）"和"关键字参数"时，可以使用一个**单星号**来分隔这两种参数，这种叫做**命名关键字参数**。命名关键字参数可以有默认值，例子如下：

```python
def mix(a,b,*,x,y=4):
    """位置参数与命名关键字参数混合"""
    return a,b,x,y

# 星号前面的a和b是位置参数，星号后面的x和y是关键字参数，并且 x 有默认值
# 调用mix()函数并传入参数时，关键字参数一定要使用"变量名=值"的形式传入数据，如果同位置参数一样传入数据，就会引发一个TypeError异常

print(mix(1,2,x=3,y=4))
print(mix(1,2,x=3))
# print(mix(1,2,3,4)) 
# Error, mix() takes 2 positional arguments but 4 were given
```



如果函数定义中已经有了一个可变参数，后面跟着的命名关键字参数就不再需要一个特殊分隔符*了（**func2中的 b 是命名关键字参数，不是默认参数**），但是调用时必须显示指明给命名参数的赋值，否则会被可变参数全部接收，无法改变默认值。出现你意料之外的问题，如下例子：

```python
# 位置参数，默认参数，可变参数
def func1(a,b=3,*c):
    print(vars())

func1(1,2,3,4)     # 2会赋值给默认参数b

# 位置参数，可变参数，命名关键字参数
def func2(a,*c,b=3):
    print(vars())

func2(1,2,3,4)     # 2，3，4全部被可变参数接收，默认参数b不会被改变
func2(1,2,3,b=4)   # 除非显示指定要改变默认参数

 # {'c': (3, 4), 'b': 2, 'a': 1}
 # {'c': (2, 3, 4), 'b': 3, 'a': 1}
 # {'c': (2, 3), 'b': 4, 'a': 1}
```



综合的例子：

```python
# a,b 是位置参数，c 是默认参数，d 是可变参数，e,f 是命名关键字参数，并且 e 有默认值，kw 是关键字参数
def func(a,b,c=1,*d,e=1,f,**kw):
    print(vars())
    
d={'m':4}
func(*(1,2),3,*(4,5),6, e=2,f=3,**d,**{'x': 2}, y=3,**{'z': 3},g=5)
# 等价于如下：
func(1,2,3,4,5,6,e=2,f=3,**{'m':4,'x':2,'y':3,'z':3,'g':5})
```

关键字参数和命名关键字参数的区别在于：**前者可以传递任何名字的参数，而后者只能传递\*后面名字的参数。**



二、函数调用时的星号

在调用函数时，`func(1,*[2,3,4,5])`， `*`操作符会将 iterable对象（如 list、 tuple）中的元素提取出来，就像他们是另外的位置参数，替换了原本iterable对象的位置，因此相当于`func(1,2,3,4,5)`，对于字典对象的解压也是一样的。

>在 version 3.5之前，只能接收一个`*` 和 `**` unpackings，之后( [**PEP 448**](https://www.python.org/dev/peps/pep-0448)) 发生了一些改变： 函数调用时，允许接受**任意多个** `*` 和 `**` unpackings。

因此在理解调用函数时出现的`*`与`**`，**将他们替换为"位置参数"和"关键字参数"即可。**因此在调用时，位置参数 与 `*` 表达式参数 没有位置顺序的限制 ，关键字参数 与`**`表达式参数也没有位置顺序的限制。

参数顺序问题：

函数调用也是遵循参数定义的顺序规则，关键字参数 必须在 位置参数 之后， `**` 表达式参数 在 `*` 表达式参数之后。

```python
# 例子
def func(a,b,c=1,*d,e=1,**kw):
    print(vars())
    
d={'m':4}

# 调用时的顺序，以及任意多个 unpacking 参数
func(*(1,2),3,*(4,5),6, e=2, **d,**{'x': 2}, y=3,**{'z': 3},f=5)
# 等价于 func(1,2,3,4,5,6,e=2,{'m':4,'x':2,'y':3,'z':3,'f':5})  记住诀窍：将*与**替换为"位置参数"和"关键字参数"即可
```
虽然“`*`表达式参数“可以出现在"关键字参数"后面，但是 它会比“关键字参数”（`**`表达式）**优先处理** ，相当于python 帮你提前了。(在这里那个替换诀窍暂时不能用。)
```python
def f(a, b):
    print(a, b)
f(1, *(2,))
# 1 2

f(b=1, *(2,))
# 2 1

f(a=1, *(2,)) # 2先被赋值给 a，因此 a=1 会导致重复赋值
# TypeError: f() got multiple values for keyword argument 'a'
```

重复键问题：

字典与Function 对待重复键是不同的，字典允许重复键，但函数会引发error。比如如果一个位置参数`def func(a)` 通过 位置、关键字这两种方法提供了值`func(1,a=3)`就会引发错误；使用 `**` unpackings引起得出重复，比如 `f(**{'x': 2},**{'x': 3})`，也会引发错误。



#### zip与星号(\*)

zip 方法在 Python 2 和 Python 3 中的不同：

- 在 Python 3 中为了减少内存，zip() 返回的是一个对象。如需展示列表，需手动 list() 转换。
- 在python3中（**有个大坑**），处于优化内存的考虑，**只能访问一次**(操作一次），内存就会释放

1. 如果各个迭代器的元素个数不一致，则返回列表长度与最短的对象相同

   ```python
    a = [1,2,3]
    b = [4,5,6]
    c = [7,8,9,10,11]
   
    lis=list(zip(a,b,c)) 
    print(lis)
    # [(1, 4, 7), (2, 5, 8), (3, 6, 9)]
   ```

2. 星号*进行解压，解压的效果：提取iterable对象内的元素

   ```python
   a = [1,2,3]
   b = [4,5,6]
   zipped_list = list(zip(a,b))
   print(zipped_list) 
   print(*zipped_list)
   # [(1, 4), (2, 5), (3, 6)]
   # (1, 4) (2, 5) (3, 6)
   ```

3. zip 与 \*zip 进行逆操作

   ```python
   a = [1,2,3]
   b = [4,5,6]
   lis = zip(a,b)   # [(1, 4), (2, 5), (3, 6)]
   result = zip(*lis) # 等价于 zip( (1, 4), (2, 5), (3, 6) )
   aa,bb = list(result)
   print(aa,bb)
   # (1, 2, 3), (4, 5, 6) 又获得原来的a、b
   ```

## 字符串处理

### 用Shell通配符匹配字符串

`fnmatch` 模块提供了两个函数—— `fnmatch()` 和 `fnmatchcase()` ，可以用 **Unix Shell** 中常用的通配符(比如 `*.py` , `Dat[0-9]*.csv` 等)去匹配文本字符串。

`fnmatch()` 函数匹配能力介于简单的字符串方法和强大的正则表达式之间。 如果在数据处理操作中只需要简单的通配符就能完成的时候，这通常是一个比较合理的方案。

如果你的代码需要做文件名的匹配，最好使用 `glob` 模块。参考5.13小节。

```python
from fnmatch import fnmatch, fnmatchcase

print(fnmatch('foo.txt', '*.txt'))
print(fnmatch('foo.txt', '?oo.txt'))
print(fnmatch('Dat45.csv', 'Dat[0-9]*'))
names = ['Dat1.csv', 'Dat2.csv', 'config.ini', 'foo.py']
[name for name in names if fnmatch(name, 'Dat*.csv')]
# True
# True
# True
# ['Dat1.csv', 'Dat2.csv']
```

`fnmatch()` 函数使用**底层操作系统的大小写敏感规则**(不同的系统是不一样的)来匹配模式。比如：

```
# On OS X (Mac)
fnmatch('foo.txt', '*.TXT')
# False
# On Windows
fnmatch('foo.txt', '*.TXT')
# True
```

如果你对这个区别很在意，可以使用 `fnmatchcase()` 来代替。它完全使用你的模式大小写匹配。比如：

```
fnmatchcase('foo.txt', '*.TXT')
# False
```

### 正则表达式匹配字符串

如果你想匹配的是字面字符串，那么你通常只需要调用<u>基本字符串方法</u>就行， 比如 `str.find()` , `str.endswith()` , `str.startswith()` 或者类似的方法。

对于复杂的匹配需要使用正则表达式和 `re` 模块， 核心步骤就是先使用 `re.compile()` 编译正则表达式字符串， 然后使用 `match()` , `search()`,`findall()` 或者 `finditer()` 等方法。

> 需要注意的是 `match()` 方法仅仅检查字符串的从头开始部分，出现在字符串中间的子串不会匹配到。它的匹配结果有可能并不是你期望的那样。

如果你仅仅是做一次简单的文本匹配/搜索操作的话，可以略过编译部分，直接使用 `re` 模块级别的函数。如果你打算做大量的匹配和搜索操作的话，最好先编译正则表达式，然后再重复使用它。 

> 模块级别的函数会将**最近编译过的模式缓存起来**，因此并不会消耗太多的性能， 但是如果使用预编译模式的话，你将会进一步减少查找和一些额外的处理损耗。

当写正则式字符串的时候，相对普遍的做法是使用原始字符串比如 `r'(\d+)/(\d+)/(\d+)'` 。 这种字符串将不去解析反斜杠，这在正则表达式中是很有用的。 如果不这样做的话，你必须使用两个反斜杠，类似 `'(\\d+)/(\\d+)/(\\d+)'` 。

### 字符串搜索和替换

对于简单的字面模式，直接使用 `str.replace()` 方法即可，对于复杂的模式，请使用 `re` 模块中的 `sub()` 函数。如将形式为 `11/27/2012` 的日期字符串改成 `2012-11-27` 。示例如下：

```python
import re
text = 'Today is 11/27/2012. PyCon starts 3/13/2013.'
re.sub(r'(\d+)/\d+)/(\d+)', r'\3-\1-\2', text)
# 'Today is 2012-11-27. PyCon starts 2013-3-13.'
```

`sub()` 函数中的第一个参数是被匹配的模式，第二个参数是替换模式。反斜杠数字比如 `\3` 指向前面模式的捕获组号。

`sub()` 函数除了接受替换字符串外，还能接受一个回调函数。

```python
import re
text = 'UPPER PYTHON, lower python, Mixed Python'
re.findall('python', text, flags=re.IGNORECASE)
# ['PYTHON', 'python', 'Python']

re.sub('python', 'snake', text, flags=re.IGNORECASE) # 替换为字符串
# 'UPPER snake, lower snake, Mixed snake'

def matchcase(word):
    # 定义 处理不同match时替换为不同的字符串 的函数
    def replace(m):
        text = m.group()
        if text.isupper():
            return word.upper()
        elif text.islower():
            return word.lower()
        elif text[0].isupper():
            return word.capitalize()
        else:
            return word
    return replace

re.sub('python', matchcase('snake'), text, flags=re.IGNORECASE) # 使用回调函数
# 'UPPER SNAKE, lower snake, Mixed Snake'
```

### 多行匹配模式

```python
import re
comment = re.compile(r'/\*(.*?)\*/')
text1 = '/* this is a comment */'
text2 = '''/* this is a
multiline comment */
'''
comment.findall(text1)
# [' this is a comment ']
comment.findall(text2)
# []

re.compile(r'/\*(.|\n)*?\*/')
```

### 将Unicode文本标准化(统一unicode文本)

处理Unicode字符串，需要**确保所有字符串在底层有相同的表示**。

因为在Unicode中，**某些字符能够用多个合法的编码表示**。为了说明，考虑下面的这个例子：

```python
s1 = 'Spicy Jalape\u00f1o'
s2 = 'Spicy Jalapen\u0303o'
print(s1)
# 'Spicy Jalapeño'
print(s1)
# 'Spicy Jalapeño'
s1 == s2
# False
len(s1)
# 14
len(s2)
# 15
```

为了修正这个问题，你可以使用unicodedata模块先将文本标准化：

```python
import unicodedata
t1 = unicodedata.normalize('NFC', s1)
t2 = unicodedata.normalize('NFC', s2)
print(t1 == t2)
# True
print(ascii(t1))
'Spicy Jalape\xf1o'
t3 = unicodedata.normalize('NFD', s1)
t4 = unicodedata.normalize('NFD', s2)
print(t3 == t4)
# True
print(ascii(t3))
# 'Spicy Jalapen\u0303o'
```

#### 标准化的标准

`normalize()` 第一个参数指定字符串标准化的方式。 NFC表示**字符应该是整体**组成(比如可能的话就使用单一编码，会把’eu0301’2个字节压缩到1个字节’é’。)，而NFD表示字符应该**分解为多个组合字符**表示。

python同样支持扩展的标准化形式NFKC和NFKD，它们在处理某些字符的时候增加了额外的兼容特性。比如：

```python
s = '\ufb01' # 一个单字节的字符
'ﬁ'
unicodedata.normalize('NFD', s)
'ﬁ'
# 注意组合的字符如何被分解
unicodedata.normalize('NFKD', s)
'fi'
unicodedata.normalize('NFKC', s)
'fi'
```

#### 处理和音字符（TODO 2.12还没看）

在清理和过滤文本的时候字符的标准化也是很重要的。 比如，假设你想清除掉一些文本上面的变音符的时候(可能是为了搜索和匹配)：

```python
t1 = unicodedata.normalize('NFD', s1)
''.join(c for c in t1 if not unicodedata.combining(c))
'Spicy Jalapeno'
```

最后一个例子展示了 `unicodedata` 模块的另一个重要方面，也就是测试字符类的工具函数。**`combining()` 函数可以测试一个字符是否为和音字符。**

#### 正则中匹配unicode字符

如果你想在模式中包含指定的Unicode字符，你可以使用Unicode字符对应的转义序列(比如 `\uFFF`或者 `\UFFFFFFF` )。 

下面是一个匹配几个不同<u>阿拉伯编码页面</u>中所有字符的正则表达式：

```python
>>> arabic = re.compile('[\u0600-\u06ff\u0750-\u077f\u08a0-\u08ff]+')
```

#### 注意大小写转换问题

当执行匹配和搜索操作的时候，最好是先标准化并且清理所有文本为标准化格式。 但是同样也应该注意一些特殊情况，比如在忽略大小写匹配和大小写转换时的行为。

```python
pat = re.compile('stra\u00dfe', re.IGNORECASE)
s = 'straße'
pat.match(s) # 匹配成功
# <_sre.SRE_Match object at 0x10069d370>
pat.match(s.upper()) # 匹配失败
s.upper() # Case folds
# 'STRASSE'
```

### 删除字符串中空白字符（不需要的字符）

#### 开始、结束的字符

用于删除开始或结尾的字符，默认情况下，`strip、lstrip、rstrip`方法会去除空白字符，但是你也可以指定其他字符，**注意strip()指定其他字符的使用，只要在开始或者结尾位置的字符符合指定的字符之一，就会被删除。**

```python
t = '---=-hello===-='
t.lstrip('-')
# '=-hello===-='

t.rstrip('=')
# '---=-hello==-'

t.strip('-=')   #注意该情况
# 'hello'
```

#### 中间的字符

如果你**想处理中间的空格**，那么你需要求助其他技术。比如使用 `replace()` 方法或者是用正则表达式替换。示例如下：

```python
s.replace(' ', '')
# 'helloworld'
import re
re.sub('\s+', ' ', s)
# 'hello world'
```

#### strip与生成器

通常情况下你想将字符串 `strip` 操作和其他迭代操作相结合，比如从文件中读取多行数据。 如果是这样的话，那么生成器表达式就可以大显身手了。比如：

```python
with open(filename) as f:
    lines = (line.strip() for line in f)
    for line in lines:
        print(line)
```

在这里，表达式 `lines = (line.strip() for line in f)` 执行数据转换操作。 **这种方式非常高效**，因为它不需要预先读取所有数据放到一个临时的列表中去。 它仅仅只是创建一个生成器，并且每次返回行之前会先执行 `strip` 操作。

### 字符串对齐

结论：优先选择 `format()` 函数或者方法。

- `format()` 要比 `%` 操作符的功能更为强大。
- `format()` 也比使用 `ljust()` , `rjust()` 或 `center()` 方法更通用(*使用 `<,>` 或者 `^` 字符后面紧跟一个指定的宽度*)， 因为它可以用来格式化任意对象，而不仅仅是字符串。

```python
text = 'Hello World'

# 用空格对齐
format(text, '<20')   # 等价于 text.ljust(20) 
# 'Hello World         '
format(text, '>20')   # 等价于 text.rjust(20)
# '         Hello World'
format(text, '^20')   # 等价于 text.center(20)
# '    Hello World     '

# 用指定字符对齐
format(text, '=>20')  # 等价于 text.rjust(20,'=')
# '=========Hello World'
format(text, '*^20')  # 等价于 text.center(20,'*')
# '****Hello World*****'
```

当格式化多个值的时候，这些格式代码也可以被用在 `format()` 方法中。比如：

```python
'{:>10s} {:>10s}'.format('Hello', 'World')
# '     Hello      World'
```

### 合并拼接字符串

#### 推荐使用 join

如果你仅仅只是合并少数几个字符串，使用加号(+)通常已经足够了。

如果你想要合并的字符串是在一个序列或者 `iterable` 中，那么最快的方式就是使用 `join()` 方法。

```python
parts = ['Is', 'Chicago', 'Not', 'Chicago?']
' '.join(parts)
# 'Is Chicago Not Chicago?'
```

> 初看起来，这种语法看上去会比较怪，但是 `join()` 被指定为字符串的一个方法。 这样做的部分原因是你想去连接的对象可能来自各种不同的数据序列(比如列表，元组，字典，文件，集合或生成器等)， 如果在所有这些对象上都定义一个 `join()`方法明显是冗余的。 

一个相对比较聪明的技巧是**利用生成器表达式**转换数据为字符串的同时合并字符串，比如：

```python
data = ['ACME', 50, 91.1]
','.join(str(d) for d in data)
# 'ACME,50,91.1'
```

#### 避免不必要的拼接

比如在打印的时候：

```python
print(a + ':' + b + ':' + c) # Ugly
print(':'.join([a, b, c])) # Still ugly
print(a, b, c, sep=':') # 推荐
```

当混合使用I/O操作和字符串连接操作的时候，有时候需要仔细研究你的程序。 比如，考虑下面的两端代码片段：

```python
# Version 1 (string concatenation)
f.write(chunk1 + chunk2)

# Version 2 (separate I/O operations)
f.write(chunk1)
f.write(chunk2)
```

如果**两个字符串很小，那么第一个版本性能会更好些**，因为I/O系统调用天生就慢。 另外一方面，如果两个字符串很大，那么第二个版本可能会更加高效， 因为**它避免了创建一个很大的临时结果并且要复制大量的内存块数据**。

### 以指定列宽 格式化字符串

使用 `textwrap` 模块来格式化字符串的输出

```python
s = "Look into my eyes, look into my eyes, the eyes, the eyes, the eyes, not around the eyes, don't look around the eyes, look into my eyes, you're under."
import textwrap
print(textwrap.fill(s, 70))
# Look into my eyes, look into my eyes, the eyes, the eyes, the eyes,
# not around the eyes, don't look around the eyes, look into my eyes,
# you're under.

print(textwrap.fill(s, 40))
# Look into my eyes, look into my eyes,
# the eyes, the eyes, the eyes, not around
# the eyes, don't look around the eyes,
# look into my eyes, you're under.

print(textwrap.fill(s, 40, initial_indent='    '))
#     Look into my eyes, look into my
# eyes, the eyes, the eyes, the eyes, not
# around the eyes, don't look around the
# eyes, look into my eyes, you're under.

print(textwrap.fill(s, 40, subsequent_indent='    '))
# Look into my eyes, look into my eyes,
#    the eyes, the eyes, the eyes, not
#    around the eyes, don't look around
#    the eyes, look into my eyes, you're
#    under.
```

当你希望输出自动匹配终端大小的时候。 你可以使用 `os.get_terminal_size()` 方法来获取终端的大小尺寸。比如：

```python
import os
os.get_terminal_size().columns
```

`fill()` 方法接受一些其他可选参数来控制tab，语句结尾等。 参阅 [textwrap.TextWrapper文档](https://docs.python.org/3.6/library/textwrap.html#textwrap.TextWrapper) 获取更多内容。

### 在字符串中处理html和xml(TODO 未看)

https://python3-cookbook.readthedocs.io/zh_CN/latest/c02/p17_handle_html_xml_in_text.html#htmlxml

### 字符串令牌解析(TODO 未看)

https://python3-cookbook.readthedocs.io/zh_CN/latest/c02/p18_tokenizing_text.html



## 数字日期和时间

### 数字的四舍五入 

#### 简单的四舍五入

使用内置的 `round(value, ndigits)` 函数即可。

**一、关于保留的位数**

传给 `round()` 函数的 `ndigits` 参数可以是**负数**，这种情况下， 舍入运算会作用在十位、百位、千位等上面。比如：

```python
>>> a = 1627731
>>> round(a, -1)
1627730
>>> round(a, -2)
```

**二、关于四舍五入**

```
## python2 ##
round(0.5)
1.0

## python3 ##
round(0.5)
0
```

在python2.7的doc中，`round()`的最后写着保留值将保留到离上一位更近的一端（四舍六入），如果距离两端一样远，则保留到**离0远的一边**。所以round(0.5)会近似到1，而round(-0.5)会近似到-1。

>   Values are rounded to the closest multiple of 10 to the power minus *ndigits*; if two multiples are equally close, rounding is done away from 0

但是到了python3.5的doc中，文档变成了如果距离两边一样远，会**保留到偶数的一边**。比如round(0.5)和round(-0.5)都会保留到0，而round(1.5)会保留到2。

>   values are rounded to the closest multiple of 10 to the power minus *ndigits*; if two multiples are equally close, rounding is done toward the even choice

**三、关于精度问题**

```
round(2.675, 2)
2.67
```

`round(2.675, 2) `的结果，不论我们从python2还是3来看，结果都应该是2.68的，结果它偏偏是2.67，为什么？这跟浮点数的精度有关。我们知道在机器中浮点数不一定能精确表达，因为换算成一串1和0后可能是无限位数的，机器已经做出了截断处理。那么在**机器中保存的2.675这个数字就比实际数字要小那么一点点**。这一点点就导致了它离2.67要更近一点点，所以保留两位小数时就近似到了2.67。

 **总结**

以上。除非对精确度没什么要求，否则尽量避开用`round()`函数。近似计算我们还有其他的选择：

1.  使用math模块中的一些函数，比如`math.ceiling`（天花板除法）。
2.  python自带整除，python2中是`/`，3中是`//`，还有`div`函数。
3.  字符串格式化可以做截断使用，例如 `"%.2f" % value`（保留两位小数并变成字符串，如果还想用浮点数请披上`float()`的外衣）。
4.  当然，对浮点数精度要求如果很高的话，请用decimal模块。

参考 https://www.cnblogs.com/anpengapple/p/6507271.html



### 数值格式化指令

定宽度和精度的一般形式是 `'[<>^]?width[,]?(.digits)?'` ， 其中 `width` 和 `digits` 为整数，`？`代表可选部分。 同样的格式也被用在字符串的 `format()` 方法中。比如：

```
>>> 'The value is {:0,.2f}'.format(x)
'The value is 1,234.57'
```



### 进制

#### 10 进制—>其他

**带前缀的**

使用 `bin()` , `oct()` 或 `hex()` 函数

```
>>> x = 1234
>>> bin(x)
'0b10011010010'
>>> oct(x)
'0o2322'
>>> hex(x)
'0x4d2'
>>>
```

**没有前缀的**

如果不想输出 `0b` , `0o` 或者 `0x` 的前缀的话，可以使用 `format()` 函数

```
>>> format(x, 'b')
'10011010010'
>>> format(x, 'o')
'2322'
>>> format(x, 'x')
'4d2'
>>>
```



#### 其他—>10 进制

其他的进制转换整数，简单的使用带有进制的 `int()` 函数即可：

```python
>>> int('4d2', 16)
1234
>>> int('10011010010', 2)
1234
```



字节到大整数的打包与解包

https://python3-cookbook.readthedocs.io/zh_CN/latest/c03/p05_pack_unpack_large_int_from_bytes.html



### 无穷大与 NaN

**定义**

```
>>> a = float('inf')
>>> b = float('-inf')
>>> c = float('nan')
```



**检测**

测试是否为这些值，使用 `math.isinf()` 和 `math.isnan()` 函数。比如：

```
>>> math.isinf(a)
True
>>> math.isnan(c)
True
>>>
```



**传播**

无穷大数在执行数学计算的时候会传播

NaN值会在所有操作中传播，而不会产生异常

NaN值的一个特别的地方时它们之间的比较操作总是返回False。因此唯一安全的方法就是使用 `math.isnan()` 



### 分数运算

https://python3-cookbook.readthedocs.io/zh_CN/latest/c03/p08_calculating_with_fractions.html



### 随机

#### 随机小数

`random.random()`方法用于生成一个0到1的随机浮点数：`0<=n<1.0`

`random.uniform()` 计算均匀分布随机数,`0<=n<1.0`

#### 随机整数

`random.randint(a,b)`：用于生成一个指定范围内的整数。其中生成的随机数n：a<=n<=b

`random.randrange([start],stop[, step])`：从指定范围内，按指定基数递增的集合中获取一个随机数。如：`random.randrange(10,100,2)`，结果相当于从[10,12,14,16,...,96,98]序列中获取一个随机数。`random.randrange(10,100,2)`在结果上与`random.choice(range(10,100,2))`等效。

#### 随机选择数组中的1个

```python
>>> print random.choice(["JGood","is","a","handsome","body"])
is
```

#### 随机选择数组中的 N个

`random.sample(sequence,k)`：从指定序列中随机获取指定长度的片段，sample函数不会修改原有序列。

```python
>>> list=[1,2,3,4,5,6,7,8,9,10]
>>> print random.sample(list,5) #从list中随机获取5个元素，作为一个片段返回
[1, 6, 10, 8, 3]
```

https://www.cnblogs.com/chamie/p/4917820.html





### 时间处理

#### datetime 模块

`datetime` 模块，可以进行常规的加减计算。

```
from datetime import datetime
a = datetime(2012, 9, 23)
b = datetime(2012, 9, 20)
print(a - b)
# 3 days, 0:00:00
```

为了表示一个时间段，可以创建一个 `timedelta` 实例，就像下面这样：

```python
from datetime import timedelta
a = timedelta(days=2, hours=6)
b = timedelta(hours=4.5)
c = a + b
>>> c.days
2
>>> c.seconds
37800
>>> c.total_seconds() / 3600
58.5
```

在计算的时候，需要注意的是 `datetime` 会自动处理闰年。

**高级库**

如果你需要执行更加复杂的日期操作，比如处理时区，模糊时间范围，节假日计算等等， 可以考虑使用 [dateutil模块](http://pypi.python.org/pypi/python-dateutil)





#### 循环时间

```python
>>> for d in date_range(datetime(2012, 9, 1), datetime(2012,10,1),
                        timedelta(hours=6)):
...     print(d)
...
2012-09-01 00:00:00
2012-09-01 06:00:00
```

这种实现之所以这么简单，还得归功于Python中的日期和时间能够使用标准的数学和比较操作符来进行运算。



#### 字符串与时间对象

**字符串—>时间：strptime**

```python
>>> from datetime import datetime
>>> text = '2012-09-20'
>>> y = datetime.strptime(text, '%Y-%m-%d')
>>> z = datetime.now()
>>> diff = z - y
>>> diff
datetime.timedelta(3, 77824, 177393)
```

**时间—>字符串：strftime**

```python
>>> z
datetime.datetime(2012, 9, 23, 21, 37, 4, 177393)
>>> nice_z = datetime.strftime(z, '%A %B %d, %Y')
>>> nice_z
'Sunday September 23, 2012'
```

#### 时区

https://python3-cookbook.readthedocs.io/zh_CN/latest/c03/p16_manipulate_dates_involving_timezone.html



## 迭代器与生成器

### next函数

一、默认会抛出异常 `StopIteration` 用来指示迭代的结尾。

```python
with open('/etc/passwd') as f:
        try:
            while True:
                line = next(f)
                print(line, end='')
        except StopIteration:
            pass
```

二、通过返回一个指定值来标记结尾

```python
with open('/etc/passwd') as f:
    while True:
        line = next(f, None)   # 设置为不抛出异常，返回 None 的方式
        if line is None:
            break
        print(line, end='')
```



