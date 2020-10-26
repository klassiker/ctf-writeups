# Leak audit

## Task

We found an old dump of our employee database on the dark net! Please check the database and send us the requested information:

 - How many employee records are in the file?
 - Are there any employees that use the same password? (If true, send us the password for further investigation.)
 - In 2017, we switched to bcrypt to securely store the passwords. How many records are protected with bcrypt?

Flag format: answer1_answer2_answer3 (e.g., 1000_passw0rd_987).

File: BB-inDu57rY-P0W3R-L34k3r2.tar.gz

Tags: sql

## Solution

First we need to extract the file:

```bash
$ tar xfv BB-inDu57rY-P0W3R-L34k3r2.tar.gz
```

We now have a `BB-inDu57rY-P0W3R-L34k3r2.db` file:

```bash
$ file BB-inDu57rY-P0W3R-L34k3r2.db
BB-inDu57rY-P0W3R-L34k3r2.db: SQLite 3.x database, last written using SQLite version 3033000
```

We can use the `sqlitebrowser` or the command line utility `sqlite3` to browse the data.

```bash
$ sqlite3 BB-inDu57rY-P0W3R-L34k3r2.db
sqlite> .tables
personal
sqlite> .schema personal
CREATE TABLE personal (
  number INTEGER PRIMARY KEY AUTOINCREMENT,
  surname varchar(23) NOT NULL,
  givenname varchar(20) NOT NULL,
  streetaddress varchar(100) NOT NULL,
  city varchar(100) NOT NULL,
  zipcode varchar(15) NOT NULL,
  password varchar(25) NOT NULL,
  birthday varchar(10) NOT NULL
);
```

First part: How many employee records are in the file?

```bash
sqlite> SELECT COUNT(*) FROM personal;
376
```

Second part: Are there any employees that use the same password?

```bash
sqlite> SELECT password FROM personal GROUP BY password HAVING COUNT(*) > 1;
mah6geiVoo
```

Third part: How many records are protected with bcrypt?

```bash
sqlite> SELECT COUNT(*) FROM personal WHERE password LIKE '$2b$%';
21
```
