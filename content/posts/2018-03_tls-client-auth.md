+++
date = "2018-03-04T13:15:00+01:00"
title = "TLS Client Auth"
+++

# TLS Client Authentication for webservers
Instead of using basic auth for access restriction on websites with a predefined user base you can use TLS Client authentication.

This option requires the server to validate clients certificate while establishing a TLS connection. The usage of the known server-side TLS remains untouched, therefore you can use ACME provided certificates for your server and still validate client certificates you provided.

The procedure works like this: You create a CA, which is used to create certificates for your clients. The webserver needs your CA cert to validate any client connection.


## Install cfssl

We will use cloudflares excellent tool [cfssl](https://github.com/cloudflare/cfssl) to create our certificates.

```
go get -u github.com/cloudflare/cfssl/cmd/...
```

## Create general config

This config encapsulates expiry dates and usages:
```
cat > ca-config.json <<EOF
{
  "signing": {
    "default": {
      "expiry": "168h"
    },
    "profiles": {
      "server": {
        "expiry": "26280h",
        "usages": [
          "signing",
          "key encipherment",
          "server auth"
        ]
      },
      "client": {
        "expiry": "26280h",
        "usages": [
          "signing",
          "key encipherment",
          "client auth"
        ]
      },
      "client-server": {
        "expiry": "26280h",
        "usages": [
          "signing",
          "key encipherment",
          "server auth",
          "client auth"
        ]
      }
    }
  }
}
EOF
```

## 1. Create CA

```
cat > ca-csr.json <<EOF
{
  "CN": "BLANG Root CA",
  "key": {
    "algo": "rsa",
    "size": 4096
  },
  "names": [
    {
      "C": "DE",
      "L": "Munich",
      "O": "BLANG",
      "OU": "CA",
      "ST": "Bavaria"
    }
  ]
}
EOF

cfssl gencert -initca ca-csr.json | cfssljson -bare ca
```

## 2. Create intermediate CA

```
cat > ca-intermediate-csr.json <<EOF
{
  "CN": "BLANG Intermediate CA",
  "key": {
    "algo": "rsa",
    "size": 4096
  },
  "names": [
    {
      "C": "DE",
      "L": "Munich",
      "O": "BLANG",
      "OU": "Intermediate CA",
      "ST": "Bavaria"
    }
  ]
}
EOF

cfssl gencert -initca ca-intermediate-csr.json | cfssljson -bare ca-intermediate
```

## 3. Sign intermediate CA with root CA

Create a signing config for intermediates which can only sign end-certificates (path_len=0):
```
cat > ca-root-to-intermediate.json <<EOF
{
	"signing": {
		"default": {
			"expiry": "43800h",
			"ca_constraint": {
				"is_ca": true,
				"max_path_len": 0,
				"max_path_len_zero": true
			},
			"usages": [
				"digital signature",
				"cert sign",
				"crl sign",
				"signing"
			]
		}
	}
}
EOF
```

Sign the intermediate using the CA:
```
cfssl sign \
    -ca ca.pem \
    -ca-key ca-key.pem \
    -config ca-root-to-intermediate.json \
    ca-intermediate.csr | cfssljson -bare ca-intermediate
```

## 4. Create CA Bundle

```
cat ca.pem ca-intermediate.pem > ca-bundle.pem
```

## 5. Client

```
cat > client1-csr.json <<EOF
{
  "CN": "client1",
  "hosts": [
    ""
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "DE",
      "L": "Munich",
      "O": "BLANG",
      "OU": "Client",
      "ST": "Bavaria"
    }
  ]
}
EOF


cfssl gencert -ca ./ca-intermediate.pem -ca-key ./ca-intermediate-key.pem -config ca-config.json -profile client client1-csr.json | cfssljson -bare client1

# Pack to pkcs12 format, the chain is provided by the server, this avoids warnings on android
openssl pkcs12 -export -out client1.p12 -inkey client1-key.pem -in client1.pem

# Alternative: Include the CAs
openssl pkcs12 -export -out client1.p12 -inkey client1-key.pem -in client1.pem -certfile ca-bundle.pem
```

## 6. Install the client PKCS12 cert

Chrome: Settings -> Manage certificates -> Import

## 7. Setup your webserver

Transfer the `ca-bundle.pem` to your server.
```
# Nginx
ssl_client_certificate /path/to/ca-bundle.pem;
ssl_verify_client on;

# Caddy with ACME (Let's encrypt)
tls your-email@example.com {
    clients /path/to/ca-bundle.pem
}
```


## Optional: Check the certs
Check the certs extensions:
```
openssl x509 -in client1.pem -text -noout
```

For clients:
```
X509v3 extensions:
    X509v3 Key Usage: critical
        Digital Signature, Key Encipherment
    X509v3 Extended Key Usage:
        TLS Web Client Authentication
```

For CAs/Intermediate CAs:
```
X509v3 extensions:
    X509v3 Key Usage: critical
        Digital Signature, Certificate Sign, CRL Sign
    X509v3 Basic Constraints: critical
        CA:TRUE, pathlen:0
```
If `pathlen:0` is set, this means this intermediate can only sign end-point certificates, not further intermediate CAs.

## Optional: Server certificate

```
cat > server-csr.json <<EOF
{
  "CN": "iot.test.blang.io",
  "hosts": [
    "iot.test.blang.io"
  ],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "DE",
      "L": "Munich",
      "O": "BLANG",
      "OU": "Server",
      "ST": "Bavaria"
    }
  ]
}
EOF

cfssl gencert -config ca-config.json -profile server -ca ./ca-intermediate.pem -ca-key ./ca-intermediate-key.pem server-csr.json | cfssljson -bare server
```
