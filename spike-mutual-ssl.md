# Mutual SSL

## Create folders

```
mkdir -p mutual-ssl/{ca,server,client}
```

## Create CA (Certificate Authority)

```
openssl genrsa -out ./ca/ca.key 2048
openssl req -new -key ./ca/ca.key -out ./ca/ca.csr -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Org/OU=CAT/CN=CA"
openssl x509 -req -days 365 -sha256 -extensions v3_ca -signkey ./ca/ca.key -in ./ca/ca.csr -out ./ca/ca.crt
```

## Create certs for Server

```
openssl genrsa -out ./server/example.com.key 2048
openssl req -new -key ./server/example.com.key -out ./server/example.com2.csr -config ./server/example.com.conf
openssl x509 -req -sha256 -days 365 -CA ./ca/ca.crt -CAkey ./ca/ca.key -CAcreateserial -extfile server/example.com.conf -extensions v3_req -in ./server/example.com.csr -out ./server/example.com.crt
```

## Create certs for Client

```
openssl genrsa -out ./client/client.key 2048
openssl req -new -key ./client/client.key -out ./client/client.csr -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Org/OU=CAT/CN=client"
openssl x509 -req -in ./client/client.csr -CA ./ca/ca.crt -CAkey ./ca/ca.key -CAcreateserial -out ./client/client.crt -days 365 -sha256
openssl pkcs12 -export -clcerts -in ./client/client.crt -inkey ./client/client.key -out ./client/client.p12
```

## Start the docker

```
docker-compose up
```

## Try it!

```
curl -v https://example.com:8443/
```

which will fail because the server certificate is not trusted by client.

add `-k` to ignore the certificate check.

```
curl -v -k https://example.com:8443/
```

which will fail again with error `400 No required SSL certificate was sent`, because we didn't pass client certificate in.

```
curl -v -k --cert ./client/client.crt --key ./client/client.key https://example.com:8443/
```

It Works!

- `-k` ignores server side SSL certficate validation
- `-v` shows verbose output
- `--cert` specified client certificate file
- `--key`  specified client private key file
