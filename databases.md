# Databases

If you're unsure which database to use, Postgresql is a highly reliable, well maintained project and has excellent drivers for Go. the examples shown below use psql, but can be adapted for other databases. Go has drivers for all the mainstream databases, as well as a selection of time series and key/value stores written in Go.

You can find a list of go database drivers on the Go wiki page [SQL Drivers](https://github.com/golang/go/wiki/SQLDrivers) - prefer a driver which doesn't use cgo and conforms to the [database/sql](https://golang.org/pkg/database/sql/) interface if possible.

## Importing a driver

Importing a database driver initialises the database. In retrospect this is an unfortunate design decision, and it would be better to register drivers explicitly, but it won't change in Go 1. So import the driver as follows in order to use a given database:

```go
import (
  "database/sql"
  _ "github.com/lib/pq" 
)
```

has side effects - just the fact. In retrospect this is a bad design and there should be explicit registration of drivers, but because of the Go 1 promise we're stuck with it.

If you import lots of drivers, you will be importing all the code they use, and registering the driver, so only import those you need to use in your program.

## Opening a connection

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

## Closing a connection

Don't close the connection to the database frequently as it is designed to be recycled,  you should ideally create one connection per datastore which lasts for the lifetime of your application. For example you might call defer db.Close in main.

```
defer db.Close()
```

## Querying the database

Querying the database can be driver specific, especially for more advanced features, so be sure to read the driver documentation for the particular database/sql flavour you're using.

### Parameter placeholders

The different databases use different formats for parameter placeholders, for example they don't all support ? as you might expect. The Mysql driver uses ?, ? etc, the sqlite driver uses ?, ? etc , the Postgresql driver uses $1, $2 etc and the Oracle driver uses :val1, :val2 etc. Sqlx is one of the few drivers to support named query parameters.

### Reading values

```go
// Select from the db
sql := "select count from users where status=100"
rows, err := db.Query(sql)
if err != nil {
    return err
}
defer rows.Close()

// Read the count (just one row and one col)
var count int 
for rows.Next() {
   err = rows.Scan(&count)
   if err != nil {
     return err
   }
}
```

### Reading a Row into a struct

To read a single row into a struct, you can use Query Row. You may want to add a constructor to your models which takes a row and instantiates the model from it, but the code below is a minimal example.

```go
// Query a row from the database
row := db.QueryRow("select id,name,created_at from tablename where id=?", id)

// Read into a struct 
type Resource struct{
    ID int
    Name string
    CreatedAt time.Time
}

// This assumes no values are nil in the database
var r Resource
err := row.Scan(&r.ID, &r.Name, &r.CreatedAt)
if err != nil {
    return err
}
```

### Scanning into a struct

Many database libraries use reflect to attempt to introspect struct fields. You should try to avoid using reflect if you can as it is slow, prone to panics, and requires you to use struct tags and a dsl invented by the database library author. Another approach is to generate a list of columns, and pass them to the struct itself to assign, since it knows all about its fields and which values should go into them.

```go
// Read into a struct 
type User struct {
    ID int
    Name string
    CreatedAt time.Time
}

// ReadUser fills in a user from database columns
// the values are validated to not be null before assignment. 
func ReadUser(columns map[string]interface{}) *User {
    u := &User{}
    u.ID = validateInt(cols["id"])
    u.Name = validateString(cols["name"])
    u.CreatedAt = validateTime(cols["created_at"])
    return u
}
```

## Handling Relations

This can seem a bit more fiddly than other languages, however I think the best approach is a straightforward one - retrieve the indexes of relations, and then if you require all of their information, retrieve the relations separately from the database. You can of course retrieve them with a join at the same time, but in complex apps it helps to separate retrieving relations from retrieving the actual relation records, which is not always necessary \(for example you might need to know a user has 8 images, and which ids they have, but not all the image captions and image data\).

## Connections are recycled

Your connections may be called from many goroutines and connections are pooled by the driver. So you shouldn't use stateful sql commands like USE, BEGIN or COMMIT, LOCK TABLES etc and instead use the facilities offered by the sql driver. There’s no default limit on the number of connections, so it is possible to exhaust the number of connections allowed by your database. You can use  SetMaxOpenConns and SetMaxIdleConns to control this behaviour.

```
db.SetMaxIdleConns(10)
db.SetMaxOpenConns(100)
```

## Null values

If your database may contain null values, you should guard against them. One way to do this is to scan into a map of empty interface and then assert that the interface{} contains the type you \(as opposed to a special null value\). If it fails, use the zero value of the type instead.

## Getting Database columns

If you wish to know which columns are in a row, you can call:

```go
// Get a list of column names for the row
cols, err := rows.Columns()
```

This might be useful if you vary selects \(e.g. sometimes select just id and created at, sometimes select the full model\), and yet wish to always use a standard constructor for models which can be passed a map\[string\]interface{} with the values.

## Writing to the database

```go
// Insert a row
row := db.QueryRow("insert into tablename VALUES($1,$2,$3)", args...)
// Retrieve the inserted id
err = row.Scan(&id)
// Return the id and any error
return id, err
```

## Multiple Statements

The database/sql  doesn't specify that drivers should support multiple statements, which means the behaviour is undefined and it's probably best to send single statements, unless your driver explicitly supports it.

