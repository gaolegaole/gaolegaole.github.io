---
title: "Use Graphjin"
date: 2023-08-09T15:03:44+08:00
draft: false
description: "Use Graphjin"
categories:

tags:

---
> graphjin是一个可以自动生成graphql的框架，可以自动生成graphql的schema，也可以自己定义schema。
# FAQ
## 1. 生产环境如何使用？
在生产环境，将yaml配置文件中的production: true配置好，当然前提是已经生成了*.gql文件（在非production模式下）。然后在配置中使用fs配置项，将config目录添加到conf中，看如下代码：
```golang
var options []core.Option
//加载queries
fs := core.NewOsFS(path.Join(util.ProjectPath(), "config"))
options = append(options, core.OptionSetFS(fs))

modeFile := "dev.yml"
if gin.Mode() == gin.ReleaseMode {
    modeFile = "prod.yml"
}
// config can be read in from a file
cfg, err := config.NewConfig(path.Join(util.ProjectPath(), "config"), modeFile)
if err != nil {
    log.Println("config error", err)
}
// or config can be a go struct
//config := core.Config{ Production: true, DefaultLimit: 50 ,DisableAllowList: true}
gj, err = core.NewGraphJin(cfg, db, options...)

if err != nil {
    log.Println(err)
}
```
当前版本(`github.com/dosco/graphjin/core/v3 v3.0.0-20230512210738-36d89904d6e1`)不知道为何在非production模式下，生成的*.gql文件中，都是如下的样式：
```gql
uery getStorageDetail {
  data: storage(where: {id: {eq: $id}}) {
    id
    name
    __typename
  }
}
```
query变成了uery，少了一个`q`，导致切换到生产模式时查询失败`invalid query: query type and name not found: getStorageDetail`，在生产环境中需要自己修改下，而gql文件中如果存在fragment，则不会出现问题。

