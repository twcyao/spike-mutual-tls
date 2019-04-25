## self-signed ca (v3)


```
openssl genrsa -out ./ca/ca.key 4096
openssl req -new -key ./ca/ca.key -out ./ca/ca.csr -subj "/CN=service-sab-ca"
openssl x509 -req -days 3650 -sha256 -extfile ca/v3_ca.ext -signkey ./ca/ca.key -in ./ca/ca.csr -out ./ca/ca.crt
```

v3_ca.ext
```
subjectKeyIdentifier = hash
authorityKeyIdentifier = keyid,issuer
basicConstraints = CA:true
```

## Sign PingAccess CSR

```
openssl x509 -req -sha256 -days 730 -CA ./ca/ca.crt -CAkey ./ca/ca.key -CAcreateserial -in ./pingaccess/CAT_pp_service-sab.csr -out ./pingaccess/CAT_pp_service-sab.crt
openssl x509 -req -sha256 -days 730 -CA ./ca/ca.crt -CAkey ./ca/ca.key -CAcreateserial -in ./pingaccess/CAT_p_service-sab.csr -out ./pingaccess/CAT_p_service-sab.crt
```

## Sign test client
```
openssl genrsa -out ./client/client.key 4096
openssl req -new -key ./client/client.key -out ./client/client.csr -subj "/C=CN/ST=Shanghai/L=Shanghai/O=Porsche/OU=CAT/CN=client"
openssl x509 -req -sha256 -days 730 -CA ./ca/ca.crt -CAkey ./ca/ca.key -CAcreateserial -in ./client/client.csr -out ./client/client.crt
```

## verify
```
curl --cert ./client/client.crt --key ./client/client.key https://service-sab.geo.mycat.aws.porsche-preview.cloud
```
