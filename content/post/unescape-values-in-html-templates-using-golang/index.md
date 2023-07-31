---
title: "Unescape Values in Html Templates Using Golang"
date: 2023-07-31T16:24:10+08:00
draft: false
description: "Unescape Values in Html Templates Using Golang"
categories:

tags:
    - golang
---
## 问题

使用gin框架将map输出到template模板中的javascript文件的时候，会自动进行转义，导致无法将数据赋值给变量。

## 解决方案

使用html/template包的Unescape函数，将转义后的字符串还原。

[Reference https://h1z3y3.medium.com/unescape-values-in-html-templates-using-golang-6e1020c25b7a](https://h1z3y3.medium.com/unescape-values-in-html-templates-using-golang-6e1020c25b7a)

```go
r.GET("/tenants", func(ctx *gin.Context) {
		sql := `
select * from tenant ;
`
		var data []map[string]interface{}
		var code int
		var msg string
		err := store.Raw(sql).Scan(&data).Error
		marshal, _ := json.Marshal(&data)
		if err != nil {
			code = 1
			msg = err.Error()
		}
		ctx.HTML(200, "tenants", gin.H{
			"code": code,
			"msg":  msg,
			"data": string(marshal),
		})
	})
```
```html
<script type="text/babel">
    var tenants = {{.data}};
    //output
    //const data = [{&#34;created_at&#34;:&#34;2023-07-10T17:21:47.64&#43;08:00&#34;,&#34;deleted_at&#34;:null,&#34;id&#34;:1,&#34;level&#34;:1,&#34;name&#34;:&#34;test1&#34;,&#34;status&#34;:1,&#34;tenant&#34;:&#34;01G65Z755AFWAKHE12NY0CQ9FH&#34;,&#34;updated_at&#34;:null}];
</script>
```

使用template函数解决
```go
func unescapeHTML(s string) template.HTML {
	return template.HTML(s)
}
r.SetFuncMap(template.FuncMap{
		"unescapeHTML": unescapeHTML,
	})
```

```html
<script type="text/babel">
const data = {!{ .data | unescapeHTML }!};
//output
//const data = [{"created_at":"2023-07-10T17:21:47.64+08:00","deleted_at":null,"id":1,"level":1,"name":"test1","status":1,"tenant":"01G65Z755AFWAKHE12NY0CQ9FH","updated_at":null}];
</script>
```
> 可能是由于在script type="text/babel"中的原因，在script标签中不使用函数，直接使用JSON.parse({!{.data}!})也可以