# TLS

## Step 1: Generate CA files serverca.crt and servercakey.pem. This allows the signing of server and client keys ##

_linux_commands/TLS_certificate$ openssl genrsa -out servercakey.pem <br>
Generating RSA private key, 2048 bit long modulus (2 primes) <br>
...................................................................................+++++ <br>
.............................................................+++++ <br>
e is 65537 (0x010001)_ <br>

_linux_commands/TLS_certificate$ openssl req -new -x509 -key servercakey.pem -out serverca.crt <br>
You are about to be asked to enter information that will be incorporated <br>
into your certificate request. <br>
What you are about to enter is what is called a Distinguished Name or a DN. <br>
There are quite a few fields but you can leave some blank <br>
For some fields there will be a default value, <br>
If you enter '.', the field will be left blank. <br>
-----_ <br>
_Country Name (2 letter code) [AU]: <br>
State or Province Name (full name) [Some-State]: <br>
Locality Name (eg, city) []: <br>
Organization Name (eg, company) [Internet Widgits Pty Ltd]:test <br>
Organizational Unit Name (eg, section) []: <br>
Common Name (e.g. server FQDN or YOUR name) []: <br>
Email Address []:_ <br>


## Step 2: Create the server private key (server.crt) and public key (server.key) ## <br>

_linux_commands/TLS_certificate$ openssl genrsa -out server.key <br>
Generating RSA private key, 2048 bit long modulus (2 primes) <br>
...........+++++ <br>
...........+++++ <br>
e is 65537 (0x010001)_ <br>

_linux_commands/TLS_certificate$ openssl req -new -key server.key -out server_reqout.txt <br>
You are about to be asked to enter information that will be incorporated <br>
into your certificate request. <br>
What you are about to enter is what is called a Distinguished Name or a DN. <br>
There are quite a few fields but you can leave some blank <br>
For some fields there will be a default value, <br>
If you enter '.', the field will be left blank. <br>
-----_ <br>
_Country Name (2 letter code) [AU]: <br>
State or Province Name (full name) [Some-State]: <br>
Locality Name (eg, city) []: <br>
Organization Name (eg, company) [Internet Widgits Pty Ltd]:test <br>
Organizational Unit Name (eg, section) []: <br>
Common Name (e.g. server FQDN or YOUR name) []: <br>
Email Address []:_ <br>

_Please enter the following 'extra' attributes <br>
to be sent with your certificate request <br>
A challenge password []: <br>
An optional company name []: <br>
linux_commands/TLS_certificate$ openssl x509 -req -in server_reqout.txt -days 3650 -sha256 -CAcreateserial -CA serverca.crt -CAkey servercakey.pem -out server.crt
Signature ok <br>
subject=C = AU, ST = Some-State, O = test <br>
Getting CA Private Key_ <br>

## Step 3: Create the client private key (client.crt) and public key (client.key) ## <br>

_linux_commands/TLS_certificate$ openssl genrsa -out client.key <br>
Generating RSA private key, 2048 bit long modulus (2 primes) <br>
........................................................+++++ <br>
.......+++++ <br>
e is 65537 (0x010001)_ <br>

_linux_commands/TLS_certificate$ openssl req -new -key client.key -out client_reqout.txt <br>
You are about to be asked to enter information that will be incorporated <br>
into your certificate request. <br>
What you are about to enter is what is called a Distinguished Name or a DN. <br>
There are quite a few fields but you can leave some blank <br>
For some fields there will be a default value, <br>
If you enter '.', the field will be left blank. <br>
-----_ <br>
_Country Name (2 letter code) [AU]: <br>
State or Province Name (full name) [Some-State]: <br>
Locality Name (eg, city) []: <br>
Organization Name (eg, company) [Internet Widgits Pty Ltd]:test <br>
Organizational Unit Name (eg, section) []: <br>
Common Name (e.g. server FQDN or YOUR name) []: <br>
Email Address []:_ <br>

_Please enter the following 'extra' attributes <br>
to be sent with your certificate request <br>
A challenge password []: <br>
An optional company name []:_ <br>

_linux_commands/TLS_certificate$ openssl x509 -req -in client_reqout.txt -days 3650 -sha256 -CAcreateserial -CA serverca.crt -CAkey servercakey.pem -out client.crt <br>
Signature ok <br>
subject=C = AU, ST = Some-State, O = test <br>
Getting CA Private Key_ <br>

## Step 4: Set file permissions ## <br>

_linux_commands/TLS_certificate$ chmod 700 server.crt server.key_ <br>
_linux_commands/TLS_certificate$ chmod 700 client.crt client.key_ <br>

## Step 5: Crete .pfx file for browser ## <br>

_linux_commands/TLS_certificate$ openssl pkcs12 -export -in client.crt -inkey client.key -out myPrivateCert.pfx <br>
Enter Export Password:<br>
Verifying - Enter Export Password:_ <br>

## Step 6: Create .pem file to configure for webpage server ## <br>

_linux_commands/TLS_certificate$ cat server.key server.crt > server.pem <br>
linux_commands/TLS_certificate$ cat client.key client.crt > client.pem <br>
linux_commands/TLS_certificate$ ls <br>
client.crt  client.pem         myPrivateCert.pfx  servercakey.pem  server.crt  server.pem <br>
client.key  client_reqout.txt  serverca.crt       serverca.srl     server.key  server_reqout.txt_ <br>


## Step 7: Server and Client configuration ## <br>

**Server configuration**: Add the path of the server.pem and client.pem on the server in the webpage hosting file i.e lighttpd.conf in our case as follows <br>
#### SSL engine <br>
_ssl.engine                 = "enable" <br>
ssl.pemfile = "/home/TLS_cert/server/server.pem" <br>
ssl.verifyclient.activate = "enable" <br>
ssl.verifyclient.enforce = "enable" <br>
ssl.ca-file = "/home/TLS_cert/client/client.pem" <br>
ssl.verifyclient.depth = 2 <br>
ssl.verifyclient.username = "SSL_CLIENT_S_DN_CN"_ <br>

Lighttpd.conf file is also uploaded in this repo. <br>

**Client configuration:** Add the myPrivateCert.pfx file in the browser throught which we are going to access the webpage. <br>

**NOTE: Please make sure to use _"https://"_ before the URL of server** <br>
