---

layout:     post
title:      "Go学习之slice append底层原理 
subtitle:   "Go学习记录"
date:       2021-12-26 21:57:00
author:     "WS"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - Golang
---

###  

​	最近在学习go，学到切片slice，其中有一个内置方法append，作用是向切片追加新元素的 append 方法。相当于涉及到切片cap扩容的问题，什么时候扩容，扩容到多大？

​    测试方法代码如下：

```go
package main

import "fmt"

func main() {
	var numbers []int
	printSlice(numbers)

	numbers = append(numbers, 0)
	printSlice(numbers)

	numbers = append(numbers, 1)
	printSlice(numbers)

	numbers = append(numbers, 2, 3, 4)
	printSlice(numbers)

	numbers = append(numbers, 5, 6)
	printSlice(numbers)
}

func printSlice(x []int) {
	fmt.Printf("len = %d cap = %d slice = %v\n", len(x), cap(x), x)
}
```

​    printSlice中打印出存在元素len，实际切片空间大小cap:

```powershell
PS C:\workspace\GolangDailyLearn> go run .\sliceDemo1.go      
len = 0 cap = 0 slice = []
len = 1 cap = 1 slice = [0]
len = 2 cap = 2 slice = [0 1]
len = 5 cap = 6 slice = [0 1 2 3 4]
len = 7 cap = 12 slice = [0 1 2 3 4 5 6]
```

​	很多blog都说slice扩容是按二倍来扩展，但是显然从2-6不是这个按这个逻辑，深入了解后学习到go slice扩容机制涉及到是扩容字符是单个还是多个，按大小扩容完后还得按内存对齐**`roundupsize`**，相当于是向上取整的空间，所以并不是都按二倍来扩容。

​	总结来说就是：

- append单个元素，或者append少量的多个元素，这里的少量指double之后的容量能容纳，这样就会走以下扩容流程，不足1024，双倍扩容，超过1024的，1.25倍扩容。
- 若是append多个元素，且double后的容量不能容纳，直接使用预估的容量。
- 得到新容量后，均需要根据slice的类型size，算出新的容量所需的内存情况`capmem`，然后再进行`capmem`向上取整，得到新的所需内存，除上类型size，得到真正的最终容量,作为新的slice的容量



参考链接：

  [(10条消息) golang中切片扩容后的大小问题_hfutyyc的博客-CSDN博客_golang 切片扩容](https://blog.csdn.net/hfutyyc/article/details/97612190)

[Go slice扩容深度分析 - 掘金 (juejin.cn)](https://juejin.cn/post/6844903812331732999#非常规操作)

