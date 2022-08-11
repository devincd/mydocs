## Golang基本概念（Packages,Variables and Functions）

### 命名返回值
Golang 返回值可以命名，如果是这样，则将它们视为在函数顶部定义的变量。
```go
package main

import "fmt"

func split(sum int) (x, y int) {
	x = sum * 4 / 9
	y = sum - x
	return
}

func main() {
	fmt.Println(split(17))
}
```
> A `return` statement without arguments returns the named return values. This is known as a "naked" return.
> 
> `Naked` return statements should be used only in short functions, as with the example shown here.
> They can harm readability in longer functions.

### Go语言中defer和return执行顺序解析
首先要明白`return`是非原子性的，需要两步，首先要将返回值放到一个临时变量中（为返回值赋值），然后将返回值返回到被调用处。
而`defer`函数恰在`return`这两个操作之间执行。

真正的执行顺序是：

先为返回值赋值，即将返回值放到一个临时变量中，然后执行`defer`，然后`return`到函数被调用处。

如果所在函数为命名返回值函数，`return`第一步，返回值就是命名返回值变量，如果恰好`defer`函数中修改了该返回值，那么最终返回值是更新后的。
但是如果所在函数为无名返回值函数，那么`return`第一步先把返回值放到一个临时变量中，`defer`函数无法获取到这个临时变量地址，所以无论`defer`函数做任何操作，
都不会对最终返回值造成任何变动。

#### 例子：
测试用例1：无名返回值（即函数返回值为没有命名的返回值）
```go
package main

import "fmt"

func main() {
	fmt.Println("return: ", Demo1())
}

func Demo1() int {
	var i int

	defer func() {
		i++
		fmt.Println("defer2: ", i)
	}()

	defer func() {
		i++
		fmt.Println("defer1: ", i)
	}()

	return i
}
```

执行结果：
```shell
defer1:  1
defer2:  2
return:  0
```

结果分析：
实际上 return 执行了两步操作。因为返回值没有命名，所以 return 之前首先默认创建了一个临时零值变量（假设为 s）作为返回值，
然后将 i 赋值给 s，此时 s 的值为0， 后续的操作是针对 i 进行的，所以不会影响 s。

相当于以下流程：
```go
var i int
s := i
return s
```

测试用例2：命名返回值（函数返回值为已经命名的返回值）
```go
package main

import "fmt"

func main() {
	fmt.Println("return: ", Demo2())
}

func Demo2() (i int) {
	defer func() {
		i++
		fmt.Println("defer2: ", i)
	}()

	defer func() {
		i++
		fmt.Println("defer1: ", i)
	}()

	return
}
```

执行结果：
```shell
defer1:  1
defer2:  2
return:  2
```

结果分析：
因为返回值已经提前定义了，不会产生临时零值变量，返回值就是提前定义的变量，后续所有操作也都是基于已经定义的变量，任何对于返回值变量的修改都会影响到返回值本身。

### 常量
```go
package main

import "fmt"

const Pi = 3.14

func main() {
	const World = "世界"
	fmt.Println("Hello", World)
	fmt.Println("Happy", Pi, "Day")

	const Truth = true
	fmt.Println("Go rules?", Truth)
}

// Constants are declared like variables, but with the const keyword.

// Constants can be character, string, boolean, or numeric values.

// Constants cannot be declared using the := syntax.

```