## export table

#### MySQL table into CSV file
```sql
SELECT * FROM my_table
INTO OUTFILE 'my_table.csv'
CHARACTER SET euckr
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
ESCAPED BY '\\'
LINES TERMINATED BY '\n'
```

#### MySQL table into CSV file
```sql
SELECT * FROM (
    (
        SELECT
            '필드1' AS 'filed_1',
            '필드2' AS 'filed_2'
    ) UNION (
        SELECT
            filed_1,
            filed_2
        FROM my_table
    )
) AS mysql_query
INTO OUTFILE 'my_table.csv'
CHARACTER SET euckr
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
ESCAPED BY '\\'
LINES TERMINATED BY '\n'
```

#### MySQL table into CSV file
$ mysql -p my_db -e "SELECT * FROM my_table" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > my_table.csv  

#### MySQL table into CSV file
```sh
#!/bin/bash
 
db=YOUR_DB
user=YOUR_USER
pass=YOUR_PASS
 
for table in $(mysql -u$user -p$pass $db -Be "SHOW tables" | sed 1d); do
  echo "exporting $table.."
  mysql -u$user -p$pass $db -e "SELECT * FROM $table" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > $table.csv
done
```

+ gaerae/MySQL table into CSV file 1.sql  
https://gist.github.com/gaerae/6219678
