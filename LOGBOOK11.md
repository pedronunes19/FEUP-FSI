# Public-Key Infrastructure (PKI) Lab

## Task 1: Becoming a Certificate Authority (CA)

Em primeiro lugar, copiamos o ficheiro openssl.cnf para o nosso diretório.

```
[12/12/22]seed@VM:~/.../Labsetup$ cp /usr/lib/ssl/openssl.cnf .
```

Fizemos as mudanças necessárias no [ CA_default ] dentro do openssl.cnf, de forma a ficar como o enunciado pedia:

```
[ CA_default ]

dir				= ./demoCA			# Where everything is kept
certs			= $dir/certs		# Where the issued certs are kept
crl_dir			= $dir/crl			# Where the issued crl are kept
database		= $dir/index.txt	# database index file.
unique_subject	= no				# Set to 'no' to allow creation of
									# several certs with same subject.
new_certs_dir	= $dir/newcerts		# default place for new certs.
serial			= $dir/serial 		# The current serial number
```

De seguida, criamos os respetivos diretórios e ficheiros necessários:

```
[12/12/22]seed@VM:~/.../Labsetup$ mkdir demoCA
[12/12/22]seed@VM:~/.../Labsetup$ cd demoCA
[12/12/22]seed@VM:~/.../Labsetup$ mkdir certs crl newcerts
[12/12/22]seed@VM:~/.../Labsetup$ mkdir crl
[12/12/22]seed@VM:~/.../Labsetup$ mkdir newcerts
[12/12/22]seed@VM:~/.../Labsetup$ touch index.txt
[12/12/22]seed@VM:~/.../Labsetup$ echo "1000" > serial
```

Criamos o certificado CA self-signed, com o comando do enunciado:

```
[12/12/22]seed@VM:~/.../Labsetup$ openssl req -x509 -newkey rsa:4096 -sha256 -days 3650 -keyout ca.key -out ca.crt -subj "/CN=www.modelCA.com/O=Model CA LTD./C=US" -passout pass:dees
```

Depois corremos os seguintes comandos que nos permitiram responder às questões abaixo.

```
[12/12/22]seed@VM:~/.../Labsetup$ openssl x509 -in ca.crt -text -noout
[12/12/22]seed@VM:~/.../Labsetup$ openssl rsa -in ca.key -text -noout
```

### Questions

#### What part of the certificate indicates this is a CA’s certificate?

Flag que diz **CA:TRUE**.

![](https://git.fe.up.pt/fsi/fsi2223/l06g03/-/raw/main/screenshots/CATRUE.png)

#### What part of the certificate indicates this is a self-signed certificate?

Como o X509v3 Subject Key Identifier é igual ao X509v3 Authority Key Identifier, então é um certificado self-signed.

![](https://git.fe.up.pt/fsi/fsi2223/l06g03/-/raw/main/screenshots/selfsigned.png)

#### In the RSA algorithm, we have a public exponent e, a private exponent d, a modulus n, and two secret numbers p and q, such that n = pq. Please identify the values for these elements in your certificate and key files.

Como podemos ver em baixo, os valores estão identificados.

```
RSA Private-Key: (4096 bit, 2 primes)
modulus:
    00:cb:3e:84:2c:f9:a2:23:8d:33:0a:18:86:70:d6:
    95:a8:90:ab:e5:20:9c:ef:33:8e:37:4d:2b:9f:b0:
    61:39:08:2b:03:9f:bb:22:58:3e:62:b5:51:26:19:
    a4:9a:c8:f0:43:58:93:bc:fd:87:80:de:98:a9:78:
    0b:74:50:ec:da:84:65:8b:33:50:2c:ad:22:ff:c8:
    9a:aa:3e:03:fe:a1:09:99:31:b2:f8:5b:00:69:fa:
    64:9e:04:7f:70:6d:2d:66:0c:26:82:b9:20:63:a1:
    ce:5b:c4:e8:89:88:f1:74:57:fc:4c:a4:0b:8c:6f:
    51:e7:a6:24:cb:6d:98:2a:cc:18:80:1f:e2:a0:b0:
    bf:6e:d0:66:d9:82:b9:e9:af:c1:f5:db:ba:83:1e:
    40:d2:59:66:03:6e:0b:6f:7c:ce:79:18:0c:db:f1:
    a6:8e:25:f1:eb:d5:80:bf:98:15:2a:56:4f:02:bf:
    5a:58:0a:ee:28:d3:cf:7e:45:86:fc:aa:fb:e1:0d:
    72:24:31:5c:a1:a1:f0:7e:5d:61:84:5e:ca:83:04:
    5b:6c:6d:d0:63:1c:a0:ec:53:df:e6:75:82:97:e2:
    b7:e5:1e:6b:9f:e8:da:37:aa:97:1c:18:b7:2a:87:
    18:34:9f:74:25:33:4b:5f:de:e4:2c:10:e7:25:10:
    b7:d9:aa:53:95:0b:5f:07:f3:cb:b4:a7:cd:15:78:
    16:7d:9e:c7:6e:30:32:df:25:e0:97:e7:1a:1b:f6:
    c4:6b:9d:32:65:49:bd:ce:95:0d:51:95:e6:57:21:
    16:2f:00:9a:3d:c3:3d:08:ef:3f:e3:33:08:d8:de:
    f8:fd:dd:b4:a8:e0:eb:20:71:27:8c:76:e8:b9:4b:
    0a:81:ac:4c:47:03:92:26:87:64:5f:8b:7b:5f:5e:
    9b:26:81:6e:a0:e9:30:c4:b0:06:2a:39:70:45:7e:
    45:bd:2e:58:7a:90:59:7c:69:21:3b:cf:8a:41:3f:
    6b:01:05:c4:a0:1e:87:0f:d9:26:ad:fe:e6:65:2c:
    1c:18:a9:5d:f7:59:55:43:ab:f2:37:6b:bc:6c:56:
    85:37:1a:fd:35:9c:79:d5:d8:b7:68:f0:ef:f7:73:
    81:e0:41:f2:60:d4:95:b9:57:c7:8a:d3:62:c8:94:
    c3:77:a6:e4:33:e5:4e:22:11:fa:55:2b:d0:03:f7:
    5e:e8:b1:18:50:47:f4:ad:af:f8:b4:00:05:e1:96:
    50:96:7c:64:be:7a:9c:a9:7a:50:51:b2:e0:5b:e4:
    d7:ed:31:89:3a:13:9b:7b:c8:82:d8:c7:02:68:45:
    b9:fe:fc:f4:a4:60:01:8b:d8:d9:6a:b1:e5:b6:25:
    34:50:75
publicExponent: 65537 (0x10001)
privateExponent:
    00:b4:9c:8e:9a:e5:0f:b7:e6:1f:68:26:59:3a:67:
    06:c1:b0:26:81:4c:15:09:e0:57:ce:3f:0e:b8:2e:
    e6:86:e7:02:4a:8b:24:a2:25:a6:f2:d2:cc:15:3e:
    8e:6f:5a:87:60:61:93:90:4c:00:a1:7d:ae:4e:53:
    36:62:9c:13:8f:30:3e:88:90:05:fc:5d:b3:8f:78:
    36:31:79:40:d5:83:47:e3:52:2e:07:d3:de:af:4e:
    eb:21:1d:40:1e:a9:76:c1:8b:a4:a1:60:60:2f:09:
    b8:37:06:e9:da:66:ce:a3:24:19:3a:06:41:98:ff:
    c7:da:42:63:ca:3f:4f:0d:21:27:d8:9b:fd:29:ed:
    47:80:f3:43:a2:a1:30:13:41:b3:ec:86:e1:dc:e9:
    02:93:ab:0c:23:9a:24:21:63:d8:9b:f5:ca:5f:9e:
    03:f3:a8:36:ae:eb:a1:29:21:be:15:4c:73:94:2e:
    75:db:6d:83:2c:d6:e5:3a:02:11:2d:f1:c8:39:bb:
    58:26:5f:93:40:b3:86:e2:d4:9a:f7:25:c1:72:e2:
    69:58:16:d2:2b:71:62:74:01:29:24:44:62:d4:14:
    8e:74:d6:2e:b0:01:1e:02:7f:df:1f:01:bb:ab:d8:
    37:a3:73:db:a3:bf:4c:89:1a:aa:cb:9a:0d:be:7b:
    77:d4:28:e7:08:f1:f6:57:53:63:e8:3e:e2:58:fc:
    8d:34:1f:a8:19:14:3d:82:ef:29:0f:2e:03:04:ff:
    d4:9a:1a:e6:51:1b:a1:d9:f8:2b:bf:1c:73:50:c3:
    7b:18:b1:9b:01:a1:a8:75:53:9b:2a:1f:9e:53:ea:
    5e:d1:dc:2e:cd:2d:e9:62:7b:e2:8e:0d:f4:6a:f4:
    b3:bc:da:69:21:d1:4d:29:3b:f8:d3:61:64:86:01:
    76:b5:df:85:f5:d2:64:90:8f:e6:4b:2b:a8:2e:22:
    1b:10:2f:f1:db:dd:07:66:3b:33:1c:43:21:f8:5e:
    e4:f2:f8:80:e4:0a:ab:13:92:91:78:3f:ee:32:0d:
    53:a0:a6:45:53:55:82:f8:73:16:59:12:e5:11:ac:
    01:fe:9e:94:d8:40:d7:f0:a0:f8:35:97:46:27:cb:
    94:66:dd:21:89:cf:77:8a:0a:68:54:47:55:3b:df:
    7e:18:c1:e5:41:8b:2a:0e:95:ba:b7:95:78:f0:ee:
    16:88:6d:36:00:93:05:f9:24:e5:76:60:28:f4:d8:
    46:7a:61:35:df:6b:b1:96:88:ed:54:97:d8:85:d0:
    20:59:c7:94:57:dc:a9:1e:86:37:e1:5c:ec:0d:3b:
    7f:2f:5a:ab:8c:1b:3f:d6:06:79:c3:a6:07:8a:d9:
    65:8a:61
prime1:
    00:f3:94:e5:19:7b:20:fd:73:7d:d6:5a:03:7d:a3:
    2c:39:18:4f:ea:13:68:22:6e:d0:77:a8:83:93:9b:
    f3:f2:91:b5:64:d5:bc:0c:59:f6:00:9b:96:15:f7:
    8e:2f:17:35:6f:91:7f:2d:1a:91:5b:f9:30:17:4a:
    20:fe:b5:4c:1f:dc:7c:02:0f:38:d3:1f:a1:91:91:
    84:e6:17:52:b3:b1:4f:22:1d:2d:7d:63:c8:5e:45:
    33:51:fe:7e:86:cd:d5:4b:d2:bd:3b:4e:71:a0:0e:
    b3:a0:ab:b8:98:1e:1a:b7:cb:d2:37:b6:77:12:e7:
    2a:54:ae:3f:77:af:af:39:e5:37:f3:b5:22:eb:63:
    b1:5f:92:9f:81:c3:ec:0b:ef:95:5a:fe:8e:ee:e4:
    0a:eb:46:80:14:71:c0:92:01:01:ca:fd:ca:0f:12:
    db:0b:a3:e2:5c:7f:84:e2:ee:4c:21:bb:f7:f4:02:
    46:f9:27:36:49:cb:38:c9:e6:c7:ad:f4:50:06:69:
    eb:1f:89:32:eb:91:a6:7f:9d:c1:ba:3f:38:13:39:
    12:71:66:5d:c9:fa:34:46:1b:4f:5f:96:a2:30:7e:
    e9:94:00:ff:c8:6a:cf:41:a1:c3:32:0a:e3:c8:6c:
    f8:1b:91:f6:59:90:17:23:ce:71:88:26:bb:72:b1:
    e1:19
prime2:
    00:d5:9b:28:5d:52:e4:50:18:34:b2:2a:cf:6a:3c:
    74:f7:b8:83:53:20:8b:a3:b1:81:90:d0:9a:7a:f7:
    82:36:c4:d6:4d:c8:82:3e:e5:fd:87:27:54:0e:ce:
    e8:4a:00:ce:19:47:07:ed:63:68:ec:39:71:39:6e:
    65:0c:9a:ab:3c:61:41:e3:f3:19:97:b4:40:ce:55:
    fa:4e:bf:fa:da:f4:0b:33:d8:12:39:97:9b:9d:e8:
    af:5f:43:85:62:fc:aa:59:cf:64:98:e0:48:d8:37:
    ea:48:52:f1:0d:62:00:3b:c3:e9:d7:22:03:57:1d:
    d4:25:08:6a:1a:b8:3f:04:c8:54:85:3f:ec:40:e1:
    46:4c:93:ae:b0:2f:7d:7f:7a:8f:48:16:48:0e:8d:
    76:c9:8e:5a:64:89:1d:c0:61:99:58:42:94:11:59:
    9d:a8:07:a6:2b:bc:94:43:d1:4c:83:f4:ce:7d:0b:
    55:db:7b:e9:22:8b:44:bf:6f:72:89:57:81:65:2a:
    fe:5a:26:4c:f7:4a:e6:42:ec:a1:96:4d:ac:e9:cf:
    06:14:eb:01:ef:28:a1:6c:15:e6:08:b2:7a:f8:8b:
    c4:5d:c1:32:bb:ae:18:55:76:12:c5:00:43:a6:98:
    b1:dc:1c:70:13:e7:2b:c5:d0:34:f7:9a:89:05:59:
    49:bd
```

## Task 2: Generating a Certificate Request for Your Web Server

## Task 3: Generating a Certificate for your server

## Task 4: Deploying Certificate in an Apache-Based HTTPS Website

## Task 5: Launching a Man-In-The-Middle Attack

## Task 6: Launching a Man-In-The-Middle Attack with a Compromised CA
