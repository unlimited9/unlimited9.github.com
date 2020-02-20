# 10.install.n setup : logstash

>Created 목요일 30 11월 2017
데이터의 수집, 변환, 확장 및 출력. : Logstash

# Installation/Setup/Configuration

## 1. installation setup : app

### A. change account

$ su - app

### B. creating application directory

#### make directory
$ mkdir -p /apps/logstash
$ mkdir -p /data/logstash
$ mkdir -p /logs/logstash

### C. download

#### Elastic(https://www.elastic.co/downloads)
$ cd /apps/install
$ curl -O https://artifacts.elastic.co/downloads/logstash/logstash-7.1.1.tar.gz
~~$ wget https://artifacts.elastic.co/downloads/logstash/logstash-7.1.1.tar.gz -P /apps/install~~

### D. install

#### decompress tarball
$ tar -zxvf /apps/install/logstash-7.1.1.tar.gz

#### copy application directory
$ cp -R /apps/install/logstash-7.1.1 /apps/logstash/7.1.1	

### E. configure
$ mkdir -p /apps/logstash/instances/config  
$ mkdir -p /apps/logstash/instances/lib

$ vi /apps/logstash/instances/config/logstash-product.conf
```
input {
	jdbc {
		jdbc_driver_library => "/storage/apps/logstash/instances/lib/mysql-connector-java-5.1.44-bin.jar"
		jdbc_driver_class => "com.mysql.jdbc.Driver"
		jdbc_connection_string => "jdbc:mysql://14.0.92.163:3306/dreamsearch?autoReconnect=true"
		jdbc_user => "dreamsearch"
		jdbc_password => "dnjvld$%^"
		
		#parameters => { "param_01" => "param" }
		#schedule => "* * * * *"
		statement => "
			SELECT 'shop_data' as type,
			pcode as no, pnm as title, price as price, userid as userid,
			imgpath as img_url, purl as link_url,
			cate1 as category,
			unix_timestamp() as created
			FROM SHOP_DATA"
		charset => "UTF-8"
		
		jdbc_fetch_size => 100000
		jdbc_paging_enabled => true
		jdbc_page_size => 1000000
	}
}

filter {
}

output {
	stdout {
		codec => rubydebug
	}
	
	elasticsearch {
		"hosts" => "10.10.10.25:6530"
		"index"	=> "product"
	}
}
```

### E. excute

#### start process
>run logstash and specify the configuration file with the -f flag  

$ /apps/logstash/7.1.1/bin/logstash -f /apps/logstash/instances/config/logstash-product.conf
