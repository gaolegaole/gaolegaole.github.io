---
title: "Golang运算符"
date: 2023-06-01T11:50:07+08:00
draft: false
description: "Golang运算符"
categories:

tags:
    - go
---
Go 语言支持的位运算符如下表所示。假定 A 为60，B 为13：
|运算符|	描述|	实例|
|---|---|---|
|&|	按位与运算符"&"是双目运算符。 其功能是参与运算的两数各对应的二进位相与。	|(A & B) 结果为 12, 二进制为 0000 1100|
|\||	按位或运算符"\|"是双目运算符。 其功能是参与运算的两数各对应的二进位相或	|(A \| B) 结果为 61, 二进制为 0011 1101|
|^|	按位异或运算符"^"是双目运算符。 其功能是参与运算的两数各对应的二进位相异或，当两对应的二进位相异时，结果为1。|	(A ^ B) 结果为 49, 二进制为 0011 0001|
|<<	|左移运算符"<<"是双目运算符。左移n位就是乘以2的n次方。 其功能把"<<"左边的运算数的各二进位全部左移若干位，由"<<"右边的数指定移动的位数，高位丢弃，低位补0。	|A << 2 结果为 240 ，二进制为 1111 0000|
|>>|	右移运算符">>"是双目运算符。右移n位就是除以2的n次方。 其功能是把">>"左边的运算数的各二进位全部右移若干位，">>"右边的数指定移动的位数。	|A >> 2 结果为 15 ，二进制为 0000 1111|

```go
// You can edit this code!
// Click here and start typing.
package main

import (
	"math"
	"testing"
)

func TestUser(t *testing.T) {

	str := "string"
	t.Logf("%s", str)
	t.Logf("%q", str) //带双引号 "string"
	t.Logf("%v", str)
	t.Logf("%c", 97) //输出一个数字的字符ascii a

	//位移
	x := 5
	t.Logf("%d 二进制：%b", x, x)
	t.Logf("左移：左移n位就是乘以2的n次方。将5二进制位全部左移一位，高位丢弃，低位补0 %d<<1 = %d , 二进制：%b", x, x<<1, x<<1)
	t.Logf("右移：右移n位就是除以2的n次方。将5二进制位全部右移一位，高位补0，低位丢弃 %d>>1 = %d , 二进制：%b", x, x>>1, x>>1)
	t.Logf("math.MaxUint8 = %d", math.MaxUint8)

	//t.Logf("math.MaxUint8左移1位：%b << 1 = %b", math.MaxUint8, math.MaxUint8<<1)
	//上面的左移超出了int的最大值，报错
	t.Logf("math.MaxUint8 ，右移1位：%b >> 1 = %b，十进制为：%d", math.MaxUint8, math.MaxUint8>>1, math.MaxUint8>>1)

	for i := 0; i <= 1; i++ {
		for j := 0; j <= 1; j++ {
			for _, index := range []string{"&", "|", "^"} {
				switch index {
				case "&":
					t.Logf("%d & %d = %b", i, j, i&j)
				case "|":
					t.Logf("%d | %d = %b", i, j, i|j)
				case "^":
					t.Logf("%d ^ %d = %b", i, j, i^j)
				}
			}
		}
	}

}
```
output:
```shell
=== RUN   TestUser
    prog_test.go:13: string
    prog_test.go:14: "string"
    prog_test.go:15: string
    prog_test.go:16: a
    prog_test.go:20: 5 二进制：101
    prog_test.go:21: 左移：左移n位就是乘以2的n次方。将5二进制位全部左移一位，高位丢弃，低位补0 5<<1 = 10 , 二进制：1010
    prog_test.go:22: 右移：右移n位就是除以2的n次方。将5二进制位全部右移一位，高位补0，低位丢弃 5>>1 = 2 , 二进制：10
    prog_test.go:23: math.MaxUint8 = 255
    prog_test.go:27: math.MaxUint8 ，右移1位：11111111 >> 1 = 1111111，十进制为：127
    prog_test.go:34: 0 & 0 = 0
    prog_test.go:36: 0 | 0 = 0
    prog_test.go:38: 0 ^ 0 = 0
    prog_test.go:34: 0 & 1 = 0
    prog_test.go:36: 0 | 1 = 1
    prog_test.go:38: 0 ^ 1 = 1
    prog_test.go:34: 1 & 0 = 0
    prog_test.go:36: 1 | 0 = 1
    prog_test.go:38: 1 ^ 0 = 1
    prog_test.go:34: 1 & 1 = 1
    prog_test.go:36: 1 | 1 = 1
    prog_test.go:38: 1 ^ 1 = 0
--- PASS: TestUser (0.00s)
PASS

Program exited.
```
[goplayground](https://go.dev/play/p/yB5V9JK8mSO)



参考：\
[https://learn.microsoft.com/zh-cn/cpp/cpp/left-shift-and-right-shift-operators-input-and-output?view=msvc-170](https://learn.microsoft.com/zh-cn/cpp/cpp/left-shift-and-right-shift-operators-input-and-output?view=msvc-170)\
[https://www.runoob.com/go/go-operators.html](https://www.runoob.com/go/go-operators.html)