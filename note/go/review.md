---
title: Golang 知识点复习/易错点记录
author: Laeni
tags: go,golang
date: '2022-10-10'
updated: '2023-02-14'
---

# 通道

1. 向关闭的通道发送请求会导致`panic`

# 切片（Slice）

1. 典型案例1

   ```go
   func main() {
   	intSlice := []int{1, 2, 3, 4, 5}
   	i1 := intSlice[:3]
   	i2 := intSlice[2:]
   	intSlice[2] = 0
   	fmt.Printf("intSlice: %v  len: %v  cap: %v\n", intSlice, len(intSlice), cap(intSlice))
   	fmt.Printf("i1: %v  len: %v  cap: %v\n", i1, len(i1), cap(i1))
   	fmt.Printf("i2: %v  len: %v  cap: %v\n", i2, len(i2), cap(i2))
   }
   /*
   intSlice: [1 2 0 4 5]  len: 5  cap: 5
   i1:       [1 2 0]      len: 3  cap: 5
   i2:       [0 4 5]      len: 3  cap: 3
   */
   ```

   上述示例说明：

   1. 切片底层是对数组的引用。
   2. 切片值传递时，底层数组虽然引用的是同一个，但是切片的长度和容量是复制的。
   3. 向后切片时，切片的容量会减小。（假如一直往后切片不知道会不会引起内存泄露）

# gorutine

1. 典型案例1

   ```go
   func main() {
   	var wg sync.WaitGroup
   	intSlice := []int{1, 2, 3, 4, 5}
   	wg.Add(len(intSlice))
   	ans1, ans2, ans3 := 0, 0, 0
   	for _, v := range intSlice {
   		vv := v
   		go func(v3 int) {
   			defer wg.Done()
   			ans1 += v
   			ans2 += vv
   			ans3 += v3
   		}(v)
   	}
   	wg.Wait()
   	fmt.Println(ans1, ans2, ans3)
   }
   ```

   上述代码中`ans1`、`ans2`和`ans3`均不一定等于`15`。`ans2`和`ans3`虽然使用的都是`v`的复制，但是在多**gorutine**中操作同一个变量是导致它们值不一样的原因，而`ans1`除了上述原因之外，还存在另一个原因：在`for`循环中，`v`只在第一次声明，后面都是修改第一次声明的变量的值，再加上闭包函数声明时引用的时变量的地址，而不是值复制，所以等实际**gorutine**执行时访问到的值可能已经发生了变化。

   但是实际验证时可能结果是固定的，原因**gorutine**太少，可能只会被一个线程执行，但是当增加**gorutine**数量时，可能就会被分配给多个线程，那样的话每次的结果可能就不一样了：

   ```go
   func main() {
   	var wg sync.WaitGroup
   	intSlice := make([]int, 10000)
   	for i := range intSlice {
   		intSlice[i] = i + 1
   	}
   
   	wg.Add(len(intSlice))
   	ans1, ans2, ans3 := 0, 0, 0
   	for _, v := range intSlice {
   		vv := v
   		go func(v3 int) {
   			defer wg.Done()
   			ans1 += v
   			ans2 += vv
   			ans3 += v3
   		}(v)
   	}
   	wg.Wait()
   	fmt.Println(ans1, ans2, ans3)
   }
   ```

# 其他

1. 类型转换（`Type(expression)`）用于两种不同的类型，在`type A B`中`A`和`B`是两种不同的类型，但是它们是兼容的，所以使用**转换**，而不是类型断言（`expression.(Type)`）；类型断言适用于“它本身就是那种类型“的情况，并不存在转换。

