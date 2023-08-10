---
title: "golang"
date: 2023-08-10T09:32+08:00
draft: false
description: "golang"
categories:

tags:
    
---
# x
### 可以struct嵌套interface使用
这个 #gin-middleware #gin 使用在当需要给返回的header中添加信息时使用，如刷新token等。
```golang
type afterMiddlewareWriter struct {  
	gin.ResponseWriter  
	*token.Payload  
	token.Maker  
}  
  
func (w *afterMiddlewareWriter) WriteHeader(statusCode int) {  
	if time.Now().Add(time.Hour).After(w.ExpiredAt) {  
		//还有一个小时过期  
		duration := time.Hour * 24 * 7  
		createdToken, err := w.CreateToken(w.Username, w.Name, w.Tenant, duration)  
		if err != nil {  
			log.Println("create token error:", err.Error())  
		} else {  
			w.Header().Add("token", createdToken)  
			//TODO 将token写入token表，否则无法登陆  
			store, _ := model.GetStoreInstance("./app.yaml")  
			//TODO 使用缓存提高性能  
			m := &model.Token{  
			Username: w.Username,  
			Token: createdToken,  
		}  
		m.Tenant = w.Tenant  
		err := store.Debug().Model(m).Where("username = ? and tenant = ?", w.Username, w.Tenant).Updates(map[string]interface{}{"token": createdToken, "expires": time.Now().Add(duration)}).Error  
		if err != nil {  
			// w.ResponseWriter.WriteHeader(500)  
			// w.ResponseWriter.WriteString("内部错误")  
			// return  
			log.Println(err.Error())  
		}  
		  
		}  
	}  
	w.ResponseWriter.WriteHeader(statusCode)  
}  

func AfterMiddleware(c *gin.Context) {  
	c.Writer = &afterMiddlewareWriter{c.Writer}  
	c.Next()  
}  

```