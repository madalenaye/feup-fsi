# CTF 11

* Depois de analisarmos o enunciado do *ctf* e o ficheiro *chanllenge.py* verificamos que se tratava de uma cifra baseada em *RSA*, que assenta que no problema matemático da factorização de números inteiros

* Assim, o primeiro passo passou por correr o comando nc ctf-fsi.fe.up.pt 6004 e recebemos os valores ***n***, ***e*** e o ***ciphertext*** que corresponde à flag cifrada

![valores](images/logbook11-ctf.png)

* Os valores ***e*** e ***n*** foram os utilizados no RSA que cifrou a *flag*, sendo a flag cifrada o resultado de a elevar ao expoente ***e***, módulo ***n***

* De seguida, ao analisarmos o mecanismo de encriptação e desencriptação, concluimos que precisávamos de uma função que testasse se um número era primo ou não

* Nesse sentido, e por se tratarem de números muito grandes, optamos por utilizar o algoritmos de Miller-Rabin, cuja implementação se eencontra abaixo

```
def miller_rabin(n):

    if n == 2:
        return True

    if n % 2 == 0:
        return False

    r, s = 0, n - 1
    while s % 2 == 0:
        r += 1
        s //= 2
    for _ in xrange(8):
        a = random.randrange(2, n - 1)
        x = pow(a, s, n)
        if x == 1 or x == n - 1:
            continue
        for _ in xrange(r - 1):
            x = pow(x, 2, n)
            if x == n - 1:
                break
        else:
            return False
    return True
```

* Assim, tendo em conta a informação dada no enunciado, ("p é primo próximo de 2<sup>512</sup> e q é um primo próximo de 2<sup>513</sup>") descobrimos os valores de ***p*** e ***q***, usando a função acima e uma função auxiliar que percorria todos os números desde os números 2<sup>512</sup> e 2<sup>513</sup> até encontrar um primo

```python
def find_prime(n):
    while True:
        if miller_rabin(n):
            return n
        n += 1
        
p = 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084823
q = 26815615859885194199148049996411692254958731641184786755447122887443528060147093953603748596333806855380063716372972101707507765623893139892867298012169683
```	

* Com isto, conseguimos usar o ***p***, o ***q*** e a ***equação*** dada pelo enunciado ("ed % (p-1)(q-1) = 1") para calculamos o valor de ***d***

```python
d = pow(e, -1, ((p-1)*(q-1)))
```

* Assim, este valor ***d*** pode ser usado para descobrir se os valores de ***n*** e ***e*** são válidos, ou seja, se são os valores que foram usados para cifrar a flag, bastando para isso tentar decifrar ***ciphertext*** elevando a ***d*** e calculando o módulo ***n***

* Com isto, tivemos de utlizar a função ***dec*** e passar como primeiro parâmetro o ***ciphertext*** convertido para bytes e para decimal, operações inversas às que foram feitas pela função ***enc***

```python
flag = unhexlify(b"6434333639326535636564663231393237396263333832656631346333383236636436373834623364393661396164636636336338663964653836646266343635306264653236383061623431666266383463653334343762393666656564633436383136616536336561373562343636356533373230373564316631313864373237613763316435353935386533633935323037346663323461306234363438616164343436336637343664373838636461613836396334343135363135316437396231313431343864313466663437346165633865383561356533303035303964663931623534653839353839343334366665616566343133616133373630303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030303030")
n = 359538626972463181545861038157804946723595395788461314546860162315465351611001926265416954644815072042240227759742786715317579537628833244985694861278987734749889467798189216056224155419337614971247810502667407426128061959753492358794507740889756004921248165191531797899658797061840615258162959755571367021109
e = 65537
p = 13407807929942597099574024998205846127479365820592393377723561443721764030073546976801874298166903427690031858186486050853753882811946569946433649006084823
q = 26815615859885194199148049996411692254958731641184786755447122887443528060147093953603748596333806855380063716372972101707507765623893139892867298012169683
d = pow(e, -1, ((p-1)*(q-1)))
print(dec(flag, d, n))
```	

* Assim descobrimos a flag ***flag{eff249699a91b6056377745baf64417d}***

![flag](images/logbook11-flag.png)