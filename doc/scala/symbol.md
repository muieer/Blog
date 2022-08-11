# Scala 难懂符号详解
## 1. 可变参数序列 _*
示例代码
```scala
def echo(args: String*) = 
    for (arg <- args) println(arg)

val arr = Array("What's", "up", "doc?")
echo(arr: _*)
```
此符号告诉编译器将该参数看作可变参数序列
## 2. 函数表达式 =>
左边是参数列表，右边是表达式
## 3. 生成器表达式 <-
左箭头符号`<-`用于从集合或`Option`等容器中提取值
## 4. 参数子类型 <:
`class Zoo[a <: Animal]`的意思是参数`a`是类`Animal`的子类型及其本身，所以`Zoo[Cat]`，`Zoo[Dog]`是正确的，
而`Zoo[Children]`是错误的
## 5. 参数父类型 >:
`class Zoo[a >: Cat]`的意思是参数`a`是类`Cat`的父类型及其本身，所以`Zoo[Cat]`，`Zoo[Animal]`是正确的，
而`Zoo[Dog]`是错误的
## 6. 协变 +A
`List[+A]`表示传入的实际类型应该是泛型A及其子类
## 7. 逆变 -A
`List[-A]`表示传入的实际类型应该是泛型A及其父类


