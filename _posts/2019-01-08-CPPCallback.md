---
title: 回调函数
date: 2019-01-08 23:23:07
categories:
- C/C++
tags:
- function
---

回调函数就是一个通过函数指针调用的函数。如果你把函数的指针（地址）作为参数传递给另一个函数，当这个指针被用来调用其所指向的函数时，我们就说这是回调函数。回调函数不是由该函数的实现方直接调用，而是在特定的事件或条件发生时由另外的一方调用的，用于对该事件或条件进行响应。
<!--more-->
## 函数指针

函数指针也是一种指针，只是它指向的不是整型，字符型而是函数。在C中，每个函数在编译后都是存储在内存中，并且每个函数都有一个入口地址，根据这个地址，我们便可以访问并使用这个函数。函数指针就是通过指向这个函数的入口，从而调用这个函数。

### 函数指针的定义

函数指针虽然也是指针，但它的定义方式却和其他指针看上去很不一样，我们来看看它是如何定义的：

```c
/* 方法1 */
void (*p_func)(int, int, float) = NULL;

/* 方法2 */
typedef void (*tp_func)(int, int, float);
tp_func p_func = NULL;
```
这两种方式都是定义了一个指向返回值为 void 类型，参数为 (int, int, float) 的函数指针。第二种方法是为了让函数指针更容易理解，尤其是在复杂的环境下；而对于一般的函数指针，直接用第一种方法就行了。

### 函数指针的赋值

在定义完函数指针后，我们就需要给它赋值了我们有两种方式对函数指针进行赋值：
```c
void (*p_func)(int, int, float) = NULL;
p_func = &func1;
p_func = func2;
```
上面两种方法都是合法的，对于第二种方法，编译器会隐式地将 func_2 由 void ()(int, int, float) 类型转换成 void (*)(int, int, float) 类型，因此，这两种方法都行。

### 使用函数指针调用函数

因为函数指针也是指针，因此可以使用常规的带 * 的方法来调用函数。和函数指针的赋值一样，我们也可以使用两种方法：
```c
/* 方法1 */
int val1 = p_func(1,2,3.0);

/* 方法2 */
int val2 = (*p_func)(1,2,3.0);
```
方法1和我们平时直接调用函数是一样的，方法2则是用了 * 对函数指针取值，从而实现对函数的调用。

### 将函数指针作为参数传给函数

函数指针和普通指针一样，我们可以将它作为函数的参数传递给函数，下面我们看看如何实现函数指针的传参：
```c
/* func3 将函数指针 p_func 作为其形参 */
void func3(int a, int b, float c, void (*p_func)(int, int, float))
{
    (*p_func)(a, b, c);
}

/* func4 调用函数func3 */
void func4()
{
    func3(1, 2, 3.0, func_1);
    /* 或者 func3(1, 2, 3.0, &func_1); */
}
```

### 函数指针作为函数返回类型

有了上面的基础，要写出返回类型为函数指针的函数应该不难了，下面这个例子就是返回类型为函数指针的函数：
```c
void (* func5(int, int, float ))(int, int)
{
    ...
}
```
在这里， func5 以 (int, int, float) 为参数，其返回类型为 void (*)(int, int) 。

## 回调函数

下面是一个四则运算的简单回调函数例子：
```c
#include <stdio.h>
#include <stdlib.h>

/****************************************
 * 函数指针结构体
 ***************************************/
typedef struct _OP {
    float (*p_add)(float, float); 
    float (*p_sub)(float, float); 
    float (*p_mul)(float, float); 
    float (*p_div)(float, float); 
} OP; 

/****************************************
 * 加减乘除函数
 ***************************************/
float ADD(float a, float b) 
{
    return a + b;
}

float SUB(float a, float b) 
{
    return a - b;
}

float MUL(float a, float b) 
{
    return a * b;
}

float DIV(float a, float b) 
{
    return a / b;
}

/****************************************
 * 初始化函数指针
 ***************************************/
void init_op(OP *op)
{
    op->p_add = ADD;
    op->p_sub = SUB;
    op->p_mul = &MUL;
    op->p_div = &DIV;
}

/****************************************
 * 库函数
 ***************************************/
float add_sub_mul_div(float a, float b, float (*op_func)(float, float))
{
    return (*op_func)(a, b);
}

int main(int argc, char *argv[]) 
{
    OP *op = (OP *)malloc(sizeof(OP)); 
    init_op(op);
    
    /* 直接使用函数指针调用函数 */ 
    printf("ADD = %f, SUB = %f, MUL = %f, DIV = %f\n", (op->p_add)(1.3, 2.2), (*op->p_sub)(1.3, 2.2), 
            (op->p_mul)(1.3, 2.2), (*op->p_div)(1.3, 2.2));
     
    /* 调用回调函数 */ 
    printf("ADD = %f, SUB = %f, MUL = %f, DIV = %f\n", 
            add_sub_mul_div(1.3, 2.2, ADD), 
            add_sub_mul_div(1.3, 2.2, SUB), 
            add_sub_mul_div(1.3, 2.2, MUL), 
            add_sub_mul_div(1.3, 2.2, DIV));

    return 0; 
}
```