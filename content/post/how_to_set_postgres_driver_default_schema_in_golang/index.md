---
title: "golang中使用postgres设置默认schema"
date: 2023-07-19T14:30:53+08:00
draft: false
description: "How_to_set_postgres_driver_default_schema_in_golang"
categories:

tags:
    - golang
---
> 在使用database/sql时如何配置默认的schema，在执行query时不需要指定schema?有两种指定方式

我们的测试数据库是test,schema是myschema,测试数据表是users,只有两个字段id,full_name。

# DDL:
```sql
create schema myschema;

set search_path = 'myschema';

create table users(
    id serial,
    full_name text
);

insert into users(full_name)
values('User 1');
```
# Data:
|id |full_name|
---|---
 | 1|User 1   |

# 1. 在dsn中定义
What is dsn? [Reference https://stackoverflow.com/questions/3582552/what-is-the-format-for-the-postgresql-connection-string-url](https://stackoverflow.com/questions/3582552/what-is-the-format-for-the-postgresql-connection-string-url)

```golang
package tests

import (
	"context"
	"database/sql"
	"fmt"
	_ "github.com/lib/pq"
	"github.com/stretchr/testify/assert"
	"testing"
)
func TestDB(t *testing.T) {
    type config struct{
        DBHost string     
		DBPort string     
		DBName string     
		DBUsername string 
		DBPassword string 
		DBSchema string   
    }
	cfg := config{
		DBHost:     "localhost",
		DBPort:     "5432",
		DBName:     "test",
		DBUsername: "test",
		DBPassword: "test",
		DBSchema:   "myschema",
	}
	var err error

	//postgres
	dsn := fmt.Sprintf("host=%s port=%s user=%s dbname=%s password=%s search_path=%s sslmode=disable TimeZone=Asia/Shanghai",
		cfg.DBHost,
		cfg.DBPort,
		cfg.DBUsername,
		cfg.DBName,
		cfg.DBPassword,
        cfg.DBSchema)
	db, err := sql.Open("postgres", dsn)
	assert.NoError(t, err)
	assert.NotNil(t, db)

	rows, err := db.Query("select id,full_name from users")
	assert.NoError(t, err)
	type user struct {
		ID       uint
		FullName string
	}
	var users []user
	for {
		if !rows.Next() {
			break
		}
		var u user
		err := rows.Scan(&u.ID, &u.FullName)
		assert.NoError(t, err)
		users = append(users, u)
	}
	assert.NoError(t, err)
	assert.True(t, len(users) > 0)
	t.Log(users[0].FullName)
}
```
# 2. 使用SQL指定
```golang
package tests

import (
	"context"
	"database/sql"
	"fmt"
	_ "github.com/lib/pq"
	"github.com/stretchr/testify/assert"
	"testing"
)
func TestDB(t *testing.T) {
    ...
	dsn := fmt.Sprintf("host=%s port=%s user=%s dbname=%s password=%s sslmode=disable TimeZone=Asia/Shanghai",
		cfg.DBHost,
		cfg.DBPort,
		cfg.DBUsername,
		cfg.DBName,
		cfg.DBPassword)
	db, err := sql.Open("postgres", dsn)
	assert.NoError(t, err)
	assert.NotNil(t, db)
    //这里指定
    db.Exec("set search_path = $1;", cfg.DBSchema)
    //or db.Exec("set search_path to $1;", cfg.DBSchema)
    ...
}
```
