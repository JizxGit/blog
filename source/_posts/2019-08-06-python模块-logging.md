---
title: python模块-logging
tags:
  - logging
  - python
categories:
  - python
date: 2019-08-06 14:32:08
---
<Excerpt in index | 首页摘要> 

python logging 模块的使用以及详细讲解

<!-- more -->
<The rest of contents | 余下全文>



### 日志级别

logging模块定义了以下几个日志等级，它允许开发人员自定义其他日志级别，但是这是不被推荐的，尤其是在开发供别人使用的库时，因为这会导致日志级别的混乱。

| 日志等级（level） | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| DEBUG             | 最详细的日志信息，典型应用场景是 问题诊断                    |
| INFO              | 信息详细程度仅次于DEBUG，通常只记录关键节点信息，用于确认一切都是按照我们预期的那样进行工作 |
| WARNING           | 当某些不期望的事情发生时记录的信息（如，磁盘可用空间较低），但是此时应用程序还是正常运行的 |
| ERROR             | 由于一个更严重的问题导致某些功能不能正常运行时记录的信息     |
| CRITICAL          | 当发生严重错误，导致应用程序不能继续运行时记录的信息         |

默认情况下Python的logging模块将日志打印到了标准输出中，且只显示了大于等于WARNING级别的日志，这说明**默认的日志级别设置为WARNING**（日志级别等级CRITICAL > ERROR > WARNING > INFO > DEBUG）而日志的信息量是依次增多的；

开发应用程序或部署开发环境时，可以使用DEBUG或INFO级别的日志获取尽可能详细的日志信息来进行开发或部署调试；



### 简单使用

logging模块提供了两种记录日志的方式：

- 第一种方式是使用logging提供的模块级别的函数
- 第二种方式是使用Logging日志系统的四大组件



首先来看一下第一种，通过logging提供的模块级别的函数进行日志记录。

logging模块定义的模块级别的常用函数

| 函数                                   | 说明                                 |
| -------------------------------------- | ------------------------------------ |
| logging.debug(msg, *args, **kwargs)    | 创建一条严重级别为DEBUG的日志记录    |
| logging.info(msg, *args, **kwargs)     | 创建一条严重级别为INFO的日志记录     |
| logging.warning(msg, *args, **kwargs)  | 创建一条严重级别为WARNING的日志记录  |
| logging.error(msg, *args, **kwargs)    | 创建一条严重级别为ERROR的日志记录    |
| logging.critical(msg, *args, **kwargs) | 创建一条严重级别为CRITICAL的日志记录 |
| logging.log(level, *args, **kwargs)    | 创建一条严重级别为level的日志记录    |
| logging.basicConfig(**kwargs)          | 对root logger进行一次性配置          |

其中`logging.basicConfig(**kwargs)`函数用于指定“要记录的日志级别”、“日志格式”、“日志输出位置”、“日志文件的打开模式”等信息，其他几个都是用于记录各个级别日志的函数。



具体例子：

```python
import logging

logging.basicConfig(level=logging.DEBUG,# 控制台打印的日志级别
                    filename='new.log',
                    filemode='w',# 写模式：a和w，w就是写模式，a是追加模式，默认是追加模式
                    format= '%(asctime)s - %(pathname)s[line:%(lineno)d] - %(name)s - %(levelname)s: %(message)s', # 日志格式
                    datefmt='%Y-%m-%d  %H:%M:%S %a '    # 日期格式
                    )

logging.debug("info")
logging.info("info")
logging.warning("warning")
logging.error("error")
logging.critical("critical")
```



#### 关于 basicConfig 说明

主要的配置是通过`basicConfig`来定制的：

| 参数     | 描述                                                         |
| :------- | :----------------------------------------------------------- |
| level    | 指定日志器的日志级别，默认是 WARNING 级别，因此 DEBUG、INFO 都不会输出 |
| filename | 指定日志输出目标文件的文件名（可以写文件名也可以写文件的完整的绝对路径，写文件名日志放执行文件目录下，写完整路径按照完整路径生成日志文件），指定该设置项后日志信息就**不会被输出到控制台了**，如果需要同时显示，请看后面的组件用法。 |
| filemode | 指定日志文件的打开模式，默认为`a`,追加的方式，该选项要在filename指定时才有效 |
| stream   | 指定日志输出目标stream，如sys.stdout、sys.stderr以及网络stream。需要说明的是，stream和filename不能同时提供，否则会引发 `ValueError`异常 |
| handlers | Python 3.3中新添加的配置项。该选项如果被指定，它应该是一个创建了多个Handler的可迭代对象，这些handler将会被添加到root logger。需要说明的是：filename、stream和handlers这三个配置项只能有一个存在，不能同时出现2个或3个，否则会引发ValueError异常。 |
| format   | 指定日志格式字符串，即指定日志输出时所包含的字段信息以及它们的顺序。logging模块定义的格式字段请看**Formater**一节 |
| datefmt  | 指定日期/时间格式。需要注意的是，该选项要在format中**包含时间字段%(asctime)s时才有效** |
| style    | Python 3.2中新添加的配置项。指定format格式字符串的风格，可取值为'%'、'{'和'$'，默认为'%' |

注意点：

- `logging.basicConfig()`函数是一个一次性的简单配置工具使，也就是说**只有在第一次调用该函数时会起作用**，后续再次调用该函数时完全不会产生任何操作的，多次调用的设置并不是累加操作。

- 日志器（Logger）是有层级关系的，上面调用的logging模块级别的函数所使用的日志器是`RootLogger`类的实例，其名称为'root'，它是处于日志器层级关系最顶层的日志器，且**该实例是以单例模式存在的**。

- 如果要记录的日志中包含变量数据，可使用一个格式字符串作为这个事件的描述消息（`logging.debug`、`logging.info`等函数的第一个参数），然后将变量数据作为第二个参数`*args`的值进行传递，如下:

  ```
  logging.warning('%s is %d years old.', 'Tom', 10)
  # WARNING:root:Tom is 10 years old.
  ```

   

其实，logging所提供的模块级别的日志记录函数也是对logging日志系统相关类的封装而已。这里其实创建了一个 名为`root`的日志器组件，是一个默认的、单例的 logger 组件。接下来我们看一下第二种。



### Logging 组件

logging模块就是通过下面这些组件来完成日志处理的，上面所使用的logging模块级别的函数也是通过这些组件对应的类来实现的。

在介绍logging模块的日志处理流程之前，我们先来介绍下logging模块的四大组件：

| 组件名称 | 对应类名  | 功能描述                                                     |
| -------- | --------- | ------------------------------------------------------------ |
| 日志器   | Logger    | 提供了应用程序可一直使用的接口                               |
| 处理器   | Handler   | 将logger创建的日志记录发送到合适的目的输出                   |
| 过滤器   | Filter    | 提供了更细粒度的控制工具来决定输出哪条日志记录，丢弃哪条日志记录 |
| 格式器   | Formatter | 决定日志记录的最终输出格式                                   |

它们之间的合作关系如下：

- 日志器（logger）需要通过处理器（handler）将日志信息输出到目标位置，如：文件、sys.stdout、网络等；
- 不同的处理器（handler）可以将日志输出到不同的位置；
- 日志器（logger）可以设置多个处理器（handler）将同一条日志记录输出到不同的位置；
- 每个处理器（handler）都可以设置自己的过滤器（filter）实现日志过滤，从而只保留感兴趣的日志；
- 每个处理器（handler）都可以设置自己的格式器（formatter）实现同一条日志以不同的格式输出到不同的地方。

简单点说就是：**日志器（logger）是入口，真正干活儿的是处理器（handler），处理器（handler）还可以通过过滤器（filter）和格式器（formatter）对要输出的日志内容做过滤和格式化等处理操作。**



### 日志流处理简要流程

1. 创建一个logger

2. 设置下logger的日志的等级

3. 创建合适的Handler(FileHandler要有路径)

4. 设置下每个Handler的日志等级。

5. 创建下日志的格式Formater

6. 向Handler中添加上面创建的Formater

7. 将上面创建的Handler注册到logger中

8. 打印输出logger.debug\logger.info\logger.warning\logger.error\logger.critical

> 为什么会有两个`setLevel()`方法？logger 的级别决定了消息是否要传递给处理器。每个handler的级别决定了消息是否要分发到指定目标

从“简单使用”一节中我们了解到了`logging.debug()`、`logging.info()`等函数分别用以记录不同级别的日志信息 ，`logging.basicConfig()`用默认日志格式（Formatter）为日志系统建立一个默认的流处理器（StreamHandler），设置基础配置（如日志级别等）并加到 root logger 中。

接下来就按照上面的流程来介绍高级的用法，来满足各种需求。



### logger

#### 日志器的获取

如何获取一个Logger对象呢？

1. 通过Logger类的实例化方法创建一个Logger类的实例，
2. 但是我们通常都是用第二种方式：`logging.getLogger('name')`方法。

`logging.getLogger('name')`方法有一个可选参数name，该参数表示将要返回的日志器的名称标识， 默认为`root`。若以相同的name参数值多次调用`getLogger()`方法，将会返回指向**同一个logger对象的引用**。



#### 日志器的继承

- logger的名称是以`.`分割的层级结构，每个`.`后面的logger都是`.`前面的logger的children，例如，有一个名称为 foo 的logger，其它名称分别为 `foo.bar`, `foo.bar.baz` 和 `foo.bam`都是 foo 的后代。
- logger有一个"有效等级（effective level）"的概念。如果一个logger上没有被明确设置一个level，那么该logger就使用它parent的level。如果它的parent也没有明确设置level，则继续向上查找parent的parent的有效level，依次类推，直到找到个一个明确设置了level的祖先为止。需要说明的是，root logger总是会有一个明确的level设置（默认为 WARNING）。当决定是否去处理一个已发生的事件时，**logger的有效等级将会被用来决定是否将该事件传递给该logger的handlers进行处理**。
- child loggers在完成对日志消息的处理后，默认会将日志消息传递给与它们的祖先loggers相关的handlers。因此，我们不必为一个应用程序中所使用的所有loggers定义和配置handlers，只需要为一个顶层的logger配置handlers，然后按照需要创建child loggers就可足够了。我们也可以通过将一个logger的propagate属性设置为False来关闭这种传递机制。



### handler

Handler对象的作用是（基于日志消息的level）将消息分发到handler指定的位置（文件、网络、邮件等）。Logger对象可以通过addHandler()方法为自己添加0个或者更多个handler对象。比如，一个应用程序可能想要实现以下几个日志需求：

- 1）把所有日志都发送到一个日志文件中；
- 2）把所有严重级别大于等于error的日志发送到stdout（标准输出）；
- 3）把所有严重级别为critical的日志发送到一个email邮件地址。这种场景就需要3个不同的handlers，每个handler复杂发送一个特定严重级别的日志到一个特定的位置。

```
fh = logging.FileHandler("jizx_log.txt",encoding="utf-8") # 创建一个文件 handler 用于注册到 logger 中
Handler.setLevel(lel):指定被处理的信息级别，低于lel级别的信息将被忽略
Handler.setFormatter()：给这个handler选择一个格式
Handler.addFilter(filt)、Handler.removeFilter(filt)：新增或删除一个filter对象
```



应用程序代码不应该直接实例化和使用Handler实例。因为Handler是一个基类，它只定义了素有handlers都应该有的接口，同时提供了一些子类可以直接使用或覆盖的默认行为。下面是一些常用的Handler：

| Handler                                                      | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| logging.StreamHandler([strm])                                | 将日志消息发送到输出到Stream，如std.out, std.err或任何file-like对象。默认是sys.stderr |
| logging.FileHandler(filename[,mode])                         | 将日志消息发送到磁盘文件，默认`mode`是`a`,文件大小会无限增长。还可是`w` |
| logging.handlers.RotatingFileHandler( filename[, mode[, maxBytes[, backupCount]]]) | 将日志消息发送到磁盘文件，并支持日志文件按大小切割。当文件达到一定大小之后，它会自动将当前日志文件改名，然后创建 一个新的同名日志文件继续输出。maxBytes用于指定日志文件的最大文件大小。如果maxBytes为0，意味着日志文件可以无限大，这时上面描述的重命名过程就不会发生。<br/>backupCount用于指定保留的备份文件的个数。比如，如果指定为2，当上面描述的重命名过程发生时，原有的chat.log.2并不会被更名，而是被删除。 |
| logging.hanlders.TimedRotatingFileHandler( filename [,when [,interval [,backupCount]]]) | 将日志消息发送到磁盘文件，并支持日志文件按时间切割。interval是时间间隔。when参数是一个字符串。表示时间间隔的单位，不区分大小写。它有以下取值：S 秒、M 分、H 小时、D 天、W 每星期（interval==0时代表星期一）、midnight 每天凌晨 |
| logging.handlers.HTTPHandler                                 | 将日志消息以GET或POST的方式发送给一个HTTP服务器              |
| logging.handlers.SMTPHandler                                 | 将日志消息发送给一个指定的email地址                          |
| logging.NullHandler                                          | 该Handler实例会忽略error messages，通常被想使用logging的library开发者使用来避免'No handlers could be found for logger XXX'信息的出现。 |



### formater

Formater对象用于配置日志信息的最终顺序、结构和内容。与logging.Handler基类不同的是，应用代码可以直接实例化Formatter类。另外，如果你的应用程序需要一些特殊的处理行为，也可以实现一个Formatter的子类来完成。

Formatter类的构造方法定义如下：

```
  logging.Formatter.__init__(fmt=None, datefmt=None, style='%')
```

该构造方法接收3个可选参数：

- fmt：指定消息格式化字符串，如果不指定该参数则默认使用message的原始值
- datefmt：指定日期格式字符串，如果不指定该参数则默认使用"%Y-%m-%d %H:%M:%S"
- style：Python 3.2新增的参数，可取值为 '%', '{'和 '$'，如果不指定该参数则默认使用'%'

**一般直接用`logging.Formatter(fmt, datefmt)`**



#### format格式字符串说明

| 字段/属性名称   | 使用格式            | 描述                                                         |
| --------------- | ------------------- | ------------------------------------------------------------ |
| asctime         | %(asctime)s         | 将日志的时间构造成可读的形式，默认情况下是‘2016-02-08 12:00:00,123’精确到毫秒 |
| name            | %(name)s            | 所使用的日志器名称，默认是'root'，因为默认使用的是 rootLogger |
| filename        | %(filename)s        | 调用日志输出函数的模块的文件名； pathname的文件名部分，包含文件后缀 |
| funcName        | %(funcName)s        | 由哪个function发出的log， 调用日志输出函数的函数名           |
| levelname       | %(levelname)s       | 日志的最终等级（被filter修改后的）                           |
| message         | %(message)s         | 日志信息， 日志记录的文本内容                                |
| lineno          | %(lineno)d          | 当前日志的行号， 调用日志输出函数的语句所在的代码行          |
| levelno         | %(levelno)s         | 该日志记录的数字形式的日志级别（10, 20, 30, 40, 50）         |
| pathname        | %(pathname)s        | 完整路径 ，调用日志输出函数的模块的完整路径名，可能没有      |
| process         | %(process)s         | 当前进程， 进程ID。可能没有                                  |
| processName     | %(processName)s     | 进程名称，Python 3.1新增                                     |
| thread          | %(thread)s          | 当前线程， 线程ID。可能没有                                  |
| threadName      | %(thread)s          | 线程名称                                                     |
| module          | %(module)s          | 调用日志输出函数的模块名， filename的名称部分，不包含后缀即不包含文件后缀的文件名 |
| created         | %(created)f         | 当前时间，用UNIX标准的表示时间的浮点数表示； 日志事件发生的时间--时间戳，就是当时调用time.time()函数返回的值 |
| relativeCreated | %(relativeCreated)d | 输出日志信息时的，自Logger创建以 来的毫秒数； 日志事件发生的时间相对于logging模块加载时间的相对毫秒数 |
| msecs           | %(msecs)d           | 日志事件发生事件的毫秒部分。logging.basicConfig()中用了参数datefmt，将会去掉asctime中产生的毫秒部分，可以用这个加上 |



### Filter类（暂时了解）

Filter可以被Handler和Logger用来做比level更细粒度的、更复杂的过滤功能。Filter是一个过滤器基类，它只允许某个logger层级下的日志事件通过过滤。该类定义如下：

```
  class logging.Filter(name='')
      filter(record)
```

比如，一个filter实例化时传递的name参数值为'A.B'，那么该filter实例将只允许名称为类似如下规则的loggers产生的日志记录通过过滤：'A.B'，'A.B,C'，'A.B.C.D'，'A.B.D'，而名称为'A.BB', 'B.A.B'的loggers产生的日志则会被过滤掉。如果name的值为空字符串，则允许所有的日志事件通过过滤。

filter方法用于具体控制传递的record记录是否能通过过滤，如果该方法返回值为0表示不能通过过滤，返回值为非0表示可以通过过滤。



### 综合案例

**需求：**

输出log到控制台，并将日志写入log文件，保存2种类型的log：

- all.log 保存debug, info, warning, critical 信息

- error.log则只保存error信息，同时按照时间自动分割日志文件

```python
import logging
from logging import handlers


class Logger(object):
    level_relations = {
        'debug': logging.DEBUG,
        'info': logging.INFO,
        'warning': logging.WARNING,
        'error': logging.ERROR,
        'crit': logging.CRITICAL
    }  # 日志级别关系映射

    def __init__(self, filename, level='info', when='D', backCount=3, fmt='%(asctime)s - %(pathname)s[line:%(lineno)d] - %(levelname)s: %(message)s'):
        self.logger = logging.getLogger(filename)
        self.logger.setLevel(self.level_relations.get(level))  # 设置日志级别
        format_str = logging.Formatter(fmt)  # 设置日志格式

        out_screen = logging.StreamHandler()  # 往屏幕上输出
        out_screen.setFormatter(format_str)  # 设置屏幕上显示的格式

        out_file = handlers.TimedRotatingFileHandler(filename=filename, when=when, backupCount=backCount, encoding='utf-8')  # 往文件里写入，指定间隔时间自动生成文件的处理器
        out_file.setFormatter(format_str)  # 设置文件里写入的格式
        # interval是时间间隔，backupCount是备份文件的个数，如果超过这个个数，就会自动删除，when是间隔的时间单位，单位有以下几种：
        # S 秒、M 分、H 小时、、D 天、、W 每星期（interval==0时代表星期一）、midnight 每天凌晨

        self.logger.addHandler(out_screen)  # 把对象加到logger里
        self.logger.addHandler(out_file)

    def debug(self, msg):
        self.logger.debug(msg)

    def info(self, msg):
        self.logger.info(msg)

    def warning(self, msg):
        self.logger.warning(msg)

    def error(self, msg):
        self.logger.error(msg)

    def critical(self, msg):
        self.logger.critical(msg)


if __name__ == '__main__':
    log = Logger('all.log',level='debug')
    log.logger.debug('debug')
    log.logger.info('info')
    log.logger.warning(u'警告')
    log.logger.error(u'报错')
    log.logger.critical(u'严重')
    error_log= Logger('error.log', level='error')
    error_log.logger.error('error')
    
# 这种方法虽然调用简单，但是日志中“lineno”字段的信息就只会对应的到上面的 logger 类中的函数，而不是真正打印信息的行数(后期研究一下如何解决)
if __name__ == '__main__':
    all_log = Logger('all.log', level='debug')
    all_log.debug('debug')
    all_log.info('info')
    all_log.warning(u'警告')
    all_log.error(u'报错')
    all_log.critical(u'严重')
    error_log=Logger('error.log', level='error')
    error_log.error('error')
    error_log.critical(u'严重')

```





参考:https://www.cnblogs.com/Nicholas0707/p/9021672.html

https://www.cnblogs.com/nancyzhu/p/8551506.html 综合案例