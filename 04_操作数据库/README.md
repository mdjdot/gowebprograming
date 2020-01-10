# database/sql
	• database/sql包定义了对数据库的一系列错误
	• database/sql/driver包定义了需要被数据库驱动实现的接口

# 获取数据库连接
```
database.sql.Open("driverName","dataSourceName")
driverName："mysql"
databaseSourceName："user:pwd@tcp(localhost:3306)/dbname"
```
返回的DB可以安全的被多个go协程同时使用，并会维护自身的闲置链接池。Open函数只需要调用一次，很少需要关闭DB
```
package db
import "database/sql"
import _"github.com/go-sql-driver/mysql"
var (
    SQLDb *sql.DB
    err   error
)
func init() {
    sql.Register()
    SQLDb, err = sql.Open("mysql", "test:windtest@tcp(localhost:3306)/wordpress")
    if err != nil {
        panic(err)
    }
}

package main
import "github.com/mdjdot/gocoreprograming/dbs/db"
func main() {
    err := db.SQLDb.Ping()
    if err != nil {
        panic(err)
    }
}
```

# 增删改查
```
create table users(
    id int primary key auto_increment,
   username varchar(100) unique not null,
    password varchar(100) not null,
   email varchar(100)
    );

package db
import "database/sql"
import _ "github.com/go-sql-driver/mysql"
var (
    SQLDb *sql.DB
    err   error
)
func init() {
    SQLDb, err = sql.Open("mysql", "test:windtest@tcp(localhost:3306)/test")
    if err != nil {
        panic(err)
    }
}

package model
import "github.com/mdjdot/gocoreprograming/dbs/db"
import "fmt"
type User struct {
    ID       int
    UserName string
    Password string
    Email    string
}
func (u *User) AddUser(userName, password, email string) error {
    sqlStr := "insert into users(username,password,email) values(?,?,?)"
    // 预编译
    stmt, err := db.SQLDb.Prepare(sqlStr)
    if err != nil {
        return err
    }
    result, err := stmt.Exec(userName, password, email)
    if err != nil {
        return err
    }
    fmt.Println(result)
    return nil
}
func (u *User) ListUser() error {
    var id int
    var username, password, email string
    rows, err := db.SQLDb.Query("select id,username,password,email from users")
    if err != nil {
        return err
    }
    defer rows.Close()
    for rows.Next() {
        rows.Scan(&id, &username, &password, &email)
        fmt.Println(id, username, password, email)
    }
    fmt.Println(rows.Err())
    return nil
}
func (u *User) DelUser(id int) error {
    stmt, err := db.SQLDb.Prepare("delete from users where id=?")
    if err != nil {
        return err
    }
    result, err := stmt.Exec(id)
    if err != nil {
        return err
    }
    fmt.Println(result)
    return nil
}

package main
import "github.com/mdjdot/gocoreprograming/dbs/db"
import "github.com/mdjdot/gocoreprograming/dbs/model"
func main() {
    err := db.SQLDb.Ping()
    if err != nil {
        panic(err)
    }
    err = (&model.User{}).AddUser("maliu", "maliu", "maliu@wu.com")
    if err != nil {
        panic(err)
    }
    err = (&model.User{}).ListUser()
    if err != nil {
        panic(err)
    }
    err = (&model.User{}).DelUser(5)
    if err != nil {
        panic(err)
    }
}
```

# 单元测试
```
user_test.go
package model
import (
    "fmt"
    "os"
    "testing"
    "github.com/stretchr/testify/assert"
)
func TestMain(m *testing.M) {
    fmt.Println("开始测试")
    os.Exit(m.Run())
}
func TestUser_ListUser(t *testing.T) {
    err := (&User{}).ListUser()
    assert.NoError(t, err)
}
func ttestUser_ListUser(t *testing.T) {
    err := (&User{}).ListUser()
    assert.NoError(t, err)
}
func Test_Runtest(t *testing.T) {
    t.Run("执行非大写Test开头的用例", ttestUser_ListUser)
}
```

go test github.com/mdjdot/gocoreprograming/dbs/model -v  
可以在TestMain中的m.Run之前setup，m.run之后teardown
