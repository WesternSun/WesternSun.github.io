---
title: Golang 继承
date: 2018-12-15 11:52:36
categories:
- Golang
tags:
- class
---

# Golang 继承


Go语言的interface概念相对于C++中的基类，通过interface来实现多态功能。

在C++中，当需要实现多态功能时，步骤是首先定义一个基类，该基类使用虚函数或者纯虚函数抽象了所有子类会用到的共同的最基本的成员函数，之后子类继承该基类，然后每个派生类自定义自己的虚函数实现。最后在使用基类指针或者引用的地方传入派生类，程序根据具体传入的对象来调用对象的函数。
<!--more-->
在Go中，定义一个interface类型，该类型说明了它有哪些方法，这就完成了类似C++中的基类定义，然后在其他的函数中，将该interface类型作为函数的形参，任意一个实现了interface类型的实参都能作为该interface的实例对象。interface类型和作为interface类型的实参对象之间就相当于存在继承关系，或者说叫实现接口（Java说法），但这种继承关系（实现接口）是隐式的（自动的），也即我们无需主动说明（显式implements实现）该实参类型是某interface类型的派生类，代码如下：

```go
type base interface { //类似基类定义
    virtualfunc() int //类似虚函数抽象定义
}
 
type der1 int //类似派生类1定义
 
func (der1) virtualfunc() int { //类似派生类1虚函数具体实现
    fmt.Printf("I'm der1\n")
    return 1
}
 
type der2 struct { //类似派生类2定义
    //nothing
}
 
func (der2) virtualfunc() int { //类似派生类2虚函数具体实现
    fmt.Printf("I'm der2\n")
    return 2
}
 
func somefunc(b base) { //作为某个函数的形参
    b.virtualfunc()
}
```

上述代码中base是interface类型，b作为somefunc( )函数的形参，因为base接口类型要求实现virtualfunc( )函数，而der1和der2都实现了该函数，因为der1和der2自动是base的派生类，在somefunc( )函数中，要求base类型的实参时，可以用der1或者der2的实例对象传入，这就实现了不同类型不同行为，也即多态。这是Go实现面向对象特性的一种特殊方法，并不是通过类和继承来完成的，Go也没有继承这种功能。

上面的代码并没有说明interface的全部特性，还有一些说明如下：

实现某个接口的类型（如上面的der2）可以有其他的方法。这正如派生类还可以额外增加基类并没有的成员函数一样，但增加的函数不是接口中的部分。
一个类型可以实现多个接口。这类似于C++的多继承，一个派生类可以当做多种基类来使用。
从somefunc( )函数中的形实参结合可以看到，我们能定义一个interface类型的变量，并用一个“派生类”去初始化或者赋值，代码如下：

```go
func main() {
    var baseobj base
 
    var d1 der1
    baseobj = d1
    somefunc(baseobj)
 
    var d2 der2
    baseobj = d2
    somefunc(baseobj)
}
```
上面代码中，第2行是定义了一个默认初始化的interface base类型对象，第5行和第9行代码则是用两个“派生类”去赋值interface base类型对象。运行结果如下：

```shell
1
2
I'm der1
I'm der2
```

Go中没有继承的功能，它是通过接口来实现类似功能，Go中还有一种叫做组合的概念，如下：

```go
package main
 
import (
    "fmt"
)
 
type Base struct {
    // nothing
}
 
func (b *Base) ShowA() {
    fmt.Println("showA")
    b.ShowB()
}
func (b *Base) ShowB() {
    fmt.Println("showB")
}
 
type Derived struct {
    Base
}
 
func (d *Derived) ShowB() {
    fmt.Println("Derived showB")
}
 
func main() {
    d := Derived{}
    d.ShowA()
}
```
上述代码执行结果不会输出“Derived showB”，因为Go中没有继承的概念，只有组合，上面的Derived包含了Base，自动的含有了Base的方法，因为其不是继承，所以不会根据具体传入的对象而执行对应的方法。

 

下面的的代码又说明了type name和type struct的区别：

```go
package main
 
import (
    "fmt"
)
 
type Mutex struct {
    // nothing
}
 
func (m *Mutex) Lock() {
    fmt.Println("mutex lock")
}
 
func (m *Mutex) Unlock() {
    fmt.Println("mutex unlock")
}
 
type newMutex Mutex
 
type structMutex struct {
    Mutex
}
 
func main() {
    m1 := Mutex{}
    m1.Lock()
 
    // n1 := newMutex{}  
    // n1.Lock()      没有Lock()方法
 
    x1 := structMutex{}
    x1.Lock()
 
}
```
上面的代码中n1不能执行Lock( )函数，因为Golang不支持隐式类型转换，虽然newMutex就是Mutex，但语法上它们是两种类型，因此newMutex不能执行Lock( )方法。