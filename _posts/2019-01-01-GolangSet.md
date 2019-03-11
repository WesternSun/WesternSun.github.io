---
title: 数组与切片
date: 2019-01-01 22:17:52
categories:
- Golang
tags:
- golang
---

## 数组

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

### 原理

```go
// slice 数据结构
type slice struct {
	array unsafe.Pointer
	len   int
	cap   int
}

func makeslice(et *_type, len, cap int) unsafe.Pointer {
	mem, overflow := math.MulUintptr(et.size, uintptr(cap))
	if overflow || mem > maxAlloc || len < 0 || len > cap {
		// NOTE: Produce a 'len out of range' error instead of a
		// 'cap out of range' error when someone does make([]T, bignumber).
		// 'cap out of range' is true too, but since the cap is only being
		// supplied implicitly, saying len is clearer.
		// See golang.org/issue/4085.
		mem, overflow := math.MulUintptr(et.size, uintptr(len))
		if overflow || mem > maxAlloc || len < 0 {
			panicmakeslicelen()
		}
		panicmakeslicecap()
	}

	return mallocgc(mem, et, true)
}

func makeslice64(et *_type, len64, cap64 int64) unsafe.Pointer {
	len := int(len64)
	if int64(len) != len64 {
		panicmakeslicelen()
	}

	cap := int(cap64)
	if int64(cap) != cap64 {
		panicmakeslicecap()
	}

	return makeslice(et, len, cap)
}


// growslice handles slice growth during append.
// It is passed the slice element type, the old slice, and the desired new minimum capacity,
// and it returns a new slice with at least that capacity, with the old data
// copied into it.
// The new slice's length is set to the old slice's length,
// NOT to the new requested capacity.
// This is for codegen convenience. The old slice's length is used immediately
// to calculate where to write new values during an append.
// TODO: When the old backend is gone, reconsider this decision.
// The SSA backend might prefer the new length or to return only ptr/cap and save stack space.
func growslice(et *_type, old slice, cap int) slice {
	if raceenabled {
		callerpc := getcallerpc()
		racereadrangepc(old.array, uintptr(old.len*int(et.size)), callerpc, funcPC(growslice))
	}
	if msanenabled {
		msanread(old.array, uintptr(old.len*int(et.size)))
	}

	if cap < old.cap {
		panic(errorString("growslice: cap out of range"))
	}

	if et.size == 0 {
		// append should not create a slice with nil pointer but non-zero len.
		// We assume that append doesn't need to preserve old.array in this case.
		return slice{unsafe.Pointer(&zerobase), old.len, cap}
	}

	newcap := old.cap
	doublecap := newcap + newcap
	if cap > doublecap {
		newcap = cap
	} else {
		if old.len < 1024 {
			newcap = doublecap
		} else {
			// Check 0 < newcap to detect overflow
			// and prevent an infinite loop.
			for 0 < newcap && newcap < cap {
				newcap += newcap / 4
			}
			// Set newcap to the requested cap when
			// the newcap calculation overflowed.
			if newcap <= 0 {
				newcap = cap
			}
		}
	}

	var overflow bool
	var lenmem, newlenmem, capmem uintptr
	// Specialize for common values of et.size.
	// For 1 we don't need any division/multiplication.
	// For sys.PtrSize, compiler will optimize division/multiplication into a shift by a constant.
	// For powers of 2, use a variable shift.
	switch {
	case et.size == 1:
		lenmem = uintptr(old.len)
		newlenmem = uintptr(cap)
		capmem = roundupsize(uintptr(newcap))
		overflow = uintptr(newcap) > maxAlloc
		newcap = int(capmem)
	case et.size == sys.PtrSize:
		lenmem = uintptr(old.len) * sys.PtrSize
		newlenmem = uintptr(cap) * sys.PtrSize
		capmem = roundupsize(uintptr(newcap) * sys.PtrSize)
		overflow = uintptr(newcap) > maxAlloc/sys.PtrSize
		newcap = int(capmem / sys.PtrSize)
	case isPowerOfTwo(et.size):
		var shift uintptr
		if sys.PtrSize == 8 {
			// Mask shift for better code generation.
			shift = uintptr(sys.Ctz64(uint64(et.size))) & 63
		} else {
			shift = uintptr(sys.Ctz32(uint32(et.size))) & 31
		}
		lenmem = uintptr(old.len) << shift
		newlenmem = uintptr(cap) << shift
		capmem = roundupsize(uintptr(newcap) << shift)
		overflow = uintptr(newcap) > (maxAlloc >> shift)
		newcap = int(capmem >> shift)
	default:
		lenmem = uintptr(old.len) * et.size
		newlenmem = uintptr(cap) * et.size
		capmem, overflow = math.MulUintptr(et.size, uintptr(newcap))
		capmem = roundupsize(capmem)
		newcap = int(capmem / et.size)
	}

	// The check of overflow in addition to capmem > maxAlloc is needed
	// to prevent an overflow which can be used to trigger a segfault
	// on 32bit architectures with this example program:
	//
	// type T [1<<27 + 1]int64
	//
	// var d T
	// var s []T
	//
	// func main() {
	//   s = append(s, d, d, d, d)
	//   print(len(s), "\n")
	// }
	if overflow || capmem > maxAlloc {
		panic(errorString("growslice: cap out of range"))
	}

	var p unsafe.Pointer
	if et.kind&kindNoPointers != 0 {
		p = mallocgc(capmem, nil, false)
		// The append() that calls growslice is going to overwrite from old.len to cap (which will be the new length).
		// Only clear the part that will not be overwritten.
		memclrNoHeapPointers(add(p, newlenmem), capmem-newlenmem)
	} else {
		// Note: can't use rawmem (which avoids zeroing of memory), because then GC can scan uninitialized memory.
		p = mallocgc(capmem, et, true)
		if writeBarrier.enabled {
			// Only shade the pointers in old.array since we know the destination slice p
			// only contains nil pointers because it has been cleared during alloc.
			bulkBarrierPreWriteSrcOnly(uintptr(p), uintptr(old.array), lenmem)
		}
	}
	memmove(p, old.array, lenmem)

	return slice{p, old.len, newcap}
}
```
切片提供了一个与指向数组的动态窗口。

```text
A slicing operation creates a new slice by describing a contiguous section of an already-created array or slice:

var array [10]int
slice := array[2:4]
The capacity of the slice is the maximum number of elements that the slice may hold, even after reslicing; it reflects the size of the underlying array.
In this example, the capacity of the slice variable is 8.

(capacity of the underlying array minus the position of the start of the slice in the array)

array : [0 0 0 0 0 0 0 0 0 0]
 array : <---- capacity --->
 slice :    [0 0]
 slice :    <-- capacity --> 8 (10-2)
Go 1.2 adds new syntax to allow a slicing operation to specify the capacity as well as the length.
A second colon introduces the capacity value, which must be less than or equal to the capacity of the source slice or array, adjusted for the origin.

For instance,

slice = array[2:4:6]

array : [0 0 0 0 0 0 0 0 0 0]
 array : <---- capacity --->   10
 slice :    [0 0]
 slice :    <- cap->           4 (6-2)
sets the slice to have the same length as in the earlier example but its capacity is now only 4 elements (6-2).
It is impossible to use this new slice value to access the last two elements of the original array.

The main argument is to give programmers more control over append.

a[i : j : k]
That slice has:

indices starting at 0
length equals to j - i
capacity equals to k - i
The evaluation panics if i <= j <= k <= cap(a) is not true.
 ```

切片作为参数传递时，函数接收到的参数是数组切片的一个复制，虽然两个是不同的变量，但是它们都有一个指向同一个地址空间的array指针，当修改一个数组切片时，另外一个也会改变，所以数组切片看起来是引用传递，其实是值传递。

对切片进行append操作是，如果导致了扩容的情况，那么新的切片中array不再指向原来的数组的首元素的地址，所以对这个切片的元素进行修改，无法影响到原有的数组或使用这个数组的其它切片。

可以用传递切片地址的方式来保证对切片的修改一定会在原有切片上生效。

```go
func main() {
	slice1 := make([]int, 0)
	TestValue(slice1)
	slice2 := make([]int, 0)
	TestAddr(&slice2)
	fmt.Printf("%d\n", slice1)
	fmt.Printf("%d\n", slice2)
}

func TestValue(slice []int) {
	for i := 0; i < 10; i++ {
		slice = append(slice, i)
	}
}

func TestAddr(slice *[]int) {
	for i := 0; i < 10; i++ {
		*slice = append(*slice, i)
	}
}

// 输出：
// []
// [0 1 2 3 4 5 6 7 8 9]
```
### nil和空切片

nil 切片被用在很多标准库和内置函数中，描述一个不存在的切片的时候，就需要用到 nil 切片。比如函数在发生异常的时候，返回的切片就是 nil 切片。nil 切片的指针指向 nil。

空切片一般会用来表示一个空的集合。比如数据库查询，一条结果也没有查到，那么就可以返回一个空切片。

```go
var slice []int // nil切片
slice := make([]int, 0)  // 空切片
slice := []int{} // 空切片
```

空切片和 nil 切片的区别在于，空切片指向的地址不是nil，指向的是一个内存地址，但是它没有分配任何内存空间，即底层元素包含0个元素。

最后需要说明的一点是。不管是使用 nil 切片还是空切片，对其调用内置函数 append，len 和 cap 的效果都是一样的。





