```go
	db, err := sql.Open("mysql", "root:xwt123456789@tcp(172.17.0.2:3306)/shortestpath")　
	// 还没有connection呢
	err = db.Ping() // ping　下数据库　有个connection 看连接是好的不
    tx, err := db.Begin()
	stmt, err := tx.Prepare("insert into record values(?,?,?)") 
	tx.Commit() // tx end 
```

```go
s := sql.NullString 
rows.Scan(&s)
if s.Valid{
    // not null handle 
}
```

