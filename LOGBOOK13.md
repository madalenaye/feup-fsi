# Seed Labs - Packet Sniffing and Spoofing Lab

## Tarefa 1: Usar *Scapy* para fazer *Sniff* e *Spoof* de Pacotes

* Para configurarmos e inciarmos os *containers*, corremos `dcbuild` e `dcup`

* Em primeiro lugar, corremos `ifconfig` e procuramos o endereço IP `10.9.0.1` para descobrir o nome da *interface* de rede: `br-85326ea37364`

![Interface](/images/logbook13-tarefa1-1.png)

* De seguida, dentro da pasta partilhada `volumes`, criamos o ficheiro `mycode.py` com o código abaixo

```python
#!/usr/bin/env python3

from scapy.all import *

a = IP()
a.show()
```

* Ao correr o *script* dentro do *container*, o *output* foi o seguinte

![IP](/images/logbook13-tarefa1-2.png)

### Tarefa 1.1: *Sniffing* de Pacotes

* Para esta tarefa, criamos (dentro do mesmo diretório partilhado) o ficheiro `sniffer.py` com o seguinte código, de maneira a fazer *sniff* dos pacotes na *interface* `br-85326ea37364` e a imprimi-los

```python
#!/usr/bin/env python3
from scapy.all import *

def print_pkt(pkt):
	pkt.show()

pkt = sniff(iface='br-85326ea37364', prn=print_pkt)
```

#### Tarefa 1.1A

* Tornamos o programa executável com o comando `chmod a+x sniffer.py` e corremo-lo com privilégios *root*, ficando à espera que os outros *containers* enviassem pacotes para que pudessem ser *sniffed*

* Então, no *container* A, corremos `ping 10.9.0.6` para que fossem trocados pacotes com o *container* B

* Assim, no *container* em que corria o `sniffer.py`, conseguimos observer os pacotes trocados, conforme comprovam as capturas de ecrã abaixo

![Pacote](/images/logbook13-tarefa1-1a-1.png)
![Pacote](/images/logbook13-tarefa1-1a-2.png)

* As camadas dos pacotes *sniffed* são quatro: ARP, Ethernet, IP e ICMP

* A camada ARP ... TODO

* A camada Ethernet ... TODO

* A camada IP ... TODO

* A camada ICMP ... TODO

* Ao fazer `su seed` para tentar correr o `sniffer.py` sem privilégios *root*, observamos que a operação não nos é permitida

![OperationNotPermitted](/images/logbook13-tarefa1-1a-3.png)

#### Tarefa 1.1B

* Para capturar apenas o pacote ICMP, alteramos a chamada à função `sniff`, acrescentando-lhe o parâmetro `filter='icmp'`, resultando na seguinte linha de código: `pkt = sniff(iface='br-85326ea37364', filter='icmp' prn=print_pkt)`

* Para enviar pacotes ICMP, bastou-nos correr o comando `ping 10.9.0.6` no *container* A

* Conforme mostram as capturas de ecrã abaixo, conseguimos fazer *spoofing* dos pacotes ICMP

![ICMP](/images/logbook13-tarefa1-1b-1.png)
![ICMP](/images/logbook13-tarefa1-1b-2.png)

* Para capturar qualquer pacote TCP que vem de um endereço IP específico (`10.9.0.5`) e com um porto de destino número 23, editamos o filtro para `filter='tcp and src host 10.9.0.5 and port dst 23'`

* Para enviar pacotes TCP, tivemos que correr `telnet 10.9.0.6` no *container* A, dado que o endereço IP do *container* A é `10.9.0.5` e o protocolo `telnet` envia pacotes para o porto número 23

* Tal como demonstra a capturas de ecrã anexa, conseguimos fazer *spoofing* dos pacotes TCP com origem no endereço IP `10.9.0.5` e destino no porto número 23 (`telent`)

![TCP](/images/logbook13-tarefa1-1b-3.png)

* Para capturar quaisquer pacotes que vêm de ou vão para uma rede específica (`128.230.0.0/16`), modificamos o filtro para `filter='net 128.230.0.0'`

* Para enviar pacotes de e para esta rede, corremos `ping 128.230.0.1` no *container* A

* Como comprovam as capturas de ecrã presentes abaixo, conseguimos fazer *spoofing* dos pacotes com origem e destino na rede `128.230.0.0`

![NET](/images/logbook13-tarefa1-1b-4.png)
![NET](/images/logbook13-tarefa1-1b-5.png)
![NET](/images/logbook13-tarefa1-1b-6.png)

### Tarefa 1.2: *Spoofing* de Pacotes ICMP

* Para fazer *spoofing* de um pacote ICMP com um endereço IP de origem arbitrário, bastou adaptar o código fornecido no guião, adicionando-lhe a linha `a.src = '8.8.8.8'` (sendo este endereço IP de origem do pacote *spoofed*)

* Deste modo, criamos o ficheiro `spoofer.py` com o seguinte código:

```python
#!/usr/bin/env python3
from scapy.all import *

a = IP()
a.src = '8.8.8.8'
a.dst = '10.9.0.6'

b = ICMP()

p = a/b

send(p)

p.show()
```

* A linha `a.src = '8.8.8.8'` acrescentada ao *script* faz com que o pacote ICMP criado tenha `8.8.8.8` como endereço de origem, tal como pretendido

* A última linha (`p.show()`) serve apenas para imprimir o pacote enviado, presente na captura de ecrã abaixo, que comprova que o pacote foi enviado

![Spoofing](/images/logbook13-tarefa1-2-1.png)

* A prova de que a resposta ao pacote foi enviada para o endereço IP especificado é visível no *log* do Wireshark anexo abaixo, no qual se apresentam os endereços IP de origem e de destino dos pacotes ICMP trocados: o pacote *spoofed* (com "origem" `8.8.8.8` e destino `10.9.0.6`) e a resposta (com origem `10.9.0.6` e destino `8.8.8.8`)

![Wireshark](/images/logbook13-tarefa1-2-2.png)

### Tarefa 1.3: *Traceroute*

* De maneira a simular o funcionamento da ferramente `traceroute`, criamos um ficheiro `traceroute.py` com o seguinte código:

```python
#!/usr/bin/env python3
from scapy.all import *

i = 1

while (True):
	a = IP()
	a.dst = '8.8.8.8'
	a.ttl = i
	b = ICMP()
	r = sr1(a/b, timeout=5, verbose=0)
	if (r is not None):
		print(i, r.src)
		if (r.src == '8.8.8.8'):
			break
	i += 1
```

* De forma semelhante ao efetuado na tarefa anterior, este *script* cria um pacote ICMP, atribuindo-lhe `8.8.8.8` como endereço de destino

* O pacote é enviado através da chamada à função `sr1`, que aguarda pela resposta (os parâmetros indicam que o *timeout* deve ser de 5 segundos e que a função não deve imprimir qualquer *output*)

* O ciclo `while` incrementa, em cada iteração, o valor da variável `i`, que é atribuída ao *Time-To-Live* (TTL) do pacote, ou seja, indica por quantos *routers* é que o pacote pode passar

* Por cada resposta recebida, o programa imprime o endereço IP do *router* que a enviou (`print(i, r.src)`)

* Finalmente, o ciclo termina quando for recebida uma resposta cuja origem é o destino especificado inicialmente para o pacote: `8.8.8.8`

* Assim, ao correr `./traceroute.py`, verificamos que o pacote enviado passou por 13 *routers* (incluindo a origem e o destino) entre a máquina virtual e o destino `8.8.8.8`, sendo esta a distância entre ambas as máquinas

![Traceroute](/images/logbook13-tarefa1-3.png)

### Tarefa 1.4: *Sniffing* e depois *Spoofing*

* De modo a cumprir o pretendido nesta tarefa - fazer *sniffing* e depois *spoofing* de pacotes de rede -, criamos o ficheiro `task4.py`, adaptando os códigos desenvolvidos nas tarefas anteriores

* Em primeiro lugar, editamos o filtro presente no *script* de *sniffing* para se limitar a filtrar pacotes ICMP *echo request*, substituindo o filtro anteriormente presente por `filter='icmp[icmptype] == icmp-echo'`

* Em segundo lugar, criamos uma função `send_pkt` baseada no *script* de *spoofing* cujo objetivo é criar um pacote do tipo ICMP *echo reply* que simule ser uma resposta a um pacote ICMP *echo request*

* Para isto, basta manter todos os dados do pacote recebido e enviar um novo pacote com os meus dados, trocando apenas a origem e o destino, para que seja uma resposta

* Finalmente, basta invocar a função `send_pkt` sempre que for *sniffed* um pacote que cumpra os critérios do filtro, tal como é feito na última linha do código abaixo

```python
#!/usr/bin/env python3
from scapy.all import *

def send_pkt(pkt):
	a = IP()
	a.src = pkt[1].dst
	a.dst = pkt[1].src
	
	b = ICMP()
	b.type = 0
	b.id = pkt[2].id
	b.seq = pkt[2].seq
	
	load = pkt[3].load
	
	p = a/b/load
	send(p)
	p.show()
	
pkt = sniff(iface='br-85326ea37364', filter='icmp[icmptype] == icmp-echo', prn=send_pkt)
```

* Para testar o código desenvolvido, fizemos `ping` de três endereços IP diferentes, a partir do *container* A

* Ao fazer `ping 1.2.3.4` (um endereço inexistente na Internet), observamos que o *sniffing* e o *spoofing* foram bem-sucedidos, tendo em conta que foram recebidas respostas aos pacotes enviados, apesar de não existir nenhuma máquina alojada no endereço `1.2.3.4`

* As imagens abaixo demonstram os resultados obtidos pelo comando `ping` (as respostas), um pacote *spoofed* enviado (com origem `1.2.3.4` e destino `10.9.0.5`) e o *log* do Wireshark com os pacotes ICMP *request* e *reply* trocados

![Ping1](/images/logbook13-tarefa1-4-1.png)
![Pacote1](/images/logbook13-tarefa1-4-2.png)
![Wireshark1](/images/logbook13-tarefa1-4-3.png)

* Ao correr `ping 10.9.0.99` (um endereço inexistente na LAN), verificamos que, apesar de o *script* de *sniffing* e *spoofing* estar a correr, nenhum pacote passa no filtro de *sniffing* e, por isso, nenhum pacote é *spoofed*, pelo que não são enviadas respostas aos pacotes ICMP *request* enviados pelo comando `ping`, conforme comprovam as duas primeiras capturas de ecrã

* Isto deve-se ao facto de a máquina alojada no *container* A não saber como chegar ao endereço de destino, ou seja, não conhecer o caminho pelo qual os pacotes devem passar desde o endereço de origem (`10.9.0.5`) até ao de destino (`10.9.0.99`), tal como é possível verificar no *log* do Wireshark apresentado, no qual apenas se vêem pacotes ARP a solicitar que seja indicado à máquina em `10.9.0.5` qual o endereço MAC correspondente ao IP do destino (`10.9.0.99`)

![Ping2](/images/logbook13-tarefa1-4-4.png)
![Pacote2](/images/logbook13-tarefa1-4-5.png)
![Wireshark2](/images/logbook13-tarefa1-4-6.png)

* Ao realizar `ping 8.8.8.8` (um endereço existente na Internet), constatamos que os pacotes pretendidos são *sniffed* e *spoofed*, dado que as respostas esperadas chegam ao *container* A

* As respostas chegam em duplicado uma vez que o endereço `8.8.8.8` corresponde a uma máquina existente na Internet, pelo que, em primeiro lugar, são recebidas as respostas resultantes do *script* desenvolvido e, em segundo lugar, chegam as resposts "reais" enviadas pela máquina alojada no endereço de destino

* De facto, é possível verificar que o *spoofing* é realizado através da segunda imagem anexa e que as respostas chegam em duplicado através das outras duas fotografias (os resultados do `ping` e o *log* do Wireshark)

![Ping3](/images/logbook13-tarefa1-4-7.png)
![Pacote3](/images/logbook13-tarefa1-4-8.png)
![Wireshark3](/images/logbook13-tarefa1-4-9.png)


# CTF 13

* Este CTF consistiu na análise e interpretação de um ficheiro PCAP com várias conexões TLS, através do *Wireshark*

* Em primeiro lugar, descarregamos o ficheiro `dump.pcapng` e procuramos o procedimento *handshake* cujo número número aleatório utilizado na mensagem de `Client Hello` era o fornecido no enunciado: ***52362c11ff0ea3a000e1b48dc2d99e04c6d06ea1a061d5b8ddbf87b001745a27***

* Através de um filtro pelos pacotes do protocolo `TLS` e com a informação `Client Hello`, encontramos o pacote pretendido - número 814

![Pacote 814](/images/logbook13-framestart.png)

* Assim, percebemos que o procedimento de *handshake* que teríamos de analisar era aquele que continha o pacote número 814, apresentado na imagem abaixo

![Handshake](/images/logbook13-handshake.png)

* O formato da *flag* era `flag{<frame_start>-<frame_end>-<selected_cipher_suite>-<total_encrypted_appdata_exchanged>-<size_of_encrypted_message>}`

* O `frame_start` corresponde ao número do primeiro *frame* do procedimento de *handshake* do TLS, ou seja, ao *frame* `Client Hello` já identificado, com o número ***814***

* O `frame_end` corresponde ao número do último *frame* do procedimento de *handshake* do TLS, ou seja, ao *frame* que contém `New Session Ticket, Change Cipher Spec, Encrypted Handshake Message`, tendo este o número ***819***

* A *ciphersuite* escolhida para a conexão TLS encontra-se no *frame* 816 que contém a mensagem `Server Hello`, sendo o seu nome ***TLS_RSA_WITH_AES_128_CBC_SHA256*** e sendo este o valor a colocar em `selected_cipher_suite`

![Ciphersuite](/images/logbook13-ciphersuite.png)

* No canal, até à sua terminação, foram trocadas duas mensagens com dados cifrados da aplicação, nos *frames* números 820 e 821 (`Application Data`)

* O tamanho dos dados cifrados na primeira mensagem foi 80 e na segunda foi 1184, pelo que a sua soma é ***1264***, sendo este o valor a preencher em `total_encrypted_appdata_exchanged`

![ApplicationData](/images/logbook13-applicationdata.png)

* Finalmente, o tamanho da mensagem cifrada no *handshake* que conclui o procedimento de *handshake* - ou seja, no *frame* número 819, com a mensagem `Encrypted Handshake Message` - foi ***80***, sendo este o valor correspondente a `size_of_encrypted_message`

![Size](/images/logbook13-size.png)

* Deste modo, obtivemos a *flag* `flag{814-819-TLS_RSA_WITH_AES_128_CBC_SHA256-1264-80}`
