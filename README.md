<!--
{% comment %}
Licensed to the Apache Software Foundation (ASF) under one or more
contributor license agreements.  See the NOTICE file distributed with
this work for additional information regarding copyright ownership.
The ASF licenses this file to you under the Apache License, Version 2.0
(the "License"); you may not use this file except in compliance with
the License.  You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
{% endcomment %}
-->

本文介绍通过 GoLang接入Lindorm的最佳实践和使用示例。

## 前提条件

- 已安装Go环境，建议安装go 1.17及以上版本。
- 已获取Lindorm宽表SQL的连接地址并配置白名单，具体操作，请参见[访问实例](https://help.aliyun.com/document_detail/264919.htm#concept-2090785)。

## 准备工作
在您的Go应用程序的`go.mod`文件中添加依赖
```java
require github.com/apache/calcite-avatica-go/v5 v5.0.0

replace github.com/apache/calcite-avatica-go/v5 => github.com/aliyun/alibabacloud-lindorm-go-sql-driver/v5 v5.0.1
```

## 操作步骤

1. 在代码中添加数据库驱动的依赖
```go
import (
		avatica "github.com/apache/calcite-avatica-go/v5"
)
```

2. 初始化连接池， 并配置连接池参数
```go
databaseUrl := "http://localhost:30060" // 这里的链接地址与lindorm-cli的链接地址比，需要去掉http之前的字符串

conn := avatica.NewConnector(databaseUrl).(*avatica.Connector)
conn.Info = map[string]string{
	"user":     "sql",     // 数据库用户名
	"password": "test",    // 数据库密码
	"database": "default", // 初始化连接指定的默认database
}

db := sql.OpenDB(conn)
// 设置连接池参数
// 连接最大空闲时间， 可以根据实际情况调整
db.SetConnMaxIdleTime(8 * time.Minute)
// 连接池中允许的最大连接数， 可以根据实际情况调整
db.SetMaxOpenConns(20)
// 连接池中允许的最大空闲连接数量, 可以根据实际情况调整
db.SetMaxIdleConns(2)
```

3. 获取链接并进行普通CURD操作

```go
// 创建表
_, err := db.Exec("create table if not exists user_test(id int, name varchar,age int, primary key(id))")
if err != nil {
	fmt.Println("create table error ", err)
	return
}

// 写入数据
_, err = db.Exec("upsert into user_test(id,name,age) values(1,'zhangsan',17)")
if err != nil {
	fmt.Println("insert data error", err)
	return
}

// 查询数据
rows, err := db.Query("select * from user_test")
if err != nil {
	fmt.Println("query data error", err)
	return
}
defer rows.Close()
	var id int
	var name string
	var age int
	for rows.Next() {
		err = rows.Scan(&id, &name, &age)
		if err != nil {
			fmt.Println("scan data error", err)
			return
		}
		fmt.Println("id:", id, "name:", name, "age:", age)
	}

//删除数据
_, err = db.Exec("delete from user_test where id=1")
if err != nil {
	fmt.Println("delete data error", err)
	return
}

```
说明：完整的使用示例，您可以参考[示例Demo](https://github.com/aliyun/aliyun-apsaradb-hbase-demo/blob/master/lindormsql-go/demo/demo.go)

4. 通过绑定参数进行写入和查询
```go
// 使用绑定参数进行写入
stmt, err := db.Prepare("upsert into user_test(id,name,age) values(?,?,?)")
if err != nil {
	fmt.Println("prepare error", err)
	return
}
_, err = stmt.Exec(1, "zhangsan", 17)
if err != nil {
	fmt.Println("upsert error", err)
	return
}

// 使用绑定参数进行查询
stmt, err = db.Prepare("select * from user_test where id=?")
if err != nil {
	fmt.Println("prepare error", err)
	return
}
rows, err := stmt.Query(1)
if err != nil {
	fmt.Println("query data error", err)
	return
}
defer rows.Close()
	var id int
	var name string
	var age int
	for rows.Next() {
		err = rows.Scan(&id, &name, &age)
		if err != nil {
			fmt.Println("scan data error", err)
			return
		}
		fmt.Println("id:", id, "name:", name, "age:", age)
	}
```
说明：完整的使用示例，您可以参考[示例Demo](https://github.com/aliyun/aliyun-apsaradb-hbase-demo/blob/master/lindormsql-go/prepare_demo/prepare_demo.go)

