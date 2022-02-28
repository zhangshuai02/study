# with 语句使用
* 文件读写模式
  * r -- 只读 文件不存在 报错
  * r+ -- 可读可写 文件不存在报错 覆盖
  * w -- 只写 文件不存在则创建 覆盖
  * w+ -- 可读可写 文件不存在则创建 覆盖
  * a -- 只写 文件不存在则创建 追加
  * a+ -- 可读可写 文件不存在则创建 
  * b -- 二进制文件
## read读方法 
  * read 读取整个文件 可调用read(size)方法，限制读取内容字节大小
  * readline 每次只读取一行，比readlines()慢
  * readlines 自动将文件内容分析成一个行的列表，该列表可以由 Python 的 for ... in ... 结构进行处理
## write写方法
  * write 将字符串写入到文件中
  * writelines 针对列表的操作，它接收一个字符串列表作为参数，将他们写入到文件中，换行符不会自动的加入，内容中需要加入换行符
### open方式操作数据
```python
# 写入方式创建
data = open('1.txt', 'w')
# 内容
data.write("hello, world")
# 关闭
data.close()
# ⽂件使⽤完后必须关闭，因为⽂件对象会占⽤操作系统的资源，并且操作系统同⼀时间能打开的 ⽂件数量也是有限的
```
* 由于⽂件读写时都有可能产⽣IOError，⼀旦出错，后⾯的f.close()就不会调⽤。 为了保证⽆论是否出错都能正确地关闭⽂件，我们可以使⽤try ... finally来解决
```python
try:
# 写入方式创建
    data = open('1.txt', 'r')
    # 内容
    data.write("hello, world")
except IOError as e:
    print("文件读写异常", e)
finally:
    # 关闭
    data.close()
```
* 虽然代码运⾏良好,但是缺点就是代码过于冗⻓,并且需要添加try-except-finally语句 
* Python提供了 with 语句写法，既简单⼜安全，并且 with 语句执⾏完成以后⾃动调⽤关闭⽂件操作，即使出现异常也会⾃动调⽤关闭⽂件操作
```python
with open("1.txt", 'r+') as f :
    # 写入文件内容
    f.write("hello, world")
```
# 上下文管理
## 概述
⼀个类只要实现了 __enter__()和__exit__() 这个两个⽅法，通过该类创建的对象我们就称之为上下⽂管理器(with open 的open方法就是一个上下文管理器)
## 优点
* 提高代码的复用率、优雅度、可读性
* 常用于数据库操作(连接->查询->关闭)
```python
class isOpen(object):
    def __init__(self, data):
      # 初始化参数 ,可要可不要
      self.__data = data
    
    def __enter__(self):
        print("上文方法")
        return self
        # 返回这个实例

    def __exit__(self, exc_type, exc_val, exc_tb):
        print("下文方法")
        # __exit__ 函数 必须包含三个参数， 主逻辑无异常时，都为None
        # exc_type：异常类型
        # exc_val：异常值
        # exc_tb：异常的错误栈信息

    def isRead(self):
        print("内容:")

with isOpen("传入的data") as f:
    f.isRead()
# 上文方法
# 内容:
# 下文方法
```
* 大量的异常处理影响代码可读性，上下文管理器，with 将异常的处理隐藏起来 ↓
```python
# 将1/0 这个一定会抛出异常的代码写在 operate 里
class Resource():
    def __enter__(self):
        print('===connect to resource===')
        return self
        # 返回实例

    def __exit__(self, exc_type, exc_val, exc_tb):
        print('===close resource connection===')
        return True
        # 没有return 就默认为 return False

    def operate(self):
        1/0

with Resource() as res:
    res.operate()
# 没有报错
# 异常可以在__exit__ 进行捕获并由你自己决定如何处理，是抛出呢还是在这里就解决了。
# 在__exit__ 里返回 True（没有return 就默认为 return False），就相当于告诉 Python解释器，这个异常我们已经捕获了，不需要再往外抛
```
## contextlib 装饰器
* 将函数对象变成一个上下文管理器
* @contextlib.contextmanager
```python
import contextlib
@contextlib.contextmanager
def isOpen(fileName):
    # 上文方法 __enter__方法
    data = open(fileName, "r")
    # 无法处理异常，必须主动添加
    try:
        yield data
        # 在被装饰函数里，必须是一个生成器 yield
    except Exception as e:
        print("读写异常", e)
    finally:
        # 下文方法 __exit__方法
        data.close()


with isOpen(r"1.txt") as f:
        for line in f:
            0/1 # 模拟异常
            print(line)
# hello world
```
* 在被装饰函数里，必须是一个生成器（带有yield），而yield之前的代码，就相当于__enter__里的内容。yield 之后的代码，就相当于__exit__ 里的内容。