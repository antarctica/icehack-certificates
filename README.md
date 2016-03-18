# Icehack Certificates

Collection of TLS certificates to secure production instances of the Icehack virtual environment.

## Overview

* Let's Encrypt is used to generate short-lived, free, TLS certificates.
* Multiple certificates are used, with 4 hostnames registered within each (except for 1-5 which has 5)

## Requirements

* Control of the `icehack.bas.ac.uk` sub-domain
* An account (private) key for Let's Encrypt

## Setup

1. Create a virtual machine accessible on the public internet
2. Download the [ACME Tiny](https://github.com/diafygi/acme-tiny) script to the VM
3. Register the relevant series of subdomains to point to the VM (e.g. `node1.icehack.bas.ac.uk`)
4. Upload account key to VM
5. Create private key for certificate [1]
6. Create a CSR for certificate [2]
7. Create challenges directory [3]
7. Install Nginx
8. Replace default site [4]
9. Register certificates [5]
10. Download certificate, CSR and private key to localhost
11. Build certificate trust chain [6]

[1]
```shell
openssl genrsa 4096 > nodes-01-05.icehack.bas.ac.uk.key
```

[2]
```shell
openssl req -new -sha256 -key nodes-01-05.icehack.bas.ac.uk.key -subj "/" -reqexts SAN -config <(cat /etc/ssl/openssl.cnf <(printf "[SAN]\nsubjectAltName=DNS:node1.icehack.bas.ac.uk,DNS:node2.icehack.bas.ac.uk,node3.icehack.bas.ac.uk,DNS:node4.icehack.bas.ac.uk,node5.icehack.bas.ac.uk")) > nodes-01-05.icehack.bas.ac.uk.csr
```

[3]
```shell
sudo mkdir -p /var/www/challenges/
sudo chmod 777 /var/www/challenges/
```

[4]
```conf
server {
    listen 80;
    server_name _;

    location /.well-known/acme-challenge/ {
        alias /var/www/challenges/;
        try_files $uri =404;
    }
}
```

[5]
```shell
python acme_tiny.py --account-key ./account.key --csr ./nodes-01-05.icehack.bas.ac.uk.csr --acme-dir /var/www/challenges/ > ./nodes-01-05.icehack.bas.ac.uk.crt
```

[6]
```shell
cat 01-05/nodes-01-05.icehack.bas.ac.uk.crt lets-encrypt-x1-cross-signed.pem > 01-05/nodes-01-05.icehack.bas.ac.uk-certificate-including-trust-chain.crt
```

## Usage

Use certificates and private keys as normal.

Note: Ensure you suitably protect private key material.
