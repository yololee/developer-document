# 数组和切片

## 数组

数组：是同一种数据类型的固定长度的序列

### 数组定义

```go
var 变量名 [数组大小]元素类型

// 定义拥有10个元素的数组a，元素类型是int
var a [10]int

// 定义拥有2个元素的字符串数组
var arr [2]string 
```

### 数组初始化

```go
package main

import "fmt"

var arr0 = [5]int{1, 2, 3}
var arr1 = [5]int{1, 2, 3, 4, 5}
var arr2 = [...]int{1, 2, 3, 4, 5, 6}
var str = [5]string{3: "hello world", 4: "tom"}

func main() {
	a := [3]int{1, 2}           // 未初始化元素值为 0。
	b := [...]int{1, 2, 3, 4}   // 通过初始化值确定数组长度。
	c := [5]int{2: 100, 4: 200} // 使用引号初始化元素。
	d := [...]struct {
		name string
		age  uint8
	}{
		{"user1", 10}, // 可省略元素类型。
		{"user2", 20}, // 别忘了最后一行的逗号。
	}
	fmt.Println(arr0, arr1, arr2, str)
	fmt.Println(a, b, c, d)
}
```

输出结果

```
[1 2 3 0 0] [1 2 3 4 5] [1 2 3 4 5 6] [   hello world tom]
[1 2 0] [1 2 3 4] [0 0 100 0 200] [{user1 10} {user2 20}]
```

### 读取数组

```go
package main

import "fmt"

func main() {

	a := [5]int{1, 2, 3, 4, 5}
	var b [2]string

	b[0] = "hello"
	b[1] = "world"

	i := a[3]
	fmt.Println(i)
	fmt.Println(a)
	fmt.Println(b)
}
```

输出结果

```
4
[1 2 3 4 5]
[hello world]
```

### 遍历数组

```go
package main

import (
	"fmt"
	"strconv"
)

func main() {

	arr1 := [3]string{"hello", "world", "go"}
	for i := 0; i < len(arr1); i++ {
		fmt.Println("第一种方式：第" + strconv.Itoa(i) + "个元素是" + arr1[i])
	}

	for i, s := range arr1 {
		fmt.Println("第二种方式：第" + strconv.Itoa(i) + "个元素是" + s)
	}

}
```

输出结果

```
第一种方式：第0个元素是hello
第一种方式：第1个元素是world
第一种方式：第2个元素是go
第二种方式：第0个元素是hello
第二种方式：第1个元素是world
第二种方式：第2个元素是go
```

### 数组拷贝和传参

```go
package main

import "fmt"

func main() {

	var arr [5]int
	printArr(arr)
	fmt.Println(arr)
	arr2 := [...]int{2, 4, 6, 8, 10}
	printArr2(&arr2)
	fmt.Println(arr2)
}

func printArr2(arr *[5]int) {
	arr[0] = 10
	for i, v := range arr {
		fmt.Println(i, v)
	}
}

func printArr(arr [5]int) {
	arr[0] = 10
	for i, v := range arr {
		fmt.Println(i, v)
	}
}
```

输出结果

> 数组是值类型，赋值和传参会复制整个数组，而不是指针。因此改变副本的值，不会改变本身的值
>
> 指针数组 [n]*T，数组指针 *[n]T

```
0 10
1 0
2 0
3 0
4 0
[0 0 0 0 0]
0 10
1 4
2 6
3 8
4 10
[10 4 6 8 10]
```

## 切片

数组的长度是固定的，在实际应用中非常不方便，因此go语言提供了slice机制，我们一般翻译成切片，可以将切片当成动态数组用，动态数组指的是数组的长度可以动态调整。

切片底层依赖数组存储数据，切片本身是不存储数据，如果底层数组无法存储更多的数据，就会自动新申请一个更大存储空间的数组，将老的数组中的数据拷贝到新的数组，这样我们看起来slice就像动态数组一样可以存储任意数量的数据。

从底层存储角度看，切片（slice）就是数组的引用

### 定义切片

```go
package main

import "fmt"

func main() {
	//1.声明切片
	var s1 []int
	if s1 == nil {
		fmt.Println("是空")
	} else {
		fmt.Println("不是空")
	}
	// 2.:=
	s2 := []int{}
	// 3.make()
	var s3 []int = make([]int, 0)
	fmt.Println(s1, s2, s3)
	// 4.初始化赋值
	var s4 []int = make([]int, 0, 0)
	fmt.Println(s4)
	s5 := []int{1, 2, 3}
	fmt.Println(s5)
	// 5.从数组切片
	arr := [5]int{1, 2, 3, 4, 5}
	var s6 []int
	// 前包后不包
	s6 = arr[1:4]
	fmt.Println(s6)
}
```

输出结果

```
是空
[] [] []
[]
[1 2 3]
[2 3 4]
```

### 切片语法

![image-20230718163659605](https://gitee.com/huanglei1111/phone-md/raw/master/images/image-20230718163659605.png)

```go
package main

import "fmt"

var arr = [...]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
var slice0 = arr[2:8]
var slice1 = arr[0:6]         //可以简写为 var slice []int = arr[:end]
var slice2 = arr[5:10]        //可以简写为 var slice[]int = arr[start:]
var slice3 = arr[0:]          //var slice []int = arr[:]
var slice4 = arr[:len(arr)-1] //去掉切片的最后一个元素

func main() {
	fmt.Printf("全局变量：arr %v\n", arr)
	fmt.Printf("全局变量：slice0 %v\n", slice0)
	fmt.Printf("全局变量：slice1 %v\n", slice1)
	fmt.Printf("全局变量：slice2 %v\n", slice2)
	fmt.Printf("全局变量：slice3 %v\n", slice3)
	fmt.Printf("全局变量：slice4 %v\n", slice4)
	fmt.Printf("-----------------------------------\n")
	arr2 := [...]int{9, 8, 7, 6, 5, 4, 3, 2, 1, 0}
	slice5 := arr[2:8]
	slice6 := arr[0:6]         //可以简写为 slice := arr[:end]
	slice7 := arr[5:10]        //可以简写为 slice := arr[start:]
	slice8 := arr[0:]          //slice := arr[:]
	slice9 := arr[:len(arr)-1] //去掉切片的最后一个元素
	fmt.Printf("局部变量： arr2 %v\n", arr2)
	fmt.Printf("局部变量： slice5 %v\n", slice5)
	fmt.Printf("局部变量： slice6 %v\n", slice6)
	fmt.Printf("局部变量： slice7 %v\n", slice7)
	fmt.Printf("局部变量： slice8 %v\n", slice8)
	fmt.Printf("局部变量： slice9 %v\n", slice9)
}
```

输出结果

```
全局变量：arr [0 1 2 3 4 5 6 7 8 9]
全局变量：slice0 [2 3 4 5 6 7]
全局变量：slice1 [0 1 2 3 4 5]
全局变量：slice2 [5 6 7 8 9]
全局变量：slice3 [0 1 2 3 4 5 6 7 8 9]
全局变量：slice4 [0 1 2 3 4 5 6 7 8]
-----------------------------------
局部变量： arr2 [9 8 7 6 5 4 3 2 1 0]
局部变量： slice5 [2 3 4 5 6 7]
局部变量： slice6 [0 1 2 3 4 5]
局部变量： slice7 [5 6 7 8 9]
局部变量： slice8 [0 1 2 3 4 5 6 7 8 9]
局部变量： slice9 [0 1 2 3 4 5 6 7 8]
```

### 通过make来创建切片

```go
package main

import "fmt"

func main() {
	s1 := []int{0, 1, 2, 3, 8: 100} // 通过初始化表达式构造，可使用索引号。
	fmt.Println(s1, len(s1), cap(s1))

	s2 := make([]int, 6, 8) // 使用 make 创建，指定 len 和 cap 值。
	fmt.Println(s2, len(s2), cap(s2))

	s3 := make([]int, 6) // 省略 cap，相当于 cap = len。
	fmt.Println(s3, len(s3), cap(s3))
}
```

输出结果

```
[0 1 2 3 0 0 0 0 100] 9 9
[0 0 0 0 0 0] 6 8
[0 0 0 0 0 0] 6 6
```

### 向切片添加元素

```go
package main

import "fmt"

func main() {
	var s []int
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
	// 将0增加到切片尾部
	s = append(s, 0)
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)
	// 将1增加到切片尾部
	s = append(s, 1)
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)

	// append函数支持一次性添加多个元素
	// 将2, 3, 4添加到切片尾部
	s = append(s, 2, 3, 4)
	fmt.Printf("len=%d cap=%d %v\n", len(s), cap(s), s)

	var a = []int{1, 2, 3}
	fmt.Printf("slice a : %v\n", a)
	var b = []int{4, 5, 6}
	fmt.Printf("slice b : %v\n", b)
	c := append(a, b...)
	fmt.Printf("slice c : %v\n", c)
}
```

输出结果

```
len=0 cap=0 []
len=1 cap=1 [0]
len=2 cap=2 [0 1]
len=5 cap=6 [0 1 2 3 4]
slice a : [1 2 3]
slice b : [4 5 6]
slice c : [1 2 3 4 5 6]
```

### 如何读写切片元素

需要注意的是，因为切片底层引用的是数组，如果多个切片引用同一个数组，修改其中一个切片的元素，会影响关联的所有切片

```go
package main

import "fmt"

func main() {
	arr := [4]string{"hello", "world", "go", "slice"}
	fmt.Println(arr)

	// 创建a,b两个切片
	a := arr[0:2]
	b := arr[1:3]
	fmt.Println(a, b)

	// 修改切片b的第一个元素
	b[0] = "XXX"
	fmt.Println(a, b)
	fmt.Println(arr)
}
```

输出结果

```
[hello world go slice]
[hello world] [world go]
[hello XXX] [XXX go]
[hello XXX go slice]
```

### 遍历切片

```go
package main

import "fmt"

func main() {
	s1 := make([]int, 6)
	fmt.Println(s1)
	s1[0] = 1
	s1[2] = 1
	s1[3] = 1
	s1[4] = 1
	s1[5] = 2

	for _, value := range s1 {
		fmt.Println(value)
	}

}
```

```
// 忽略元素值
for i, _ := range arr
// 忽略下标
for _, value := range arr
// 如果只写一个变量，则忽略元素值
for i := range arr
```

输出结果

```
[0 0 0 0 0 0]
1
0
1
1
1
2
```

