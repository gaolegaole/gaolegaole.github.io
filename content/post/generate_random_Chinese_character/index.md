---
title: "golang生成中文随机字符串"
date: 2023-07-20T10:30:47+08:00
draft: false
description: "Generate_random_Chinese_character"
categories:

tags:
    - golang
---
> 中文字符串unicode编码范围，参考：[Reference https://www.qqxiuzi.cn/zh/hanzi-unicode-bianma.php](https://www.qqxiuzi.cn/zh/hanzi-unicode-bianma.php) [Reference https://zh.wikipedia.org/wiki/Unicode%E5%8D%80%E6%AE%B5](https://zh.wikipedia.org/wiki/Unicode%E5%8D%80%E6%AE%B5)

# 随机中文
```golang
// RandomStringChinese 生成中文字符
// 字符区间参考：https://www.qqxiuzi.cn/zh/hanzi-unicode-bianma.php
func RandomStringChinese(length int) string {
	var result string
	for i := 0; i < length; i++ {
		index := rand.Intn(0x9fa5-0x4e00) + 0x4e00
		result += string(rune(index))
	}
	return result
}
```
需要注意的是，如果要生成区间数字，可以使用以下公式：rand.Intn(max - min) + min ,rand.Intn(x)生成的是0 - x之间的数字（不包含x）。

unicode编码转character使用run(0x4e00)进行转换。
# 随机英文数字字符串
```golang
const alphabetAndNumber = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"

func RandomString(length int) string {
	var result string
	for i := 0; i < length; i++ {
		index := rand.Intn(len(alphabetAndNumber))
		result += string([]rune(alphabetAndNumber)[index])
	}
	return result
}
```
# 其他的一些随机生成
```golang
const alphabetAndNumber = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789"
func RandomDate() string {
	/*year := rand.Intn(24) + 2000
	month := rand.Intn(12) + 1
	dayOfMonth := rand.Intn(28) + 1
	return time.Date(year, time.Month(month), dayOfMonth, 0, 0, 0, 0, time.Local).Format("2006-01-02")*/
	return RandomTime().Format("2006-01-02")
}
// RandomMoney 生成两位小数的float64
func RandomMoney() float64 {
	var result float64
	for {
		result = Money(rand.Float64() * float64(random10X()))
		if result > 0.0 {
			break
		}
	}
	return result
}
// RandomMoney 生成三位小数的float64
func RandomCount() float64 {
	var result float64
	for {
		result = Count(rand.Float64() * float64(random10X()))
		if result > 0.0 {
			break
		}
	}
	return result
}
func random10X() int {
	nums := []int{0, 10, 100, 1000, 10000, 100000}
	index := rand.Intn(len(nums))
	return nums[index]
}

func RandomTime() time.Time {
	now := time.Now()
	// 生成随机时间
	randomTime := time.Date(
		now.Year(),                  // 使用当前年份
		time.Month(rand.Intn(12)+1), // 生成1到12之间的随机月份
		rand.Intn(28)+1,             // 生成1到28之间的随机日期
		rand.Intn(24),               // 生成0到23之间的随机小时数
		rand.Intn(60),               // 生成0到59之间的随机分钟数
		rand.Intn(60),               // 生成0到59之间的随机秒数
		rand.Intn(1000)*1e6,         // 生成0到999之间的随机纳秒数
		now.Location(),              // 使用当前时区
	)
	return randomTime
}
```
