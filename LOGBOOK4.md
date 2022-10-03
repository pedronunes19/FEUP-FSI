# Task 1

Nesta tarefa observamos o comportamento das funções `printenv` e `env` - imprimem todas as variáveis de ambiente do sistema - e manipulamo-las, adicionando e eliminando, através dos comandos `export` e unset, respetivamente.

# Task 2

Concluímos que os child processes herdam as variáveis de ambiente dos parent processes. O comando `diff` não teve qualquer output, o que significa que as variáveis de ambiente são as mesmas (ficheiros são iguais).

# Task 3

Na primeira compilação, quando o 3º parâmetro (`envp[]`) da função `execve` é **NULL**, não passasmos ao novo programa (que vai correr dentro do processo atual), nenhuma variável de ambiente, pois não houve output do `myenv.c`. Concluímos então que temos de passar as variáveis através do 3º argumento (envp[])

Na 2ª compilação podemos confirmar que as variáveis de ambiente foram passadas do programa inicial para o novo programa, ao chamar `execve("usr/bin/env", arg, environ)`.

# Task 4

Nesta tarefa, pudemos confirmar que a função system usa a função `execl`, de forma a executar `"bin/sh"`, e `execl` chama `execve`, passando-lhe a array com variáveis de ambiente. Assim, com a função `system`, as variàveis de ambiente são passadas ao novo programa.

# Task 5

Verificamos que as variáveis de ambiente **PATH** e **ANY_NAME** foram alteradas no processo criado. No entanto a **LD_LIBRARY_PATH** não foi alterada. Isto é um mecanismo de defesa que impede outro utilizador de alterar de alterar linked libraries e executar código malicioso.

# Task 6 

Como root, conseguimos alterar a variável **PATH** de outro utilizador, algo que verificamos na tarefa anterior no programa **SET-UID**.
Conseguindo alterar o **PATH**, e como o programa executa comandos que utilizam o caminho relativo, se alterarmos o **PATH** para um caminho que contenha um executável `ls`, irá ser executado esse código, em vez do suposto `/bin/ls/`. Isto é uma vulnerabilidade porque, se alguém conseguir alterar a variável `PATH`, acaba por conseguir ter todo o controlo.
