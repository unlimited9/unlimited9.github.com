su - app

### install
cd /apps/install  
wget http://apache.mirror.cdnetworks.com/tomcat/tomcat-9/v9.0.22/bin/apache-tomcat-9.0.22.tar.gz  
tar xvf apache-tomcat-9.0.22.tar.gz

mkdir -p /apps/tomcat  
mv /apps/install/apache-tomcat-9.0.22 /apps/tomcat/9.0.22

### instance configure

#### make directory
mkdir -p \  
/apps/tomcat/instances/es.01/bin  \  
/apps/tomcat/instances/es.01/conf \  
/apps/tomcat/instances/es.01/logs \  
/apps/tomcat/instances/es.01/temp \  
/apps/tomcat/instances/es.01/webapps/ROOT

cp -R /apps/tomcat/9.0.22/conf/* /apps/tomcat/instances/es.01/conf

sed -i -e 's/8005/\${tomcat\.port\.shutdown}/g' /apps/tomcat/instances/es.01/conf/server.xml  
sed -i -e 's/8080/\${tomcat\.port\.http}/g' /apps/tomcat/instances/es.01/conf/server.xml  
sed -i -e 's/8009/\${tomcat\.port\.ajp}/g' /apps/tomcat/instances/es.01/conf/server.xml  
sed -i -e 's/8443/\${tomcat\.port\.https}/g' /apps/tomcat/instances/es.01/conf/server.xml  

vi /apps/tomcat/instances/es.01/conf/server.xml
```
... 
    <Host name="localhost"  appBase="/pgms/tomcat/webapps"
          unpackWARs="true" autoDeploy="true">
    
        <Context path="/default" docBase="es" reloadable="false" />
...
```

mkdir -p /pgms/tomcat/webapps  
chown -R app.app /pgms 
