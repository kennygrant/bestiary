# Databases

If you're unsure which database to use, Postgresql is a highly reliable, well maintained project and has excellent drivers for Go. the examples shown below use psql, but can be adapted for other databases.  

### Opening a connection

Importing a driver has side effects - just the fact. In retrospect this is a bad design and there should be explicit registration of drivers, but because of the Go 1 promise we're stuck with it. 

### Opening a connection

When you open a connection to the database, you must ping it to ensure 

```go
// Open the database (no connection is miade)
db, err := sql.Open("postgres","postgres://pqgotest:password@localhost/pqgotest?sslmode=verify-full")
		"user:password@tcp(127.0.0.1:3306)/hello")
if err != nil {
    return err
}
	
// Check the db connection
err = db.Ping()
if err != nil {
    return err
}
```

### Closing a connection 

Don't close the connection frequently,  you should ideally create one connection per datastore which lasts for the lifetime of your application. For example you might call defer db.Close in main. 

```
defer db.Close()
```

### Querying the database

Perhaps unfortunately, 

### Reading into a struct

### Writing to the database



