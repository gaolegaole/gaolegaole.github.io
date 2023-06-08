---
title: "url query with sort"
date: 2023-06-08T18:09:25+08:00
draft: false
description: "url query with sort"
categories:

tags:
    - go
---
> 背景 请求方式前端使用antd ui ProTable 的request请求分页数据，column开始sort:true后使用服务端排序，传到后端参数如下第3种
# 1. 前端请求3种方式
* http://domain/path?sort=name&sort=age&sort=address
* http://domain/path?sort=name,age,address
* http://domain/path?sort=name%2Cage%2Caddress&order=ascend%2Cascend&2Cascend
# 2. 后端接收
```go
type reqParams struct{
    Sort []string `form:"sort"`
}
func (s *Server)AllUser(ctx *gin.Context){
    var req reqParams
    if err:=ctx.ShouldBind(&req);err!=nil{
        //handle error
    }
    println(req.Sort)
    //output
    //[name age address] slice
}
```
# 3. antd默认无法进行两列同时排序
开启服务器排序后，点击一列后就自动重新请求，没法保存之前排序的状态。待补充
