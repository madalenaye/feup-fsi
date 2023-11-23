# SEED Labs - Secret-Key Encryption Lab

## Tarefa 1: Análise de Frequências

* Em primeiro lugar, corremos `./freq.py` para realizar uma análise de frequêcias ao ficheiro `ciphertext.txt`

* Com isto, descobrimos que o trigrama mais frequente é "ytn" e o segundo mais frequente é "vup"

* Consultando a página https://en.wikipedia.org/wiki/Trigram, percebemos que o "ytn" corresponderia a "THE" e "vup" a "AND"

* Então, corremos `tr ytnvup THEAND < ciphertext.txt > out.txt` para verificar o resultado desta tentativa de desencriptação parcial

* Ao interpretar o conteúdo do ficheiro `out.txt`, consideramos que nos parecia um texto plausível, pelo que continuamos a tentativa de decifração iterativamente

* De seguida, percebemos, através da leitura do artigo, que "sq" corresponderia a "KS" e passamos para a análise de letras individualmente, gerando sempre novos ficheiros `out.txt` e analisando-os

* Deste modo, chegamos ao comando `tr ytnvupsqxhcmaibferlgdjzokw THEANDKSORMICLFVPGWBYQUJXZ < ciphertext.txt > out.txt` (no qual as letras aparecem pela ordem em que foram descobertas), que resultou no seguinte texto do artigo:

```
THE OSCARS TURN ON SUNDAY WHICH SEEMS ABOUT RIGHT AFTER THIS LONG STRANGE AWARDS TRIP THE BAGGER FEELS LIKE A NONAGENARIAN TOO

THE AWARDS RACE WAS BOOKENDED BY THE DEMISE OF HARVEY WEINSTEIN AT ITS OUTSET AND THE APPARENT IMPLOSION OF HIS FILM COMPANY AT THE END AND IT WAS SHAPED BY THE EMERGENCE OF METOO TIMES UP BLACKGOWN POLITICS ARMCANDY ACTIVISM AND A NATIONAL CONVERSATION AS BRIEF AND MAD AS A FEVER DREAM ABOUT WHETHER THERE OUGHT TO BE A PRESIDENT WINFREY THE SEASON DIDNT JUST SEEM EXTRA LONG IT WAS EXTRA LONG BECAUSE THE OSCARS WERE MOVED TO THE FIRST WEEKEND IN MARCH TO AVOID CONFLICTING WITH THE CLOSING CEREMONY OF THE WINTER OLYMPICS THANKS PYEONGCHANG

ONE BIG QUESTION SURROUNDING THIS YEARS ACADEMY AWARDS IS HOW OR IF THE CEREMONY WILL ADDRESS METOO ESPECIALLY AFTER THE GOLDEN GLOBES WHICH BECAME A JUBILANT COMINGOUT PARTY FOR TIMES UP THE MOVEMENT SPEARHEADED BY POWERFUL HOLLYWOOD WOMEN WHO HELPED RAISE MILLIONS OF DOLLARS TO FIGHT SEXUAL HARASSMENT AROUND THE COUNTRY

SIGNALING THEIR SUPPORT GOLDEN GLOBES ATTENDEES SWATHED THEMSELVES IN BLACK SPORTED LAPEL PINS AND SOUNDED OFF ABOUT SEXIST POWER IMBALANCES FROM THE RED CARPET AND THE STAGE ON THE AIR E WAS CALLED OUT ABOUT PAY INEQUITY AFTER ITS FORMER ANCHOR CATT SADLER QUIT ONCE SHE LEARNED THAT SHE WAS MAKING FAR LESS THAN A MALE COHOST AND DURING THE CEREMONY NATALIE PORTMAN TOOK A BLUNT AND SATISFYING DIG AT THE ALLMALE ROSTER OF NOMINATED DIRECTORS HOW COULD THAT BE TOPPED

AS IT TURNS OUT AT LEAST IN TERMS OF THE OSCARS IT PROBABLY WONT BE 

WOMEN INVOLVED IN TIMES UP SAID THAT ALTHOUGH THE GLOBES SIGNIFIED THE INITIATIVES LAUNCH THEY NEVER INTENDED IT TO BE JUST AN AWARDS SEASON CAMPAIGN OR ONE THAT BECAME ASSOCIATED ONLY WITH REDCARPET ACTIONS INSTEAD A SPOKESWOMAN SAID THE GROUP IS WORKING BEHIND CLOSED DOORS AND HAS SINCE AMASSED MILLION FOR ITS LEGAL DEFENSE FUND WHICH AFTER THE GLOBES WAS FLOODED WITH THOUSANDS OF DONATIONS OF OR LESS FROM PEOPLE IN SOME COUNTRIES

NO CALL TO WEAR BLACK GOWNS WENT OUT IN ADVANCE OF THE OSCARS THOUGH THE MOVEMENT WILL ALMOST CERTAINLY BE REFERENCED BEFORE AND DURING THE CEREMONY ESPECIALLY SINCE VOCAL METOO SUPPORTERS LIKE ASHLEY JUDD LAURA DERN AND NICOLE KIDMAN ARE SCHEDULED PRESENTERS

ANOTHER FEATURE OF THIS SEASON NO ONE REALLY KNOWS WHO IS GOING TO WIN BEST PICTURE ARGUABLY THIS HAPPENS A LOT OF THE TIME INARGUABLY THE NAILBITER NARRATIVE ONLY SERVES THE AWARDS HYPE MACHINE BUT OFTEN THE PEOPLE FORECASTING THE RACE SOCALLED OSCAROLOGISTS CAN MAKE ONLY EDUCATED GUESSES

THE WAY THE ACADEMY TABULATES THE BIG WINNER DOESNT HELP IN EVERY OTHER CATEGORY THE NOMINEE WITH THE MOST VOTES WINS BUT IN THE BEST PICTURE CATEGORY VOTERS ARE ASKED TO LIST THEIR TOP MOVIES IN PREFERENTIAL ORDER IF A MOVIE GETS MORE THAN PERCENT OF THE FIRSTPLACE VOTES IT WINS WHEN NO MOVIE MANAGES THAT THE ONE WITH THE FEWEST FIRSTPLACE VOTES IS ELIMINATED AND ITS VOTES ARE REDISTRIBUTED TO THE MOVIES THAT GARNERED THE ELIMINATED BALLOTS SECONDPLACE VOTES AND THIS CONTINUES UNTIL A WINNER EMERGES

IT IS ALL TERRIBLY CONFUSING BUT APPARENTLY THE CONSENSUS FAVORITE COMES OUT AHEAD IN THE END THIS MEANS THAT ENDOFSEASON AWARDS CHATTER INVARIABLY INVOLVES TORTURED SPECULATION ABOUT WHICH FILM WOULD MOST LIKELY BE VOTERS SECOND OR THIRD FAVORITE AND THEN EQUALLY TORTURED CONCLUSIONS ABOUT WHICH FILM MIGHT PREVAIL

IN IT WAS A TOSSUP BETWEEN BOYHOOD AND THE EVENTUAL WINNER BIRDMAN IN WITH LOTS OF EXPERTS BETTING ON THE REVENANT OR THE BIG SHORT THE PRIZE WENT TO SPOTLIGHT LAST YEAR NEARLY ALL THE FORECASTERS DECLARED LA LA LAND THE PRESUMPTIVE WINNER AND FOR TWO AND A HALF MINUTES THEY WERE CORRECT BEFORE AN ENVELOPE SNAFU WAS REVEALED AND THE RIGHTFUL WINNER MOONLIGHT WAS CROWNED

THIS YEAR AWARDS WATCHERS ARE UNEQUALLY DIVIDED BETWEEN THREE BILLBOARDS OUTSIDE EBBING MISSOURI THE FAVORITE AND THE SHAPE OF WATER WHICH IS THE BAGGERS PREDICTION WITH A FEW FORECASTING A HAIL MARY WIN FOR GET OUT

BUT ALL OF THOSE FILMS HAVE HISTORICAL OSCARVOTING PATTERNS AGAINST THEM THE SHAPE OF WATER HAS NOMINATIONS MORE THAN ANY OTHER FILM AND WAS ALSO NAMED THE YEARS BEST BY THE PRODUCERS AND DIRECTORS GUILDS YET IT WAS NOT NOMINATED FOR A SCREEN ACTORS GUILD AWARD FOR BEST ENSEMBLE AND NO FILM HAS WON BEST PICTURE WITHOUT PREVIOUSLY LANDING AT LEAST THE ACTORS NOMINATION SINCE BRAVEHEART IN THIS YEAR THE BEST ENSEMBLE SAG ENDED UP GOING TO THREE BILLBOARDS WHICH IS SIGNIFICANT BECAUSE ACTORS MAKE UP THE ACADEMYS LARGEST BRANCH THAT FILM WHILE DIVISIVE ALSO WON THE BEST DRAMA GOLDEN GLOBE AND THE BAFTA BUT ITS FILMMAKER MARTIN MCDONAGH WAS NOT NOMINATED FOR BEST DIRECTOR AND APART FROM ARGO MOVIES THAT LAND BEST PICTURE WITHOUT ALSO EARNING BEST DIRECTOR NOMINATIONS ARE FEW AND FAR BETWEEN
```

* Assim, concluímos que a chave utilizada foi a presente na seguinte tabela:

| Alfabeto | Alfabeto Cifrado |
|----------|------------------|
| a | c |
| b | f |
| c | m |
| d | y |
| e | p |
| f | v |
| g | b |
| h | r |
| i | l |
| j | q |
| k | x |
| l | w |
| m | i |
| n | e |
| o | j |
| p | d |
| q | s |
| r | g |
| s | k |
| t | h |
| u | n |
| v | a |
| w | z |
| x | o |
| y | t |
| z | u |

## Tarefa 2: Encriptação usando Diferentes Cifras e Modos

* Antes de iniciarmos a tarefa, renomeamos o ficheiro `out.txt` anteriormente determinado para `article.txt`

* Em primeiro lugar, experimentamos a cifra `-aes-128-cbc`, correndo o comando `openssl enc -aes-128-cbc -e -in article.txt -out cipher1.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708`

* Depois de encriptado o ficheiro, conseguimos desencriptá-lo com `openssl enc -aes-128-cbc -d -in cipher1.bin -out out1.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708`

![-aes-128-cbc](/images/logbook10-tarefa2-1.png)

* Em segundo lugar, experimentamos a cifra `-bf-cbc`, correndo o comando `openssl enc -bf-cbc -e -in article.txt -out cipher2.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708`

* Depois de encriptado o ficheiro, conseguimos desencriptá-lo com `openssl enc -bf-cbc -d -in cipher2.bin -out out2.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708`

![-bf-cbc](/images/logbook10-tarefa2-2.png)

* Em terceiro lugar, experimentamos a cifra `-aes-128-cfb`, correndo o comando `openssl enc -aes-128-cfb -e -in article.txt -out cipher3.bin -K 00112233445566778889aabbccddeeff -iv 0102030405060708`

* Depois de encriptado o ficheiro, conseguimos desencriptá-lo com `openssl enc -aes-128-cfb -d -in cipher3.bin -out out3.txt -K 00112233445566778889aabbccddeeff -iv 0102030405060708`

![-aes-128-cfb](/images/logbook10-tarefa2-3.png)

## Tarefa 3: Modo de Encriptação - ECB vs. CBC

* Em primeiro lugar, correndo o comando `openssl enc -bf-ecb -e -in pic_original.bmp -out pic_ecb.bmp -K 00112233445566778889aabbccddeeff`, encriptamos o ficheiro da imagem usando ECB (*Eletctronic Code Book*)

* De seguida, fizemos `head -c 54 pic_original.bmp > header`, `tail -c +55 pic_ecb.bmp > body` e `cat header body > new_ecb.bmp`, de maneira a poder tratar o ficheiro encriptado como uma imagem

* Assim, abrimos a imagem encriptada com `eog new_ecb.bmp` e observamos que a imagem encriptada é muito semelhante à imagem original, tendo apenas ocorrido uma alteração das cores dos píxeis de forma constante (isto é, a cor X foi sempre convertida na cor Y)

![new_ecb.bmp](/images/logbook10-tarefa3-1.png)

* Em segundo lugar, correndo o comando `openssl enc -bf-cbc -e -in pic_original.bmp -out pic_cbc.bmp -K 00112233445566778889aabbccddeeff -iv 0102030405060708`, encriptamos o ficheiro da imagem usando CBC (*Cipher Block Chaining*)

* De seguida, fizemos `head -c 54 pic_original.bmp > header`, `tail -c +55 pic_cbc.bmp > body` e `cat header body > new_cbc.bmp`, de maneira a poder tratar o ficheiro encriptado como uma imagem

* Assim, abrimos a imagem encriptada com `eog new_cbc.bmp` e observamos que a imagem encriptada é, desta vez, muito diferente da imagem original, estando aleatorizada e irreconhecível

![new_cbc.bmp](/images/logbook10-tarefa3-2.png)

* Repetimos os dois processos anteriores para a imagem seguinte, tendo obtido os resultados expostos abaixo, para os modos de encriptação ECB e CBC, respetivamente

![imagem](/images/logbook10-tarefa3-3.png)

![ecb](/images/logbook10-tarefa3-4.png)

![cbc](/images/logbook10-tarefa3-5.png)

* Neste caso, observamos que a imagem resultante do modo de encriptação ECB não está tão semelhante à original como no caso anterior, mas o ficheiro resultante do modo de encriptação CBC continua a parecer mais distante/diferente do orignal


# CTF 10

* Investigando o ficheiro `cipherspec.py` (que contém os algoritmos de geração de chaves, cifração e decifração), observamos que, apesar de a mensagem ser cifrada utilizando AES-CTR (AES, em modo de operação *counter mode*), a chave é gerada de forma incorreta

* Efetivamente, ainda que a o tamanho da chave sejam 16 *bytes* (`KEYLEN = 16`), apenas os 3 *bytes* (`offset = 3`) menos significativos são aleatórios, enquanto que os restantes 13 *bytes* são `0x00` (`key = bytearray(b'\x00'*(KEYLEN-offset))`)

* Ora, este método de geração da chave faz com que, na verdade, apesar de ela ser constituída por 16 *bytes* (e, por isso, à partida, poder tomar qualquer valor entre 0 e 256<sup>16</sup>), esteja limitada a um valor máximo de 256<sup>3</sup>, isto é, esteja compreendida entre 0 e 16777216

* Deste modo, o espaço de procura torna-se suficientemente pequeno ([0, 16777216]) para permitir um ataque por força-bruta

* A *ciphersuite* fornecida pode ser usada para cifrar dados através da função `enc`, que recebe uma chave (`k`) para a cifra, uma mensagem para cifrar (`m`) e um número que só pode ser utilizado uma vez, para inicializar a encriptação (`nonce`)

* De forma análoga, esta *ciphersuite* pode ser usada para decifrar dados através da função `dec`, que recebe a chave (`k`) da cifra, a mensagem para decifrar (`c`) e o número de uso único utilizado no processo de encriptação

* Ao corrermos `nc ctf-fsi.fe.up.pt 6003`, recebemos do servidor a resposta com `nonce: ae1b1491929498abefb5d184220d6ba7` e `ciphertext: 18f90bee1739c36912045bf32b665efb271e772cb9d40e6768bb55be539b7bc2545a25cead77b8`, sendo estes os parâmetros a utilizar no ataque a realizar para obter a *flag*

* Assim, para explorar a vulnerabilidade observada de maneira a quebrar o código, basta recorrer à função `dec` num ataque de força-bruta, percorrendo todos os valores possíveis para a chave (ou seja, entre 0 e 16777216) e tentando, para cada um deles, decifrar a mensagem (`ciphertext`) enviada pelo servidor, passando o `nonce` (recebido do servidor) como parâmetro

* Finalmente, para automatizar este processo de maneira a fazer com que o ataque saiba que encontrou a *flag*, basta comparar o início da tentativa de mensagem decifrada com "flag" (`if (flag[:4] == b"flag"):`) e, se coincidir, imprimir a mensagem decifrada, tendo em conta que esta igualdade significa que a chave foi corretamente encontrada e a mensagem decifrada

* Segundo os princípios acima expostos, desenvolvemos o excerto de código presente abaixo (no final do ficheiro `cipherspec.py`), que tenta encontra a chave (por força-bruta) para decifrar a mensagem recebida do servidor

```py
#!/usr/bin/python3

from cryptography.hazmat.primitives.ciphers import Cipher, algorithms, modes
import os

KEYLEN = 16

def gen(): 
	offset = 3 # Hotfix to make Crypto blazing fast!!
	key = bytearray(b'\x00'*(KEYLEN-offset)) 
	key.extend(os.urandom(offset))
	print(bytes(key))
	return bytes(key)

def enc(k, m, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	encryptor = cipher.encryptor()
	cph = b""
	cph += encryptor.update(m)
	cph += encryptor.finalize()
	return cph

def dec(k, c, nonce):
	cipher = Cipher(algorithms.AES(k), modes.CTR(nonce))
	decryptor = cipher.decryptor()
	msg = b""
	msg += decryptor.update(c)
	msg += decryptor.finalize()
	return msg

for i in range(16777216):
	flag = dec(i.to_bytes(16), bytes.fromhex("18f90bee1739c36912045bf32b665efb271e772cb9d40e6768bb55be539b7bc2545a25cead77b8"), bytes.fromhex("ae1b1491929498abefb5d184220d6ba7"))
	if (flag[:4] == b"flag"):
		print(flag)
```

* Assim, depois de corrermos o *script* e esperarmos algum tempo, obtivemos a *flag*: `flag{e63d6e1db7cdb7093a042a73b847227e}`

![Flag](/images/logbook10-flag.png)
