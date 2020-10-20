# Configuring NGINX for Mutual TLS and test clients

Notes setting up Self-Signed TLS (Server and Client) Certificates and Configuring NGINX for Mutual TLS
and test clients for vin router use case

Examples below will be setting up self-signed certificate for testing

## Certificate Authority

Create a Certificate Authority root (which represents this server)

Organization & Common Name: Some identifier for this server CA

```bash
# Create designated folder
mkdir certificate_authority

# Create the CA Key and Certificate for signing Client Certs
openssl genrsa -out certificate_authority/ca.key 4096 # use -des3 , for password
openssl req -new -x509 -days 36500 -key certificate_authority/ca.key -out certificate_authority/ca.crt \
-subj "/C=US/ST=Colorado/L=Denver/O=Example Inc/CN=*.example.com" # 10years (36500 days) for test purposes only
```

## Server: Key, CSR, and Certificate

Create the Server Key, CSR, and Certificate for NGINX 

```bash
# Create designated folder
mkdir server
openssl genrsa -out server/server.key 4096 # use -des3 , for password
# Set Organization (`O`) as Organization/Group Identifer and Common Name (`CN`) to the FQDN or wildcard domain name
openssl req -new -key server/server.key -out server/server.csr -subj "/C=US/ST=Colorado/L=Denver/O=Example Inc/CN=*.example.com"

# We're self signing our own server cert here.  Obviously a is a no-no in production!
openssl x509 -req -days 36500 -in server/server.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key -set_serial 01 -out server/server.crt # 10years (36500 days) for test purposes only
```

**create folders**

```
mkdir -p client_certs/client_1 client_certs/client_2 client_certs/client_3
```

**client_1**

 * subj "C=DE, O=Daimler AG, OU=MBIIS-CERT, CN=WDC2539151V2xxx"
 * valid
 * serial - 01

```bash
# client_1
openssl genrsa -out client_certs/client_1/client_1.key 4096 # use -des3 , for password

openssl req -new -key client_certs/client_1/client_1.key -out client_certs/client_1/client_1.csr \
-subj "/C=DE/O=Daimler AG/OU=MBIIS-CERT/CN=WDC2539151V2xxx"

# Self-sign and set expire to 10years (36500 days) for test purposes only
openssl x509 -req -days 36500 -in client_certs/client_1/client_1.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key \
-set_serial 01 -out client_certs/client_1/client_1.crt  

openssl pkcs12 -export -clcerts -in client_certs/client_1/client_1.crt -inkey client_certs/client_1/client_1.key -out client_certs/client_1/client_1.p12 -password pass:password # weak password for testing

openssl pkcs12 -in client_certs/client_1/client_1.p12 -out client_certs/client_1/client_1.pem -clcerts -passin pass:password -passout pass:password # weak password for testing
```

**client_2**

 * subj "C=DE, O=Daimler AG, OU=MBIIS-CERT, CN=WDCABCDEFGxxx"
 * valid
 * serial 02

```bash
# client_2
openssl genrsa -out client_certs/client_2/client_2.key 4096 # use -des3 , for password

openssl req -new -key client_certs/client_2/client_2.key -out client_certs/client_2/client_2.csr \
-subj "/C=DE/O=Daimler AG/OU=MBIIS-CERT/CN=WDC2539151V2xxx"

# Self-sign and set expire to 10years (36500 days) for test purposes only
openssl x509 -req -days 36500 -in client_certs/client_2/client_2.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key \
-set_serial 01 -out client_certs/client_2/client_2.crt  

openssl pkcs12 -export -clcerts -in client_certs/client_2/client_2.crt -inkey client_certs/client_2/client_2.key -out client_certs/client_2/client_2.p12 -password pass:password # weak password for testing

openssl pkcs12 -in client_certs/client_2/client_2.p12 -out client_certs/client_2/client_2.pem -clcerts -passin pass:password -passout pass:password # weak password for testing

```


**client_2**

 * subj "C=DE, O=Daimler AG, OU=MBIIS-CERT, CN=invalid"
 * invalid
 * serial 02

```bash
# client_2
openssl genrsa -out client_certs/client_2/client_2.key 4096 # use -des3 , for password

openssl req -new -key client_certs/client_2/client_2.key -out client_certs/client_2/client_2.csr \
-subj "/C=DE/O=Daimler AG/OU=MBIIS-CERT/CN=invalid"

# Self-sign and set expire to 10years (36500 days) for test purposes only
openssl x509 -req -days 36500 -in client_certs/client_2/client_2.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key \
-set_serial 01 -out client_certs/client_2/client_2.crt  

openssl pkcs12 -export -clcerts -in client_certs/client_2/client_2.crt -inkey client_certs/client_2/client_2.key -out client_certs/client_2/client_2.p12 -password pass:password # weak password for testing

openssl pkcs12 -in client_certs/client_2/client_2.p12 -out client_certs/client_2/client_2.pem -clcerts -passin pass:password -passout pass:password # weak password for testing
```