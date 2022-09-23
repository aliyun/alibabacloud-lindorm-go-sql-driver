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

# Alibaba Cloud Lindorm SQL Driver

[![Go Reference](https://pkg.go.dev/badge/github.com/apache/calcite-avatica-go/v5.svg)](https://pkg.go.dev/github.com/apache/calcite-avatica-go/v5)

Alibaba Cloud Lindorm SQL Driver is a Go [database/sql](https://golang.org/pkg/database/sql/) driver forked from project [Apache Calcite's Avatica Go](https://github.com/apache/calcite-avatica-go).
It was adapted for the Alibaba Cloud Lindorm([CN](https://www.aliyun.com/product/apsaradb/lindorm), [EN](https://www.alibabacloud.com/product/lindorm)).

## Quick Start
Install using Go modules and Go 1.17+.

```
$ go get github.com/apache/calcite-avatica-go
```

Add the following dependency declaration to your `go.mod` file:

```
require github.com/apache/calcite-avatica-go/v5 v5.0.0

replace github.com/apache/calcite-avatica-go/v5 => github.com/aliyun/alibabacloud-lindorm-go-sql-driver/v5 v5.0.1
```

The funcationality of Alibaba Cloud Lindorm SQL Driver keeps the same as Phoenix/Avatica driver, which implements Go's `database/sql/driver` interface, 
so, import the `database/sql` package and the driver:

```
import "database/sql"
import _ "github.com/apache/calcite-avatica-go/v5"

lindormUrl := "http://ld-xxxxx.lindorm.rds.aliyuncs.com:30060"
conn := avatica.NewConnector(lindormUrl).(*avatica.Connector)
conn.Info = map[string]string{
    "user":     "usr",     // username
    "password": "psw",     // password
    "database": "db1",     // the database used by default
}

db := sql.OpenDB(conn)
```

You can refer to the following demo for more details about the usage of Alibaba Cloud Lindorm SQL Driver.
* [Demo](https://github.com/aliyun/aliyun-apsaradb-hbase-demo/blob/master/lindormsql-go/demo/demo.go)

