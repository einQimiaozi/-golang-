## 准备

golang操作mysql需要用到一个自带的数据库和一个第三方mysql驱动

``` golang
import (
	"database/sql"
	_"github.com/go-sql-driver/mysql"
)
```

## 数据库结构

```
+-------+-------------+------+-----+---------+----------------+
| Field | Type        | Null | Key | Default | Extra          |
+-------+-------------+------+-----+---------+----------------+
| id    | bigint(20)  | NO   | PRI | NULL    | auto_increment |
| name  | varchar(45) | YES  |     |         |                |
| age   | int(11)     | YES  |     | NULL    |                |
| sex   | tinyint(3)  | YES  |     | NULL    |                |
| phone | varchar(45) | NO   |     |         |                |
+-------+-------------+------+-----+---------+----------------+
```

## 用于操作数据库的结构体

```golang
// 表结构
type user struct {
	id int `json:"id"`
	age int `json:"age"`
	name string `json:"name"`
	sex int `json:"sex"`
	phone string `json:"phone"`
}
// 操作数据库的对象
type Db struct {
	mysql *sql.DB
}
```

## 连接数据库

```golang
func (db *Db) initDatabase() {
	// test是库名 用户名密码和ip用你自己的即可
	con,err := sql.Open("mysql","用户名:密码@tcp(服务器ip:3306)/test")
	if err!=nil {
		panic(err)
	}else {
		fmt.Println("connect success!")
	}
	con.SetConnMaxLifetime(time.Minute*5)
	con.SetMaxOpenConns(100)
	con.SetMaxIdleConns(100)
	db.mysql = con
}
```

# 插入

```golang
func (db *Db) insert(tables string,values user) int64 {
	stmt,err := db.mysql.Prepare(fmt.Sprintf("Insert %s set age=?,name=?,sex=?,phone=?",tables))
	if err!=nil {
		panic(err)
	}
	res,err := stmt.Exec(values.age,values.name,values.sex,values.phone)
	if err != nil {
		panic(err)
	}
	// 执行结果 1=成功 2=失败
	ids,_ := res.LastInsertId()
	defer stmt.Close()
	return ids
}
```

## 删除

```golang
func (db *Db) delete(id int) int64 {
	stmt,err := db.mysql.Prepare("delete from user where id=?")
	defer stmt.Close()
	if err!=nil {
		panic(err)
	}
	res,err := stmt.Exec(id)
	if err!=nil {
		panic(err)
	}
	ids,_ := res.LastInsertId()
	return ids
}
```

ps：更新也一样，只不过sql改一下即可

## 单条查询

```golang
func (db *Db) querySingle(id int) (user){
	stmt,err := db.mysql.Prepare("select name,age,sex,phone from user where id=?")
	if err != nil {
		panic(err)
	}
	res := stmt.QueryRow(id)
	var u user
	// 将结果流入user对象中
	err = res.Scan(&u.name,&u.age,&u.sex,&u.phone)
	if err != nil {
		panic(err)
	}
	defer stmt.Close()
	return u
}
```

## 多条查询

```golang
func (db *Db) query(sex int) ([]user) {
	stmt,err := db.mysql.Prepare("select id,name,age,phone from user where sex=?")
	defer stmt.Close()
	if err!=nil {
		panic(err)
	}
	res,err := stmt.Query(sex)
	defer res.Close()
	if err!=nil {
		panic(err)
	}
	u := make([]user,1)
	for res.Next() {
		var newu user
		err = res.Scan(&newu.id,&newu.name,&newu.age,&newu.phone)
		if err!=nil {
			panic(err)
		}
		u = append(u,newu)
	}
	return u
}
```

## 事务

```golang
// 这里的方法我直接把参数写死了
func (db*Db) Txn() {
	// 设置隔离级别 
	txn := sql.TxOptions{sql.LevelRepeatableRead,false}
	// 事务开始
	tx,err := db.mysql.BeginTx(context.Background(),&txn)
	if err!=nil {
		panic(err)
	}
	// 执行事务1
	stmt1,err1 := tx.Prepare("delete from user where id=?")
	defer stmt1.Close()
	if err1!=nil {
		panic(err1)
	}
	stmt1.Exec(6)
	// 执行事务2
	stmt2,err2 := tx.Prepare("delete from user where id=?")
	defer stmt2.Close()
	if err2!=nil {
		panic(err2)
	}
	stmt1.Exec(4)
	
	// 提交or回滚，注意，提交和回滚不能一起使用
	
	//err3 := tx.Rollback()
	err3 := tx.Commit()
	if err3!=nil {
		panic(err3)
	}
}
```

## 关闭

```golang
func (db *Db) Close() {
	if db.mysql!=nil {
		db.mysql.Close()
	}
}
```
