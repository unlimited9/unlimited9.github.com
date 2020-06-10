# LOAD DATA INFILE

#### LOAD CSV FILE DATA

`USAGE`  
```
LOAD DATA INFILE '{file_name}'

    INTO TABLE {table_name}
    CHARACTER SET utf8
    FIELDS
       TERMINATED BY '{field_terminator}'  # 각 필드 구분 문자 (예: CSV라면 컴마)
        OPTIONALLY ENCLOSED BY '"'  # 필요할 경우, 따옴표(")로 구분
    LINE TERMINATED BY '\n'
    IGNORE 1 LINES  # 제목이 포함된 첫 번째 줄은 생략
    (col1, col2, ... )  # 컬럼명
```

`EXAMPLE`  
```
LOAD DATA INFILE '/pure/adtruth/workspace/adtruth_log1.csv'
REPLACE INTO TABLE `dreamsearch`.`ADTRUTH_LOG` COLUMNS TERMINATED BY ',' ENCLOSED BY '"'
LINES TERMINATED BY '\n'
(@STARTDTH, @AUID, @IP, @ADTRUTH_ID,@USER_AGENT)
SET `STARTDTH` = @STARTDTH, `AUID` = @AUID, `IP` = @IP, `ADTRUTH_ID` = @ADTRUTH_ID, `USER_AGENT` = @USER_AGENT;
```

#### LOAD LOCAL CSV FILE DATA
```
LOAD DATA LOCAL INFILE 'file_name'
    [REPLACE | IGNORE]
    INTO TABLE tbl_name
    [CHARACTER SET charset_name]
    [{FIELDS | COLUMNS}
        [TERMINATED BY 'string']
        [[OPTIONALLY] ENCLOSED BY 'char']
        [ESCAPED BY 'char']
    ]
    [LINES
        [STARTING BY 'string']
        [TERMINATED BY 'string']
    ]
    [IGNORE number LINES]
    [(col_name_or_user_var,...)]
    [SET col_name = expr,...]
```

...

## 9. Appendix

#### reference site

+ [MySQL || MariaDB] 데이터 파일 IMPORT 하기(Feat.LOAD DATA INFILE)  
https://java119.tistory.com/55  
