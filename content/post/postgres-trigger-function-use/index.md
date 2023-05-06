---
title: "Postgres Trigger Function Use"
date: 2023-05-06T17:26:16+08:00
draft: false
description: "Postgres Trigger Function Use"
categories:
tags:
    - postgres
    - trigger
    - golang
---
# 背景
业务中有一个表newtable1存在2种业务4个字段（线上会：real_stime,real_etime;线下会real_offline_stime,real_offline_etime),由于在前端来说无论线上会还是线下会这俩个字段(*stime,*etime;开始时间，结束时间)都应该一致，并且可以根据时间（stime或etime)来筛选，由于后端业务问题，没办法合并这些两种业务的4个字段为2个字段；所以使用触发器来保持real_stime == real_offline_stime ; real_etime == real_offline_etime。
# newtable DDL
```sql
CREATE TABLE newtable1 (
	id serial4 NOT NULL,
	"content" text NULL,
	real_stime text NULL,
	real_etime text NULL,
	real_offline_stime text NULL,
	real_offline_etime text NULL
);
```
# 触发器函数
```sql
CREATE OR REPLACE FUNCTION update_real_stime_func() RETURNS TRIGGER AS $BODY$
   begin
	  case 
	  when TG_OP = 'INSERT' then 
	  	if new.real_stime is not null and new.real_offline_stime is null then
	  		new.real_offline_stime := new.real_stime;
	  	elsif new.real_offline_stime is not null and new.real_stime is null then 
	  		new.real_stime := new.real_offline_stime;
	  	end if;
	  	
	  	if new.real_etime is not null and new.real_offline_etime is null then
	  		new.real_offline_etime := new.real_etime;
	  	elsif new.real_offline_etime is not null and new.real_etime is null then 
	  		new.real_etime := new.real_offline_etime;
	  	end if;
	  	
	  when TG_OP = 'UPDATE' then
	  	if new.real_stime is not null and (new.real_offline_stime = old.real_offline_stime or new.real_offline_stime is null ) then
	  		new.real_offline_stime := new.real_stime;
	  	elsif new.real_offline_stime is not null and (new.real_stime = old.real_stime or new.real_stime is null) then 
	  		new.real_stime := new.real_offline_stime;
	  	end if;
	  	
	  	if new.real_etime is not null and (new.real_offline_etime = old.real_offline_etime or new.real_offline_etime is null) then
	  		new.real_offline_etime := new.real_etime;
	  	elsif new.real_offline_etime is not null and (new.real_etime = old.real_etime or new.real_etime is null) then 
	  		new.real_etime := new.real_offline_etime;
	  	end if;
	  end case;	
      RETURN NEW;
   END;
$BODY$ LANGUAGE plpgsql;
```
# 触发器
```sql
CREATE TRIGGER update_real_stime_trigger 
before INSERT or update of real_stime,real_etime,real_offline_stime,real_offline_etime
ON newtable1 
FOR EACH ROW 
EXECUTE PROCEDURE update_real_stime_func();
```
# 单元测试
```go
//trigger.go
package db

import (
	"fmt"
	"gorm.io/driver/postgres"
	"gorm.io/gorm"
	"gorm.io/gorm/schema"
)

type Newtable1 struct {
	ID               int
	RealStime        string
	RealEtime        string
	RealOfflineStime string
	RealOfflineEtime string
	Content          string
}

func CreateTriggerFunction() error {
	db, err := GetStore()
	if err != nil {
		return err
	}
	//create trigger
	sql := `
CREATE OR REPLACE FUNCTION update_real_stime_func() RETURNS TRIGGER AS $BODY$
   begin
	  case 
	  when TG_OP = 'INSERT' then 
	  	if new.real_stime is not null and new.real_offline_stime is null then
	  		new.real_offline_stime := new.real_stime;
	  	elsif new.real_offline_stime is not null and new.real_stime is null then 
	  		new.real_stime := new.real_offline_stime;
	  	end if;
	  	
	  	if new.real_etime is not null and new.real_offline_etime is null then
	  		new.real_offline_etime := new.real_etime;
	  	elsif new.real_offline_etime is not null and new.real_etime is null then 
	  		new.real_etime := new.real_offline_etime;
	  	end if;
	  	
	  when TG_OP = 'UPDATE' then
	  	if new.real_stime is not null and (new.real_offline_stime = old.real_offline_stime or new.real_offline_stime is null ) then
	  		new.real_offline_stime := new.real_stime;
	  	elsif new.real_offline_stime is not null and (new.real_stime = old.real_stime or new.real_stime is null) then 
	  		new.real_stime := new.real_offline_stime;
	  	end if;
	  	
	  	if new.real_etime is not null and (new.real_offline_etime = old.real_offline_etime or new.real_offline_etime is null) then
	  		new.real_offline_etime := new.real_etime;
	  	elsif new.real_offline_etime is not null and (new.real_etime = old.real_etime or new.real_etime is null) then 
	  		new.real_etime := new.real_offline_etime;
	  	end if;
	  end case;	
      RETURN NEW;
   END;
$BODY$ LANGUAGE plpgsql;
`
	return db.Exec(sql).Error
}

func CreateTrigger() error {
	sql := `
drop trigger update_real_stime_trigger on newtable1;

CREATE TRIGGER update_real_stime_trigger 
before INSERT or update of real_stime,real_etime,real_offline_stime,real_offline_etime
ON newtable1 
FOR EACH ROW 
EXECUTE PROCEDURE update_real_stime_func();
`
	db, err := GetStore()
	if err != nil {
		return err
	}
	return db.Exec(sql).Error
}

func GetStore() (db *gorm.DB, err error) {
	var (
		host, port, dbname, dbschema, user, pass string
	)

	host = "localhost"
	port = "5432"
	dbname = "postgres"
	dbschema = "public."
	user = "postgres"
	pass = "123123"

	dsn := fmt.Sprintf("host=%s port=%s user=%s dbname=%s password=%s sslmode=disable TimeZone=Asia/Shanghai",
		host,
		port,
		user,
		dbname,
		pass)
	db, err = gorm.Open(postgres.Open(dsn), &gorm.Config{
		//SkipDefaultTransaction:                   false,
		NamingStrategy: schema.NamingStrategy{
			TablePrefix:   dbschema,
			SingularTable: true,
			//NameReplacer:  nil,
			//NoLowerCase:   false,
		},
		//FullSaveAssociations:                     false,
		//Logger:                                   nil,
		//NowFunc:                                  nil,
		//DryRun:                                   false,
		//PrepareStmt:                              false,
		//DisableAutomaticPing:                     false,
		//DisableForeignKeyConstraintWhenMigrating: false,
		//DisableNestedTransaction:                 false,
		//AllowGlobalUpdate:                        false,
		//QueryFields:                              false,
		//CreateBatchSize:                          0,
		//ClauseBuilders:                           nil,
		//ConnPool:                                 nil,
		//Dialector:                                nil,
		//Plugins:                                  nil,
	})
	return
}
```
```go
//trigger_test.go
package db

import (
	"github.com/stretchr/testify/assert"
	"strings"
	"testing"
)

func TestCreateTrigger(t *testing.T) {

	err := CreateTriggerFunction()
	assert.NoError(t, err)

	err = CreateTrigger()
	assert.NoError(t, err)

	db, err := GetStore()
	assert.NoError(t, err)
	assert.NotNil(t, db)

	tests := []struct {
		name    string
		preSql  string //update usage
		sql     string
		params  []interface{}
		wantMap map[string]interface{}
	}{
		//insert
		{name: "insert all set", sql: `insert into newtable1(content , real_stime,real_etime, real_offline_stime,real_offline_etime)values('test' , '1','2', '3','4') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "1", "real_etime": "2", "real_offline_stime": "3", "real_offline_etime": "4"}},
		{name: "insert real_time set", sql: `insert into newtable1(content , real_stime)values('test' , '1') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "1", "real_etime": "", "real_offline_stime": "1", "real_offline_etime": ""}},
		{name: "insert real_etime set", sql: `insert into newtable1(content , real_etime)values('test' , '2') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "", "real_etime": "2", "real_offline_stime": "", "real_offline_etime": "2"}},
		{name: "insert real_offline_stime set", sql: `insert into newtable1(content , real_offline_stime)values('test' , '3') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "3", "real_etime": "", "real_offline_stime": "3", "real_offline_etime": ""}},
		{name: "insert real_offline_stime set", sql: `insert into newtable1(content , real_offline_etime)values('test' , '4') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "", "real_etime": "4", "real_offline_stime": "", "real_offline_etime": "4"}},
		{name: "insert real_*time set", sql: `insert into newtable1(content , real_stime,real_etime)values('test' , '1','2') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "1", "real_etime": "2", "real_offline_stime": "1", "real_offline_etime": "2"}},
		{name: "insert real_offline_*time set", sql: `insert into newtable1(content , real_offline_stime,real_offline_etime)values('test' , '3','4') returning *;`,
			wantMap: map[string]interface{}{"real_stime": "3", "real_etime": "4", "real_offline_stime": "3", "real_offline_etime": "4"}},

		//update
		{name: "update all set",
			preSql:  `insert into newtable1(content , real_stime,real_etime, real_offline_stime,real_offline_etime)values('test' , '1','2', '3','4') returning *;`,
			sql:     `update newtable1 set real_stime=?,real_etime=?,real_offline_stime=?,real_offline_etime=? where id=? returning *;`,
			params:  []interface{}{"a", "b", "c", "d", 0},
			wantMap: map[string]interface{}{"real_stime": "a", "real_etime": "b", "real_offline_stime": "c", "real_offline_etime": "d"}},

		{name: "update all set where all null",
			preSql:  `insert into newtable1(content )values('test') returning *;`,
			sql:     `update newtable1 set real_stime=?,real_etime=?,real_offline_stime=?,real_offline_etime=? where id=? returning *;`,
			params:  []interface{}{"a", "b", "c", "d", 0},
			wantMap: map[string]interface{}{"real_stime": "a", "real_etime": "b", "real_offline_stime": "c", "real_offline_etime": "d"}},

		{name: "update all set where real_*time is null",
			preSql:  `insert into newtable1(content ,real_stime , real_etime)values('test' , '1' , '2') returning *;`,
			sql:     `update newtable1 set real_stime=?,real_etime=? where id=? returning *;`,
			params:  []interface{}{"a", "b", 0},
			wantMap: map[string]interface{}{"real_stime": "a", "real_etime": "b", "real_offline_stime": "a", "real_offline_etime": "b"}},

		{name: "update all set where real_offline_*time is null",
			preSql:  `insert into newtable1(content ,real_offline_stime , real_offline_etime)values('test' , '3' , '4') returning *;`,
			sql:     `update newtable1 set real_offline_stime=?,real_offline_etime=? where id=? returning *;`,
			params:  []interface{}{"c", "d", 0},
			wantMap: map[string]interface{}{"real_stime": "c", "real_etime": "d", "real_offline_stime": "c", "real_offline_etime": "d"}},

		{name: "update all set where real_offline_*time is null",
			preSql:  `insert into newtable1(content , real_stime,real_etime, real_offline_stime,real_offline_etime)values('test' , '1','2', '3','4') returning *;`,
			sql:     `update newtable1 set real_stime=?,real_etime=? where id=? returning *;`,
			params:  []interface{}{"a", "b", 0},
			wantMap: map[string]interface{}{"real_stime": "a", "real_etime": "b", "real_offline_stime": "a", "real_offline_etime": "b"}},
	}
	for _, tt := range tests {
		t.Run(tt.name, func(t *testing.T) {
			var row Newtable1
			var preRow Newtable1
			if strings.Contains(tt.name, "insert") {
				//insert case
				if db.Raw(tt.sql).Scan(&row).Error != nil {
					t.Error(tt.name, err.Error(), "\n")
				}
			} else {
				//update case
				//presql
				if db.Raw(tt.preSql).Scan(&preRow).Error != nil {
					t.Error(tt.name, err.Error(), "\n")
				}
				tt.params[len(tt.params)-1] = preRow.ID
				if db.Raw(tt.sql, tt.params...).Scan(&row).Error != nil {
					t.Error(tt.name, err.Error(), "\n")
				}
			}
			assert.Equal(t, tt.wantMap["real_stime"], row.RealStime)
			assert.Equal(t, tt.wantMap["real_etime"], row.RealEtime)
			assert.Equal(t, tt.wantMap["real_offline_stime"], row.RealOfflineStime)
			assert.Equal(t, tt.wantMap["real_offline_etime"], row.RealOfflineEtime)
		})
	}
}

```
# 总结
在before触发器这里如果需要重新赋值，需要使用`NEW.column:=xxx`来进行赋值，这里使用了`RETURN NEW`是将这个新row返回给调用触发器的地方。\

参考：\
http://postgres.cn/docs/10/sql-createtrigger.html\
https://www.cnblogs.com/kuang17/p/13140977.html\
https://dba.stackexchange.com/questions/182678/postgresql-trigger-to-update-a-column-in-a-table-when-another-table-gets-inserte