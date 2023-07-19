# Map

map是一种无序的基于key-value的数据结构，Go语言中的map是引用类型，必须初始化才能使用

### 定义map类型

```go
var m map[key类型]value类型
```

### 初始化map

> map必须要先初始化才能使用

没有初始化的map变量，默认值是nil，需要通过make初始化map类型变量

```go
make(map[key类型]value类型)
```

例子：

```go
// 定义一个map
var m map[string]int

// 初始化map
m = make(map[string]int)

// 不需要预先定义变量m的类型，直接通过短变量声明符（:=）,直接定义和初始化变量
m := make(map[string]string)
```

### 读取数据

```go
package main

import "fmt"

func main() {

	m := make(map[string]int)

	m["hello"] = 100
	m["world"] = 200

	v := m["hello"]
	fmt.Println(v)

	fmt.Println(m)

	n := map[string]int{
		"zhangsan": 1,
		"lisi":     2,
	}

	v1 := n["lisi"]
	fmt.Println(v1)

	fmt.Println(n)
}

```

输出结果

```
100
map[hello:100 world:200]
2
map[lisi:2 zhangsan:1]
```

### 判断某个键是否存在

Go语言中有个判断map中键是否存在的特殊写法，格式如下:

```
    value, ok := map[key]   
```

```go
package main

import "fmt"

func main() {
	scoreMap := make(map[string]int)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	// 如果key存在ok为true,v为对应的值；不存在ok为false,v为值类型的零值
	v, ok := scoreMap["张三"]
	i, ok := scoreMap["小明"]

	if ok {
		fmt.Println(i)
	} else {
		fmt.Println("查无此人小明")
	}

	if ok {
		fmt.Println(v)
	} else {
		fmt.Println("查无此人张三")
	}
}

```

输出结果

```
100
90
```

### 删除数据

使用delete()内建函数从map中删除一组键值对，delete()函数的格式如下：

```
    delete(map, key)  
```

```go
package main

import "fmt"

func main() {
	scoreMap := make(map[string]int)
	scoreMap["张三"] = 90
	scoreMap["小明"] = 100
	scoreMap["王五"] = 60
	fmt.Println("删除之前", scoreMap)
	delete(scoreMap, "小明") //将小明:100从map中删除
	fmt.Println("删除之后", scoreMap)
}
```

输出结果

```
删除之前 map[小明:100 张三:90 王五:60]
删除之后 map[张三:90 王五:60]
```

### 遍历map

```go
package main

import "fmt"

func main() {

	m := make(map[string]string)

	m["张三"] = "100"
	m["李四"] = "200"
	m["王五"] = "300"

	for key, value := range m {
		fmt.Println(key, value)
	}

	//只打印map的key
	for key := range m {
		fmt.Println(key)
	}

	//只打印map的key可以省略
	for _, v := range m {
		fmt.Println(v)
	}

}
```

输出结果

```
李四 200
王五 300
张三 100
李四
王五
张三
300
100
200
```

### 元素为map类型的切片

```go
package main

import "fmt"

func main() {
	var mapSlice = make([]map[string]string, 3)
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v\n", index, value)
	}
	fmt.Println("after init")
	// 对切片中的map元素进行初始化
	mapSlice[0] = make(map[string]string, 10)
	mapSlice[0]["name"] = "王五"
	mapSlice[0]["password"] = "123456"
	mapSlice[0]["address"] = "红旗大街"
	for index, value := range mapSlice {
		fmt.Printf("index:%d value:%v\n", index, value)
	}
}
```

输出结果

```
index:0 value:map[]
index:1 value:map[]
index:2 value:map[]
after init
index:0 value:map[address:红旗大街 name:王五 password:123456]
index:1 value:map[]
index:2 value:map[]
```

### 值为切片类型的map

```go
package main

import "fmt"

func main() {
	var sliceMap = make(map[string][]string, 3)
	fmt.Println(sliceMap)
	fmt.Println("after init")
	key := "中国"
	value, ok := sliceMap[key]
	if !ok {
		value = make([]string, 0, 2)
	}
	value = append(value, "北京", "上海")
	sliceMap[key] = value
	fmt.Println(sliceMap)
}

```

输出结果

```
map[]
after init
map[中国:[北京 上海]]
```

