# Task 1

Nesta tarefa observamos o comportamento das funções `printenv` e `env` - imprimem todas as variáveis de ambiente do sistema - e manipulamo-las, adicionando e eliminando variáveis, através dos comandos `export` e `unset`, respetivamente.

# Task 2

Concluímos que os child processes herdam as variáveis de ambiente dos parent processes. O comando `diff` não teve qualquer output, o que significa que as variáveis de ambiente são as mesmas (ficheiros resultantes do output são iguais para ambos os processos).

# Task 3

Na primeira compilação, quando o 3º parâmetro (`envp[]`) da função `execve` é **NULL**, não passamos ao novo programa (que vai correr dentro do processo atual) nenhuma variável de ambiente, não resultando qualquer output do programa `myenv.c`. Concluímos então que temos de passar as variáveis de ambiente através do 3º argumento (`envp[]`), como é especificado na entrada do manual (`man execve`).

Na 2ª compilação podemos confirmar que as variáveis de ambiente foram passadas do programa inicial para o novo programa, ao chamar `execve("usr/bin/env", arg, environ)`.

# Task 4

Nesta tarefa, pudemos confirmar que a função system usa a função `execl`, de forma a executar `"bin/sh"`, e `execl` chama `execve`, passando-lhe a array com variáveis de ambiente. Assim, chamando a função `system`, as variáveis de ambiente são passadas ao novo programa.

# Task 5

Verificamos que as variáveis de ambiente **PATH** e **ANY_NAME** foram alteradas no processo criado. No entanto **LD_LIBRARY_PATH** não foi alterada. Isto é um mecanismo de defesa que impede outro utilizador de alterar de alterar linked libraries e executar código malicioso.

# Task 6 

Como root, conseguimos alterar a variável **PATH** de outro utilizador, algo que verificamos na tarefa anterior no programa **SET-UID**.
Conseguindo alterar o **PATH**, e como o programa executa comandos que utilizam o caminho relativo, se alterarmos o **PATH** para um caminho que contenha um executável `ls`, irá ser executado esse código, em vez do suposto `/bin/ls/`.

my_ls.c
```c
#include <stdlib.h>
#include <stdio.h>
#include <unistd.h>

int main(){
    printf("my_ls.c %d\n", geteuid());
    system("ls");
    return 0;
}
````

ls.c
```c
#include <stdio.h>
#include <unistd.h>

int main(){
    printf("Malicious code.\n");
    system("ls: %d", getuid());
    return 0;
}
```

```
[10/03/22]seed@VM:~$ export PATH=/home/seed:$PATH
[10/03/22]seed@VM:~$ gcc my_ls.c -o my_ls
[10/03/22]seed@VM:~$ sudo chown root my_ls
[10/03/22]seed@VM:~$ sudo chmod 4755 my_ls
[10/03/22]seed@VM:~$ my_ls
my_ls.c 0
Desktop    Downloads  Music  my_ls.c   Public	  Videos
Documents  ls.c       my_ls  Pictures  Templates
[10/03/22]seed@VM:~$ gcc ls.c -o ls
[10/03/22]seed@VM:~$ my_ls
my_ls.c 0
Malicious code.
ls: 1000
```


Neste caso, ao alterar **PATH**, conseguimos executar um programa escrito por nós no lugar do comando `ls`.
Ao observar o valor devolvido pela função `geteuid()` percebemos que o nosso programa `ls` não tem privilégios root (UID != 0), ao contrário do programa `my_ls.c`. Nesta máquina virtual, `bin/sh` é um link simbólico para `bin/dash`, e este programa pôe em prática uma medida de segurança que não permite que o mesmo seja corrido como um processo Set-UID, mitigando neste caso a vulnerabilidade que encontramos.



 
