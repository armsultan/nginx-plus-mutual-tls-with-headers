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


# Create the CA Key and Certificate for signing Client Certs:

###### PICK ONE OF THE TWO FOLLOWING ######

# OPTION ONE: RSA key. these are very well-supported around the internet.
# you can swap out 4096 for whatever RSA key size you want. this'll generate a key
# with password "xxxx" and then turn around and re-export it without a password,
# because genrsa doesn't work without a password of at least 4 characters.
#
# some appliance hardware only works w/2048 so if you're doing IOT keep that in
# mind as you generate CA and client keys. i've found that frirefox & chrome will
# happily work with stuff in the bigger 8192 ranges, but doing that vs sticking with
# 4096 doesn't buy you that much extra practical security anyway.

openssl genrsa -aes256 -passout pass:xxxx -out certificate_authority/ca.pass.key 4096
openssl rsa -passin pass:xxxx -in certificate_authority/ca.pass.key -out certificate_authority/ca.key
rm ca.pass.key

# OPTION TWO: make an elliptic curve-based key.
# support for ECC varies widely, and support for the predefined curves also varies.
# it's "secp256r1" in this case, which is as well-supported as it gets but if you want to
# avoid NIST-provided things, or if you want to go with bigger/newer keys, you can
# swap that out:
#
# * check your openssl supported curves: `openssl ecparam -list_curves`
# * check client support for whatever browser/language/system/device you want to use:
#      https://en.wikipedia.org/wiki/Comparison_of_TLS_implementations#Supported_elliptic_curves

openssl ecparam -genkey -name secp256r1 | openssl ec -out certificate_authority/ca.key

###### END  "PICK ONE" SECTION ######

# whichever you picked, you should now have a `ca.key` file.

# now generate the CA root cert
# when prompted, use whatever you'd like, but i'd recommend some human-readable Organization
# and Common Name.
openssl req -new -x509 -days 3650 -key certificate_authority/ca.key -out certificate_authority/ca.crt \
-subj "/C=US/ST=Colorado/L=Denver/O=Example Inc/CN=*.example.com" # 10years (36500 days) for test purposes only
```

## Server: Key, CSR, and Certificate

Create the Server Key, CSR, and Certificate for NGINX 

```bash
# Create designated folder
mkdir server
# make an elliptic curve-based key
openssl ecparam -genkey -name secp256r1 | openssl ec -out server/server.key
# Set Organization (`O`) as Organization/Group Identifer and Common Name (`CN`) to the FQDN or wildcard domain name
openssl req -new -key server/server.key -out server/server.csr -subj "/C=US/ST=Colorado/L=Denver/O=Example Inc/CN=*.example.com"

# We're self signing our own server cert here.  Obviously a is a no-no in production!
openssl x509 -req -days 36500 -in server/server.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key -set_serial 01 -out server/server.crt # 10years (36500 days) for test purposes only
```

## Clients: Key, CSR, and Certificate

**create folders**

```
mkdir -p client_certs/client_1 client_certs/client_2 client_certs/client_3
```

### client_1

 * subj "C=DE, O=Daimler AG, OU=MBIIS-CERT, CN=WDC2539151V2xxx"
 * valid
 * serial - 01

```bash
# client_1
# client_id is *only* for the output filenames
# incrementing the serial number is important
CLIENT_ID="client_1"
CLIENT_SERIAL="01"

###### PICK ONE OF THE TWO FOLLOWING ######
# rsa
openssl genrsa -aes256 -passout pass:xxxx -out client_certs/${CLIENT_ID}/${CLIENT_ID}.pass.key 4096
openssl rsa -passin pass:xxxx -in client_certs/${CLIENT_ID}/${CLIENT_ID}.pass.key -out client_certs/${CLIENT_ID}/${CLIENT_ID}.key
rm client_certs/${CLIENT_ID}/${CLIENT_ID}.pass.key
# ec
openssl ecparam -genkey -name secp256r1 | openssl ec -out client_certs/${CLIENT_ID}/${CLIENT_ID}.key
###### END  "PICK ONE" SECTION ######

# whichever you picked, you should now have a `client.key` file.

# generate the CSR
# i think the Common Name is the only important thing here. think of it like
# a display name or login.
openssl req -new -key client_certs/${CLIENT_ID}/${CLIENT_ID}.key -out client_certs/${CLIENT_ID}/${CLIENT_ID}.csr \
-subj "/C=DE/O=Daimler AG/OU=MBIIS-CERT/CN=WDC2539151V2xxx"


# Self-sign and set expire to 10years (36500 days) for test purposes only

# issue this certificate, signed by the CA root we made in the previous section 
# set expire to 10years (36500 days) for test purposes only
openssl x509 -req -days 3650 -in client_certs/${CLIENT_ID}/${CLIENT_ID}.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key -set_serial ${CLIENT_SERIAL} -out client_certs/${CLIENT_ID}/${CLIENT_ID}.pem

# Bundle the private key & cert for end-user client use
#basically https://www.digicert.com/ssl-support/pem-ssl-creation.htm , with the entire trust chain
cat client_certs/${CLIENT_ID}/${CLIENT_ID}.key client_certs/${CLIENT_ID}/${CLIENT_ID}.pem > ${CLIENT_ID}.full.pem


```

### client_2

 * subj "C=DE, O=Daimler AG, OU=MBIIS-CERT, CN=WDCABCDEFGxxx"
 * valid
 * serial 02

```bash
# client_2
# client_id is *only* for the output filenames
# incrementing the serial number is important
CLIENT_ID="client_2"
CLIENT_SERIAL="02"

# ec key
openssl ecparam -genkey -name secp256r1 | openssl ec -out client_certs/${CLIENT_ID}/${CLIENT_ID}.key

# generate the CSR
openssl req -new -key client_certs/${CLIENT_ID}/${CLIENT_ID}.key -out client_certs/${CLIENT_ID}/${CLIENT_ID}.csr \
-subj "/C=DE/O=Daimler AG/OU=MBIIS-CERT/CN=WDCABCDEFGxxx"

# Self-sign and set expire to 10years (36500 days) for test purposes only
openssl x509 -req -days 3650 -in client_certs/${CLIENT_ID}/${CLIENT_ID}.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key -set_serial ${CLIENT_SERIAL} -out client_certs/${CLIENT_ID}/${CLIENT_ID}.pem

# Bundle the private key & cert for end-user client use
cat client_certs/${CLIENT_ID}/${CLIENT_ID}.key client_certs/${CLIENT_ID}/${CLIENT_ID}.pem > ${CLIENT_ID}.full.pem
```

### client_3

 * subj "C=DE, O=Daimler AG, OU=MBIIS-CERT, CN=invalid"
 * valid
 * serial 03


```bash
# client_3
# client_id is *only* for the output filenames
# incrementing the serial number is important
CLIENT_ID="client_3"
CLIENT_SERIAL="03"

# ec key
openssl ecparam -genkey -name secp256r1 | openssl ec -out client_certs/${CLIENT_ID}/${CLIENT_ID}.key

# generate the CSR
openssl req -new -key client_certs/${CLIENT_ID}/${CLIENT_ID}.key -out client_certs/${CLIENT_ID}/${CLIENT_ID}.csr \
-subj "/C=DE/O=Daimler AG/OU=MBIIS-CERT/CN=invalid"

# Self-sign and set expire to 10years (36500 days) for test purposes only
openssl x509 -req -days 3650 -in client_certs/${CLIENT_ID}/${CLIENT_ID}.csr -CA certificate_authority/ca.crt -CAkey certificate_authority/ca.key -set_serial ${CLIENT_SERIAL} -out client_certs/${CLIENT_ID}/${CLIENT_ID}.pem

# Bundle the private key & cert for end-user client use
cat client_certs/${CLIENT_ID}/${CLIENT_ID}.key client_certs/${CLIENT_ID}/${CLIENT_ID}.pem > client_certs/${CLIENT_ID}/${CLIENT_ID}.full.pem
```