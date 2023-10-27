
# CTF 3

## Identificação 

* A vulnerabilidade encontrada foi a **CVE-2021-34646**: WordPress Plugin WooCommerce Booster Plugin 5.4.3 - Authentication Bypass. 

* A falha permite que os invasores se façam passar por utilizadores e acionem uma verificação de endereço de *e-mail* para contas arbitrárias - incluindo contas administrativas - e façam *login* automaticamente como esse utilizador, incluindo qualquer administrador do site. 

* Assim, a flag introduzida foi **flag{CVE-2021-34646}**.

## Processo 

* Começamos por recolher informação que nos ajudasse a identificar a vulnerabilidade, desde a procura de CVEs que afetassem a versão do Wordpress mencionado no link fornecido pelo guião até aos *plugins* existentes. 

* No site fornecido, encontrámos na *tab* “Shop”, na secção “Additional Information”, que as versões do *software* são: 

    * WordPress - 5.8.1 

    * WooCommerce plugin - 5.7.1 

    * Booster for WooCommerce plugin - 5.4.3 

* Alguns dos utilizadores a comentar são: 

    * Admin 

    * Orval Sanford 

* Ao procurar nas bases de dados e no motor de pesquisa por CVE’s relacionados com estas versões, encontramos uma vulnerabilidade referente ao Booster for WooCommerce plugin - 5.4.3, a **CVE-2021-34646**.

* No site https://www.exploit-db.com/exploits/50299, descobrimos um *exploit* para a vulnerabilidade identificada.

* Para a segunda parte da CTF deparamo-nos com algumas dificuldades, no sentido em que demoramos algum tempo a perceber que para dar *exploit* da vulnerabilidade era necessário correr o código fornecido pelo site acima mencionado. No entanto, depois de termos percebido isso, tudo correu de forma tranquila. 

* Corremos o código com o primeiro argumento sendo o *link* do *website* e o segundo sendo o id *default* (o id do admin, que é 1). 

* Corrido o código, foi-nos dado um *link* como *output* que, depois de inserido no *website* dado na CTF, tornou possível entrar com a conta de administrador. 

* Abrimos a *tab* de mensagens e encontramos uma mensagem com a flag: **flag{please don't bother me}**.