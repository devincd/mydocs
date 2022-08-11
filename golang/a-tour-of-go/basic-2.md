## Golang基本概念（More types: structs, slices, and maps.）

## 指针（Pointers）
Go has pointers. A pointer holds the memory address of a value.

The type `*T` is a pointer to a `T` value. Its zero value is nil.

```
var p *int
```
The `&` operator generates a pointer to its operand.

```go
i := 42
p = &i
```

The `*` operator denotes the pointer's underlying value.
```
fmt.Println(*p) // read i through the pointer p
*p = 21         // set i through the pointer p
```
This is known as "dereferencing" or "indirecting".

Unlike C, Go has no pointer arithmetic.

示例：
```go
package main

import "fmt"

func main() {
	i, j := 42, 2701

	p := &i         // point to i
	fmt.Println(*p) // read i through the pointer
	*p = 21         // set i through the pointer
	fmt.Println(i)  // see the new value of i

	p = &j         // point to j
	*p = *p / 37   // divide j through the pointer
	fmt.Println(j) // see the new value of j
}

/* 
OUTPUT:
42
21
73
*/
```

指针在 Go 语言中华可以被拆分为两个核心概念：
- 类型指针：允许对这个指针类型的数据进行修改，传递数据可以直接使用指针，而无需拷贝数据，类型指针不能进行偏移和运算。
- 切片，由指向起始元素的原始指针，元素数量和容量组成。


要搞懂指针，需要知道几个概念：指针地址，指针类型和指针取值，下面将展开详细说明

### 认识指针地址和指针类型
一个指针变量可以指向任何一个值的内存地址

## 参考文献
1. http://c.biancheng.net/view/21.html