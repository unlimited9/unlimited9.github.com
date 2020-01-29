## Instructions for Generating Repository SSL Keystores

$ openssl pkcs12 -export -in STAR.mediacategory.com.pem -inkey STAR.mediacategory.com.key -out keystore.p12 -name "mediacategory.com"


