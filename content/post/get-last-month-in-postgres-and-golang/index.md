---
title: "Get Last Month in Postgres and Golang"
date: 2023-11-02T09:06:12+08:00
draft: false
description: "Get Last Month in Postgres and Golang"
categories:

tags:
    - postgres
    - golang
---
> 背景：业务中有在golang中获取上个月的1号的时间戳的需求，通过time.Now().AddDate(0,0,-1)获取的可能并不是上个月的时间戳。因为golang中一个月的定义是30天，如果当前时间是10月31日获取的话，返回的是10月1日的时间戳；而postgres中也有这个需求，通过date_trunc('month', CURRENT_DATE)获取的则可以准确的获取到上个月，无论是否跨年。
# golang获取上个月
```go
func Test6(t *testing.T) {
	e1 := "2023-10-31 12:00:00"
	datetime, err := time.ParseInLocation("2006-01-02 15:04:05", e1, time.Local)
	if err != nil {
		t.Fatal(err)
	}
	date := datetime.AddDate(0, -1, 0)
	t.Log(date.Year(), date.Month(), date.Day())
	//output 2023 October 1      2023年10月1日
}
```
上面明显是错误的，而除了每月1号，和2月其他日期都是正确的。
## 通过以下函数可以获取到准确的
```go   
func LastMonthTimestamp(date ...time.Time) int {
	var now time.Time
	if len(date) == 0 || date[0].IsZero() {
		now = time.Now()
	} else {
		now = date[0]
	}
	// 计算上个月的时间
	lastMonth := _lastMonth(now)

	//如果是1号，则取上上个月的数据，11.1 则取9月数据（10月月度数据没有）
	if now.Day() == 1 {
		lastMonth = _lastMonth(lastMonth)
	}

	// 设置时间为 0 时 0 分 0 秒
	lastMonthMidnight := time.Date(lastMonth.Year(), lastMonth.Month(), 1, 0, 0, 0, 0, time.Local)

	// 获取时间戳
	lastMonthTimestamp := lastMonthMidnight.Unix()

	return int(lastMonthTimestamp)
}
func _lastMonth(now time.Time) time.Time {
	if now.Month() == 1 {
		return time.Date(now.Year()-1, 12, 1, 0, 0, 0, 0, time.Local)
	}
	return time.Date(now.Year(), now.Month()-1, 1, 0, 0, 0, 0, time.Local)
}
func StartTimestamp(date ...time.Time) int {
	// 获取当前时间
	var now time.Time
	if len(date) == 0 || date[0].IsZero() {
		now = time.Now()
	} else {
		now = date[0]
	}
	var thisMonthMidnight time.Time
	//如果是1号，则取上月的数据
	if now.Day() == 1 {
		now = _lastMonth(now)
	}
	// 设置时间为 0 时 0 分 0 秒
	thisMonthMidnight = time.Date(now.Year(), now.Month(), 1, 0, 0, 0, 0, time.Local)

	// 获取时间戳
	thisMonthTimestamp := thisMonthMidnight.Unix()

	return int(thisMonthTimestamp)
}
func daysInYearMonth(year int, month time.Month) int {
	// 选择要查询的年份和月份

	// 使用 time.Date 创建下个月的时间
	if month == time.December {
		//12月
		year += 1
		month = time.January
	}
	nextMonth := time.Date(year, month+1, 1, 0, 0, 0, 0, time.Local)

	// 将时间减去一天，得到指定月份的最后一天
	lastDayOfMonth := nextMonth.Add(-time.Second)

	// 获取该月有多少天
	daysInMonth := lastDayOfMonth.Day()

	//fmt.Printf("%d年%d月有%d天\n", year, month, daysInMonth)
	return daysInMonth
}
func EndTimestamp(date ...time.Time) int {
	// 获取当前时间
	var now time.Time
	if len(date) == 0 || date[0].IsZero() {
		now = time.Now()
	} else {
		now = date[0]
	}

	//如果是1号，则取上月的数据
	day := now.Day()
	if now.Day() == 1 {
		now = _lastMonth(now)
		//上月月底
		day = daysInYearMonth(now.Year(), now.Month())
	}

	// 设置时间为 0 时 0 分 0 秒
	todayMidnight := time.Date(now.Year(), now.Month(), day, 0, 0, 0, 0, time.Local)

	// 获取时间戳
	todayMidnightTimestamp := todayMidnight.Unix()

	return int(todayMidnightTimestamp)
}

```
测试函数
```go
func Test3(t *testing.T) {
	tests := []struct {
		name string
		date string
		want string
	}{
		{name: "ok", date: "20231101-010101", want: "20230901-000000"},
		{name: "ok", date: "20231002-010101", want: "20230901-000000"},
		{name: "ok", date: "20231031-010101", want: "20230901-000000"},
		{name: "ok", date: "20231201-010101", want: "20231001-000000"},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			location, err := time.ParseInLocation("20060102-150405", tt.date, time.Local)
			if err != nil {
				t.Fatal(err)
			}
			if got := time.Unix(int64(LastMonthTimestamp(location)), 0).Format("20060102-150405"); got != tt.want {
				t.Fatalf("want:%s \n got:%s", tt.want, got)
			}
		})
	}
}
func Test4(t *testing.T) {
	tests := []struct {
		name string
		date string
		want string
	}{
		{name: "ok", date: "20231101-010101", want: "20231001-000000"},
		{name: "ok", date: "20231102-010101", want: "20231101-000000"},

		{name: "ok", date: "20230101-010101", want: "20221201-000000"},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			location, err := time.ParseInLocation("20060102-150405", tt.date, time.Local)
			if err != nil {
				t.Fatal(err)
			}
			if got := time.Unix(int64(StartTimestamp(location)), 0).Format("20060102-150405"); got != tt.want {
				t.Fatalf("want:%s \n got:%s", tt.want, got)
			}
		})
	}
}
func Test5(t *testing.T) {
	tests := []struct {
		name string
		date string
		want string
	}{
		{name: "ok", date: "20231101-010101", want: "20231031-000000"},
		{name: "ok", date: "20231102-010101", want: "20231102-000000"},

		{name: "ok", date: "20230101-010101", want: "20221231-000000"},
	}

	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			location, err := time.ParseInLocation("20060102-150405", tt.date, time.Local)
			if err != nil {
				t.Fatal(err)
			}
			if got := time.Unix(int64(EndTimestamp(location)), 0).Format("20060102-150405"); got != tt.want {
				t.Fatalf("want:%s \n got:%s", tt.want, got)
			}
		})
	}
}
```
# postgres获取上个月
使用date_trunc函数，截取想要的部分，如：'month'
```sql
select TO_CHAR(date_trunc('month' , current_date),'YYYY-MM'); -- 2023-11
```
更复杂的例子，如果是1号则获取上个月的日期（202311，20231101,20231131），否则获取本月的日期
```sql
select
	case
		when TO_CHAR(DATE_TRUNC('MONTH', CURRENT_DATE)::DATE, 'DD')::numeric::text = '1' then
		-- 上月
		TO_CHAR((DATE_TRUNC('MONTH', CURRENT_DATE) + interval '-1 MONTH')::DATE, 'YYYYMM')
		else
		-- 本月
		TO_CHAR(DATE_TRUNC('MONTH', CURRENT_DATE)::DATE, 'YYYYMM')
	end as cycle_id;

select
	case
		when TO_CHAR(DATE_TRUNC('MONTH', CURRENT_DATE)::DATE, 'DD')::numeric::text = '1' then
		-- 上月
		TO_CHAR((DATE_TRUNC('MONTH', CURRENT_DATE) + interval '-1 MONTH')::DATE, 'YYYY-MM-DD')
		else
		-- 本月
		CONCAT(TO_CHAR(DATE_TRUNC('MONTH', CURRENT_DATE)::DATE, 'YYYY-MM-'), '01')
	end as nowsaledate;

select
	case
		when TO_CHAR(DATE_TRUNC('MONTH', CURRENT_DATE)::DATE, 'DD')::numeric::text = '1' then
		-- 上月
		TO_CHAR((DATE_TRUNC('MONTH', CURRENT_DATE) + interval '-1 DAY')::DATE, 'YYYY-MM-DD')
		else
		-- 本月
		TO_CHAR((DATE_TRUNC('MONTH', CURRENT_DATE) + interval '-1 MONTH' - interval '1 DAY')::DATE, 'YYYY-MM-DD')
	end as nextsaledate;
```
每月最后一天获取方式，通过月份（不带天数的日期：202311）+1个月-1天，刚好是每月最后一天。

# 千分位分割格式化数字
在有些地方需要用千分位来分割较长的数字，如：金额、数量等，格式如：1,234,567.89
```go
func FormatFloatWithCommas(number float64) string {
	// 将浮点数四舍五入到4位小数
	rounded := fmt.Sprintf("%.4f", number)

	// 使用 strconv.FormatFloat 将浮点数格式化为千分位
	parts := strings.Split(rounded, ".")
	intPart := parts[0]
	decimalPart := ""
	if len(parts) > 1 {
		decimalPart = "." + parts[1]
	}

	// 格式化整数部分，添加千分位逗号
	formattedInt := formatIntWithCommas(intPart)

	// 返回格式化后的字符串
	return formattedInt + decimalPart
}
func formatIntWithCommas(s string) string {
	n := len(s)
	if n <= 3 {
		return s
	}

	return formatIntWithCommas(s[:n-3]) + "," + s[n-3:]
}


```