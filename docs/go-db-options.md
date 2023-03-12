## SQL DB options in golang

#### low-level standard library database/sql package.
1. very fast, and straight forward.
2. manual mapping SQL fields to variables.
3. errors only caught in runtime
```go
// Query the database for the information requested and prints the results.
// If the query fails exit the program with an error.
func Query(ctx context.Context, id int64) {
	ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
	defer cancel()

	var name string
	err := pool.QueryRowContext(ctx, "select p.name from people as p where p.id = :id;", sql.Named("id", id)).Scan(&name)
	if err != nil {
		log.Fatal("unable to execute search query", err)
	}
	log.Println("name=", name)
}
```

#### GORM
1. hight-level object-relational-mapping library
2. CRUD functions already implemented.
3. must use GORM provided functions.
4. 3-5 times slower than lower level SQL library.
```go
user:=&UserModel{Name:"John",Address:"New York"}
db.Create(user)

Internally it will create the query like
INSERT INTO `user_models` (`name`,`address`) VALUES ('John','New York')

//You can insert multiple records too
var users []UserModel = []UserModel{
    UserModel{name: "Ricky",Address:"Sydney"},
    UserModel{name: "Adam",Address:"Brisbane"},
    UserModel{name: "Justin",Address:"California"},
}

for _, user := range users {
    db.Create(&user)
}

user:=&UserModel{Name:"John",Address:"New York"}
// Select, edit, and save
db.Find(&user)
user.Address = "Brisbane"
db.Save(&user)

// Update with column names, not attribute names
db.Model(&user).Update("Name", "Jack")

db.Model(&user).Updates(
map[string]interface{}{
    "Name": "Amy",
    "Address": "Boston",
})

// UpdateColumn()
db.Model(&user).UpdateColumn("Address", "Phoenix")
db.Model(&user).UpdateColumns(
map[string]interface{}{
    "Name": "Taylor",
    "Address": "Houston",
})
// Using Find()
db.Find(&user).Update("Address", "San Diego")

// Batch Update
db.Table("user_models").Where("address = ?", "california").Update("name", "Walker")
```

#### SQLX
1. quite fast & easy to use
2. fields mapping via query txt & structure tags.
3. errors only caught in runtime
```go
schema := `CREATE TABLE place (
    country text,
    city text NULL,
    telcode integer);`
 
// execute a query on the server
result, err := db.Exec(schema)
 
// or, you can use MustExec, which panics on error
cityState := `INSERT INTO place (country, telcode) VALUES (?, ?)`
countryCity := `INSERT INTO place (country, city, telcode) VALUES (?, ?, ?)`
db.MustExec(cityState, "Hong Kong", 852)
db.MustExec(cityState, "Singapore", 65)
db.MustExec(countryCity, "South Africa", "Johannesburg", 27)

// fetch all places from the db
rows, err := db.Query("SELECT country, city, telcode FROM place")

// iterate over each row
for rows.Next() {
var country string
// note that city can be NULL, so we use the NullString type
var city    sql.NullString
var telcode int
err = rows.Scan(&country, &city, &telcode)
}
// check the error from rows
err = rows.Err()
```

#### SQLC
1. very fast & easy to use
2. automatic code generation
3. catch SQL query errors before generating codes
4. full support Postgres, but MySQL is experimental
```sql
CREATE TABLE authors (
  id   BIGSERIAL PRIMARY KEY,
  name text      NOT NULL,
  bio  text
);

-- name: GetAuthor :one
SELECT * FROM authors
WHERE id = $1 LIMIT 1;
```
SQLC will auto generate the following golang codes.
```go
package db

import (
  "context"
  "database/sql"
)

type Author struct {
  ID   int64
  Name string
  Bio  sql.NullString
}

const getAuthor = `-- name: GetAuthor :one
SELECT id, name, bio FROM authors
WHERE id = $1 LIMIT 1
`

type Queries struct {
  db *sql.DB
}

func (q *Queries) GetAuthor(ctx context.Context, id int64) (Author, error) {
  row := q.db.QueryRowContext(ctx, getAuthor, id)
  var i Author
  err := row.Scan(&i.ID, &i.Name, &i.Bio)
  return i, err
}
```
