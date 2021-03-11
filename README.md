# TLS-certificates
Certificados TLS para pruebas de sistemas *seguros*

## Utilidades necesarias
- `openssl`, que puede encontrar en la instalación de Git: `C:\Program Files\Git\usr\bin\openssl.exe`
- `keytool`, que se instala con el JDK
### 1) Genera el certificado y el juego de claves para la Certificate Authority (CA)
$ `openssl req -new -x509 -days 3650 -extensions v3_ca -keyout CA.key -out CA.cer`
```
Generating a RSA private key
.............+++++
.........................+++++
writing new private key to 'CA.key'
Enter PEM pass phrase: password
Verifying - Enter PEM pass phrase: password
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:.
State or Province Name (full name) [Some-State]:.
Locality Name (eg, city) []:.
Organization Name (eg, company) [Internet Widgits Pty Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (e.g. server FQDN or YOUR name) []:CA
Email Address []:.
```

### 2) Genera el juego de claves para el servidor
$ `openssl genrsa -des3 -out Server.key 2048`
```
Generating RSA private key, 2048 bit long modulus (2 primes)
..........................................................................+++++
..........................................................................+++++
e is 65537 (0x010001)
Enter pass phrase for Server.key: password
Verifying - Enter pass phrase for Server.key: password
```
### 3) Genera el Certificate Signing Request para el servidor
$ `openssl req -out Server.csr -key Server.key -new`
```
Enter pass phrase for Server.key: password
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:.
State or Province Name (full name) [Some-State]:.
Locality Name (eg, city) []:.
Organization Name (eg, company) [Internet Widgits Pty Ltd]:.
Organizational Unit Name (eg, section) []:.
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:.

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:password
An optional company name []:.
```
### 4) Genera el certificado del servidor
$ `openssl x509 -req -in Server.csr -CA CA.cer -CAkey CA.key -CAcreateserial -out Server.cer -days 3650`
```
Signature ok
subject=CN = localhost
Getting CA Private Key
Enter pass phrase for CA.key: password
```
### 5) Genera la KeyStore que se usará en el programa servidor
$ `openssl pkcs12 -export -in Server.cer -inkey Server.key -out ServerKeyStore.pkcs12 -name Server -noiter -nomaciter`
```
Enter pass phrase for Server.key: password
Enter Export Password: password
Verifying - Enter Export Password: password
```
### 6) Genera la TrustStore que se usará en el programa cliente
$ `keytool -import -file CA.cer -alias CA -keystore ClientTrustStore.jks`
```
Enter keystore password: password
Re-enter new password: password
... blah blah ...
Trust this certificate? [no]:  yes
Certificate was added to keystore
```
## Uso de los certificados en programas Java
Necesita los archivos `ServerKeyStore.pkcs12` para el programa servidor, y `ClientTrustStore.jks` para el programa cliente. Cópielos, por ejemplo, en el directorio raíz de su proyecto Java.
### Servidor
En el código, antes de ejecutar ninguna operación de aseguramiento TLS, debe incluir:
>```
>System.setProperty("javax.net.ssl.keyStore", "ServerKeyStore.pkcs12");
>System.setProperty("javax.net.ssl.keyStorePassword", "password");	
>```
### Cliente
En el código, antes de ejecutar ninguna operación de aseguramiento TLS, debe incluir:
```
System.setProperty( "javax.net.ssl.trustStore", "ClientTrustStore.jks" );
System.setProperty( "javax.net.ssl.trustStorePassword", "password" );
```
