# self-signed certificate

## 1. openssl - generate SSL (self-signed) certificate

#### generate certificate
$ mkdir -p /apps/certs  
$ openssl req \  
  -newkey rsa:4096 -nodes -sha256 -keyout /apps/certs/server.key \  
  -x509 -days 36500 -out /apps/certs/server.crt
```
Generating a 4096 bit RSA private key
.................................++
...........................................................++
writing new private key to '/apps/certs/server.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:KR
State or Province Name (full name) []:SEOUL
Locality Name (eg, city) [Default City]:SEOUL
Organization Name (eg, company) [Default Company Ltd]:hajimaro
Organizational Unit Name (eg, section) []:dev
Common Name (eg, your name or your server's hostname) []:*.hajimaro.com
Email Address []:admin@hajimaro.com
```

$ ls /apps/certs
```
server.crt  server.key
```

>$ openssl genrsa -des3 -out /apps/certs/server.key 2048  
>$ openssl req -new -key /apps/certs/server.key -out /apps/certs/server.csr  
>$ openssl x509 -req -days 365 -in /apps/certs/server.csr -signkey /apps/certs/server.key -out /apps/certs/server.crt  
>$ cp /apps/certs/server.key /apps/certs/server.key.origin  
>$ openssl rsa -in /apps/certs/server.key.origin -out /apps/certs/server.key

#### update ca
* ubuntu  
  $ cp /apps/certs/server.crt /usr/share/ca-certificates/  
  $ echo /apps/certs/server.crt >> /etc/ca-certificates.conf  
  $ update-ca-certificates

* centos  
  $ cp /apps/certs/server.crt /etc/pki/ca-trust/source/anchors/  
  $ update-ca-trust

* mac  
  $ security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /apps/certs/server.crt

## 9. Appendix

#### reference site

* 쿠버네티스 #6 - 실제 서비스 배포해보기  
http://j30231.tistory.com/21  
http://j30231.tistory.com/22






---


**********************************************************
1. openssl - generate SSL (self-signed) certificate
**********************************************************
	A. set openssl configuration
		>> ssl-base-dir
			redhat	: /etc/pki/tls/openssl.cnf
			debian	: /etc/ssl/openssl.cnf
		>> option
			[ CA_default ]
			dir				: CA의 관리 파일들이 저장될 디렉토리를 나타낸다. (./demoCA or /etc/pki/CA가 기본 값으로 되어있음.)
			>> #dir            = /etc/pki/CA           # Where everything is kept
				dir             = /repository/pki/CA           # Where everything is kept
			certs			: 발급된 인증서들을 넣어 놓는 디렉토리이다.
			crl				: 발급된 인증서 폐기 목록을 넣는 디렉토리이다.
			database		: 인증서의 발급과 관련된 내용을 저장하는 txt 형태의 파일이다.
			newcerts		: openssl ca 명령으로 사용자들의 인증서를 발급할때 인증서가 저장되는 디렉토리이다.
			certificate	: CA 자신의 인증서 파일의 위치이다,
			>> #certificate    = $dir/cacert.pem       # The CA certificate
				certificate     = $dir/ca.crt           # The CA certificate
			serial			: 현재까지 발급된 인증들의 시리얼 넘버이다. (1씩 증가함)
			crlnumber		: 현재까지 발급된 CRL 시리얼 넘버이다.
			private_key	: CA 자신의 개인키 파일의 위치이다.
 			>> #private_key    = $dir/private/cakey.pem# The private key
				private_key     = $dir/private/ca.key   # The private key
			default_days	: 인증서 발급시 기본 유효기간이다.(365 일 = 1년)

			[ policy_match ]	: 인증서를 발급할 때 어떤 정책을 가져갈지에 대한 부분
			countryName, stateOrProvinceName, organizationName
			>> policy value
				match		: CA와 동일하게 설정해야만 인증서 발급이 가능하다.
				optional	: 있어도, 없어도 발급이 가능하다.
				supplied	: 반드시 있어야 발급이 가능하다.

	B. create directory
		# create directory
		$ mkdir -p /repository/pki/CA↙
		$ mkdir /repository/pki/CA/certs↙
		$ mkdir /repository/pki/CA/newcerts↙
		$ mkdir /repository/pki/CA/crl↙
		$ mkdir /repository/pki/CA/private↙
		$ touch /repository/pki/CA/serial↙
		$ echo "00" > /repository/pki/CA/serial↙
		$ touch /repository/pki/CA/index.txt↙

	C. CA(Certificate Authority) certificates
		$ cd /repository/pki/CA↙

		1) generating random state file
			$ openssl md5 * > rand.dat↙
			>> error message
				...
				140170720470856:error:0200B015:system library:fread:Is a directory:bss_file.c:202:
				140170720470856:error:20082002:BIO routines:FILE_READ:system lib:bss_file.c:203:
				...

				# directory를 읽어오는데서 error가 발생했다.
				>> directory가 없는 곳에 가서 실행하거나 파일들만 가지고 생성한다.
					$ openssl md5 index.txt serial > rand.dat↙

		2) generating private key
			$ openssl genrsa -rand rand.dat -des3 -out private/ca.key 1024↙
			>> option
				-rand file:file:...		: load the file (or the files in the directory) into the random number generator
				-des3						: encrypt the generated key with DES in ede cbc mode (168 bit key)
				1024						: numbits (1024 bit key 생성)
				-out file					: output the key to file
			cf.Random Sate 사용 여부 및 암호화는 필수가 아닌 선택사항이다.
				key file을 암호화할 경우 패스워드 입력이 필요하며, 해당 패스워드를 알아야만 키파일을 사용할 수 있다. (패스워드는 암복화에 사용된다.)
				output file을 지정하는 대신 명령어 끝에 '> ca.key'로 사용할 수 있다.
			
			>> read private key
				$ openssl rsa -noout -text -in private/ca.key↙

 		3) generating CSR(Certificate Signing Request)
			$ openssl req -new -key private/ca.key -out ca.csr↙
			Enter pass phrase for private/ca.key:
			You are about to be asked to enter information that will be incorporated
			into your certificate request.
			What you are about to enter is what is called a Distinguished Name or a DN.
			There are quite a few fields but you can leave some blank
			For some fields there will be a default value,
			If you enter '.', the field will be left blank.
			-----
			Country Name (2 letter code) [XX]:KR
			State or Province Name (full name) []:Seoul
			Locality Name (eg, city) [Default City]:Seoul
			Organization Name (eg, company) [Default Company Ltd]:hajimaro
			Organizational Unit Name (eg, section) []:hajimaro
			Common Name (eg, your name or your server's hostname) []:hajimaro.com
			Email Address []:admin@hajimaro.com
			
			Please enter the following 'extra' attributes
			to be sent with your certificate request
			A challenge password []:
			An optional company name []:

			# caution : SSL/TLS 용 인증서의 CN(Common Name)은 접속 도메인 명과 같아야 한다.
			# domain이 여러개라면 "*.hajimaro.com"과 같이 지정할 수 있다.

			>> read csr
				$ openssl req -noout -text -in ca.csr↙

		4) generating self-signed CRT(Certificate)
			$ openssl req -key private/ca.key -x509 -nodes -sha1 -days 1095 -in ca.csr -out ca.crt↙
			>> or
				$ openssl ca -keyfile private/ca.key -selfsign -in ca.csr -out ca.crt -outdir .↙
			>> or
				$ openssl x509 -req -key private/ca.key -sha1 -days 1095 -in ca.csr -out ca.crt↙
			>> option
				-nodes		: don't encrypt the output key
				-sha1		: to use the sha1 message digest algorithm
				-days		: number of days a certificate generated by -x509 is valid for.
			
			>> read crt
				$ openssl x509 -in ca.crt -text↙

		cf. remove private key password
			$ openssl rsa -in ca.key -out ca.key.pem↙
			>> SSL 인증서의 private key에 비밀번호가 걸려있을 경우 이를 활용하는 어플리케이션을 실행할 때 개인키에 걸려있는 비밀번호를 입력해야만 한다.
				이는 운영/관리 관점에서 여러가지 이유로 어렵거나 번거로운 일일 수 있다. 이런 경우 private key에 걸린 비밀번호를 제거할 수 있다. ref.1.D.4)
				실제 application에서는 sub-certificates를 사용할 것이므로 ca.key의 비밀번호는 유지하자.
		cf.add private key password
			$ openssl rsa -in out server.key -des3 -out server.key.pem↙

		cf.
			>> generating private key and CSR(Certificate Signing Request)
				$ openssl req -newkey rsa:1024 -sha1 -keyout cakey.pem -out cacsr.pem↙
			>> generating self-signed CRT(Certificate)
				openssl req -x509 -sha1 -extensions v3_ca -signkey cakey.pem -in cacsr.pem -out cacert.crt -days 365↙

	D. sub certificates
		1) generating server private key (without password : except '-des3' option)
			$ openssl genrsa -des3 -out certs/hajimaro.com.key 1024↙
		
		2) generating server CSR(Certificate Signing Request)
			$ openssl req -new -key certs/hajimaro.com.key -out certs/hajimaro.com.csr↙
			Enter pass phrase for certs/hajimaro.com.key:
			You are about to be asked to enter information that will be incorporated
			into your certificate request.
			What you are about to enter is what is called a Distinguished Name or a DN.
			There are quite a few fields but you can leave some blank
			For some fields there will be a default value,
			If you enter '.', the field will be left blank.
			-----
			Country Name (2 letter code) [XX]:KR
			State or Province Name (full name) []:Seoul
			Locality Name (eg, city) [Default City]:Seoul
			Organization Name (eg, company) [Default Company Ltd]:hajimaro
			Organizational Unit Name (eg, section) []:hajimaro
			Common Name (eg, your name or your server's hostname) []:*.hajimaro.com
			Email Address []:admin@hajimaro.com
			
			Please enter the following 'extra' attributes
			to be sent with your certificate request
			A challenge password []:
			An optional company name []:
		
		3) generating server CRT(Certificate)
			$ openssl ca -keyfile private/ca.key -cert ca.crt -in certs/hajimaro.com.csr -out certs/hajimaro.com.crt -outdir ./certs↙
			>> or
				$ openssl x509 -req -signkey certs/hajimaro.com.key -sha1 -days 365 -in certs/hajimaro.com.csr -out certs/hajimaro.com.crt -CAkey private/ca.key -CA ca.crt -CAcreateserial↙

			>> error message
				failed to update database
				TXT_DB error number 2
				>> database update
					$ openssl ca -updatedb↙

		4) removing private key password
			$ cp certs/hajimaro.com.key certs/hajimaro.com.secure.key↙
			$ openssl rsa -in certs/hajimaro.com.key -out certs/hajimaro.com.key↙

	Z. Other
		>> convert crt to der
			$ openssl x509 -in ca.crt -out ca.der -outform DER↙
		
		>> convert pem to der
			$ openssl x509 -in ca.pem -out ca.der -outform DER↙
		
		>> convert der to pem
			$ openssl x509 -in ca.der -inform DER -out ca.pem -outform PEM↙
		
		>> print certificate
			$ openssl x509 -noout -text -in ca.key↙
		
		>> convert to pfx
			$ openssl pkcs12 -export in ca.crt -inkey ca.key -certfile ca.crt -out ca.pfx↙
		
		>> extract key from pfx
			$ openssl pkcs12 -in ca.pfx -nocerts -out ca.pem↙
		
		>> extract certificate from pfx
			$ openssl pkcs12 -in ca.pfx -clcerts -nokeys -out ca.pem↙
		
		>> excute radiuse
			$ /usr/local/sbin/radiuse -x↙

**********************************************************
2. keytool - generate SSL (self-signed) certificate
**********************************************************
	A. create directory
		# create directory
		$ mkdir -p /opt/pki/keystore↙

	B. generate SSL (self-signed) certificate
		1) generate a CRT(certificate) and private key pair (RSA)
			$ keytool -genkey -alias kdc -keypass quftkxkd -keystore /opt/pki/java/keystore/cacerts -keyalg RSA↙
			키 저장소 비밀번호 입력:  
			새 비밀번호 다시 입력: 
			이름과 성을 입력하십시오.
			  [Unknown]:  kdc.hajimaro.com
			조직 단위 이름을 입력하십시오.
			  [Unknown]:  architect group
			조직 이름을 입력하십시오.
			  [Unknown]:  hajimaro
			구/군/시 이름을 입력하십시오?
			  [Unknown]:  Seoul
			시/도 이름을 입력하십시오.
			  [Unknown]:  Seoul
			이 조직의 두 자리 국가 코드를 입력하십시오.
			  [Unknown]:  KR
			CN=kdc.hajimaro.com, OU=architect group, O=hajimaro, L=Seoul, ST=Seoul, C=KR이(가) 맞습니까?
			  [아니오]:  y

			>> optional
				-dname "CN=kdc.hajimaro.com, OU=hajimaro, O=hajimaro, C=KR"
				-keystore [keystore] (default : ~/.keystore)
				-validity [day] (default : 365)
			>> Tomcat에서 사용하는 기본 keypass는 'changeit'이며 다르게 사용할 경우 server.xml에 keystorePass를 정의해야 한다.

		2) generate a CRT(certificate)
			a. use the CA to create signed certificates in a java keystore
				# create a CSR(Certificate Signing Request)
				$ keytool -certreq -file /repository/pki/CA/certs/kdc.csr -keystore /opt/pki/java/keystore/cacerts -alias kdc↙

				# sign the CSR
				$ openssl x509 -CA /repository/pki/CA/ca.crt -CAkey /repository/pki/CA/private/ca.key -CAcreateserial -req -in /repository/pki/CA/certs/kdc.csr -out /repository/pki/CA/certs/kdc.crt -days 365

			b. get unsigned certificate
				# create a CRT(certificate)
				$ keytool -export -alias kdc -keypass quftkxkd -file /repository/pki/CA/certs/kdc.crt -keystore /opt/pki/java/keystore/cacerts↙

		3) update keystore include certificate in keystore
			# if use the CA to create signed certificates, update keystore with the certificate chain
			# by importing the certificate chain for the certificate
			$ keytool -import -alias ssl.ca -trustcacerts -file /repository/pki/CA/ca.crt -keystore /opt/pki/java/keystore/cacerts

			$ keytool -import -alias kdc -file /repository/pki/CA/certs/kdc.crt -keystore /opt/pki/java/keystore/cacerts

			# configure client certificate
			$ keytool -import -alias kdc -file /repository/pki/CA/certs/kdc.crt -keypass quftkxkd -keystore $JAVA_HOME/jre/lib/security/cacerts↙
			>>  -Djavax.net.ssl.trustStore=/opt/pki/java/keystore/cacerts

	cf.set SSL certificate tomcat
		# Define a SSL HTTP/1.1 Connector
		$ vi CATALINA_HOME/conf/server.xml↙
		<!-- Define a SSL HTTP/1.1 Connector on port 7749(default:8443) - start - -->
		<Connector protocol="org.apache.coyote.http11.Http11NioProtocol"
							 port="7749" minSpareThreads="5" maxSpareThreads="75"
							 enableLookups="true" disableUploadTimeout="true"
							 acceptCount="100"  maxThreads="200"
							 scheme="https" secure="true" SSLEnabled="true"
							 keystoreFile="/opt/pki/java/keystore/cacerts" keystorePass="quftkxkd"
							 clientAuth="false" sslProtocol="TLS"/>
		<!-- Define a SSL HTTP/1.1 Connector on port 7749 - end - -->

		cf.SSL debugging
			# java 실행옵션에 아래를 추가한다.
			-Djavax.net.debug=ssl,trustmanager

**********************************************************
90. trobleshooting
**********************************************************

**********************************************************
99. reference sites
**********************************************************
http://j30231.tistory.com/21  
http://j30231.tistory.com/22



