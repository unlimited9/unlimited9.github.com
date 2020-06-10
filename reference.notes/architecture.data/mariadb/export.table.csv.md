## export table

#### 1. query
```sql
SELECT * FROM my_table
INTO OUTFILE 'my_table.csv'
CHARACTER SET euckr
FIELDS TERMINATED BY ',' OPTIONALLY ENCLOSED BY '"'
ESCAPED BY '\\'
LINES TERMINATED BY '\n'
```

#### 2. command
$ mysql -p my_db -e "SELECT * FROM my_table" | sed 's/\t/","/g;s/^/"/;s/$/"/;' > my_table.csv  

#### 3. script
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

## 9. Appendix

#### reference site

+ gaerae/MySQL table into CSV file 1.sql  
https://gist.github.com/gaerae/6219678
