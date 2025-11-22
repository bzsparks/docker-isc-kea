## Kea TLS High Availability communication channel certificate

https://kea.readthedocs.io/en/kea-3.0.2/arm/hooks.html#https-support
<br/>
<br/>
<br/>
### Generate ED25519 key
```shell
openssl genpkey -algorithm ed25519 -out server.key
```

### Generate CSR
```shell
openssl req -new -subj "/CN=kea-ha-primary.bzsprks.com" -key server.key -out server.csr \
-addext "subjectAltName = DNS:=kea-ha-primary.bzsprks.com,DNS:=kea-ha-secondary.bzsprks.com"
```

### Generate 10 year self-signed certificate using my.cnf for certificate extension details
```shell
openssl x509 -req -sha256 -extfile my.cnf -extensions ext -in server.csr -days 3650 -signkey \
server.key -out server.crt
```

* In the Kea DHCP config we specifiy the server.crt as the CA and Servr certificates since it is self-signed.
* Kea http control agent - encrypted with Caddy reverse proxy using LetsEncrypt TLS, exposed on 443 externally mapped to 8000 internally
* Kea HA control channel - uses self-signed 10 year certificate to encrypt HA communications between Kea servers
* Kea DDNS server is unencryted but never exposed to the external network.  It is bound to the docker bridge network











