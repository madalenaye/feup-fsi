# Seed Labs - Packet Sniffing and Spoofing Lab

## Tarefa 1: Usar *Scapy* para fazer *Sniff* e *Spoof* de Pacotes

### Tarefa 1.1: *Sniffing* de Pacotes

#### Tarefa 1.1A

#### Tarefa 1.1B

### Tarefa 1.2: *Spoofing* de Pacotes ICMP

### Tarefa 1.3: *Traceroute*

### Tarefa 1.4: *Sniffing* e depois *Spoofing*


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
