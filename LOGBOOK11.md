# Public-Key Infrastructure (PKI) Lab

## Task 1: Becoming a Certificate Authority (CA)

Em primeiro lugar, copiamos o ficheiro openssl.cnf para o nosso diretório.

```
[12/12/22]seed@VM:~/.../Labsetup$ cp /usr/lib/ssl/openssl.cnf .
```

Fizemos as mudanças necessárias no ** [ CA_default ] ** dentro do openssl.cnf, de forma a ficar como o enunciado pedia:

```
[ CA_default ]

dir		= ./demoCA		# Where everything is kept
certs		= $dir/certs		# Where the issued certs are kept
crl_dir	= $dir/crl		# Where the issued crl are kept
database	= $dir/index.txt	# database index file.
unique_subject	= no			# Set to 'no' to allow creation of
					# several certs with same subject.
new_certs_dir	= $dir/newcerts	# default place for new certs.
serial		= $dir/serial 		# The current serial number
```


## Task 2: Generating a Certificate Request for Your Web Server

## Task 3: Generating a Certificate for your server

## Task 4: Deploying Certificate in an Apache-Based HTTPS Website

## Task 5: Launching a Man-In-The-Middle Attack

## Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA
