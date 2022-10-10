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
