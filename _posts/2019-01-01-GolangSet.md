---
title: 数组与切片
date: 2019-01-01 22:17:52
categories:
- golang
tags:
- golang
---

## 数组与切片

### 概念
Go 语言中的数组是一种 值类型（不像 C/C++ 中是指向首元素的指针）
```go
var arr1 [5]int

// 也可以通过 new() 来创建
var arr2 = new([5]int) // arr2 类型是 *[5]int
```

当把arr1一个数组赋值给另一个时，需要在做一次数组内存的拷贝操作。如果你想修改原数组，那么 arr1 必须通过&操作符以地址方式来传递。
 <!--more--> 
把一个大数组传递给函数会消耗很多内存。有两种方法可以避免这种现象：

- 传递数组的指针
- 使用数组的切片

编译时需要知道数组长度以便分配内存，所以数组长度必须是一个常量表达式，并且必须是一个非负整数，数组长度最大为 2Gb。数组长度也是数组类型的一部分，所以[5]int和[10]int是属于不同类型的。

声明数组时所有的元素都会被自动初始化为默认值。

### 数组常量

如果数组值已经提前知道了，那么可以通过 数组常量 的方法来初始化数组，而不用依次使用 []= 方法

```go
package main
import "fmt"

func main() {
    // var arrAge = [5]int{18, 20, 15, 22, 16}
    // var arrLazy = [...]int{5, 6, 7, 8, 22}
    // var arrLazy = []int{5, 6, 7, 8, 22}
    var arrKeyValue = [5]string{3: "Chris", 4: "Ron"}
    // var arrKeyValue = []string{3: "Chris", 4: "Ron"}

    for i:=0; i < len(arrKeyValue); i++ {
        fmt.Printf("Person at %d is %s\n", i, arrKeyValue[i])
    }
}
```

可以取任意数组常量的地址来作为指向新实例的指针。

```go
package main
import "fmt"

func fp(a *[3]int) { fmt.Println(a) }

func main() {
    for i := 0; i < 3; i++ {
        fp(&[3]int{i, i * i, i * i * i})
    }
}
```

### 多维数组

数组通常是一维的，但是可以用来组装成多维数组，例如：[3][5]int，[2][2][2]float64。

## 切片

### 概念

切片（slice）类型。这是一种建立在 Go 语言数组类型之上的抽象，是对数组一个连续片段的引用（该数组我们称之为相关数组，通常是匿名的），所以切片是一个引用类型（因此更类似于 C/C++ 中的数组类型，或者 Python 中的 list 类型）。这个片段可以是整个数组，或者是由起始和终止索引标识的一些项的子集。需要注意的是，终止索引标识的项不包括在切片内。

切片在内存中的组织方式实际上是一个有 3 个域的结构体：指向相关数组的指针，切片长度以及切片容量。

![]({{ site.url }}/assets/data/2019-01-01-GolangSlice.png)

声明切片的格式是： var identifier []type（不需要说明长度）。

一个切片在未初始化之前默认为 nil，长度为 0。

切片的初始化格式是：var slice1 []type = arr1[start:end]。

切片也可以用类似数组的方式初始化：
```go
var x1 := []int{2, 3, 5, 7, 11}

var x2 := [3]int{1,2,3}[:]

var arr1 [6]int
var x3 []int = arr1[2:5] // item at index 5 not included!
```

### new() 和 make() 的区别

new(T) 为每个新的类型T分配一片内存，初始化为 0 并且返回类型为*T的内存地址：这种方法 返回一个指向类型为 T，值为 0 的地址的指针，它适用于值类型如数组和结构体，它相当于 &T{}。

make(T) 返回一个类型为 T 的初始值，它只适用于3种内建的引用类型：切片、map 和 channel。

![]({{ site.url }}/assets/data/2019-01-01-GolangNewMake.png)





