# SQL Injection Attack Lab

## Task 1: Get Familiar with SQL Statements

* Depois de configurarmos o ambiente de desenvolvimento, corremos o comando *dockps* que nos indicou os IDs dos containers que foram `ac3ed20667e0 mysql-10.9.0.6` e `303b990b452e www-10.9.0.5`

* De seguida, corremos o comando *docksh ac3* para entrar no container do mysql

* Na shell do container corremos o comando *mysql -u root -pdess* para interagir com a base de dados

* Após o ponto anterior, corremos *use sqllab_users;* para consultar a base de dados e *show tables;* para ver as tabelas presentes na base de dados, neste caso a tabela *credential*

* Para imprimir a informação de perfil da Alice, fizemos *select * from credential where Name='Alice';* e obtivemos o seguinte resultado:

![ALice](images/logbook8-task1.png)

## Task 2: SQL Injection Attack on SELECT Statement

### Task 2.1: SQL Injection Attack from webpage

* Ao abrimos o url *www.seed-server.com* deparamo-nos com a página de login e tentamos fazer login com o username do *Admin*

* Analisando o código php, verificamos que podíamos realizar um ataque de *SQL Injection* 

```php
    $sql = "SELECT id, name, eid, salary, birth, ssn, phoneNumber, address, email,nickname,Password

    FROM credential

    WHERE name= '$input_uname' and Password='$hashed_pwd'";
```

* Assim, tentamos criar um ataque onde no campo do username escrevemos `Admin' -- `, de maneira a comentar a verificação da palavra passe na *query sql* e fizemos *Login*

* Deste modo, tivemos acesso à informação de todos os utilizadores

![Information](images/logbook8-task2-1.png)

### Task 2.2: SQL Injection Attack from command line

* Após uma tentativa falhada de Login, percebemos no *url* do site que o *login* era feito com recurso ao método *GET*

* Assim, com o comando *curl* conseguimos fazer um ataque de *SQL Injection* colocando o username do Admin desta forma: `Admin' -- `e assim conseguimos obter a informação de todos os utilizadores outra vez


```bash
curl http://www.seed-server.com/unsafe_home.php?username=Admin%27+--+&Password= 
```

![Information2](images/logbook8-task2-2.png)

### Task 2.3: Append a new SQL statement

* Analisando o código, verificamos que só é possível realizar uma *query* de cada vez porque a função de php usada é a *mysqli::query* 

* Assim, isto não permite que o atacante coloque um *;* no fim da primeira *query* para executar uma segunda *query*

* Se a função php fosse do tipo *msqli::multiquery* já seria possível realizar mais que uma *query* e assim realizar operações de *insert*, *delete* e *update*

```php
if (!$result = $conn->query($sql))
```

## Task 3: SQL Injection Attack on UPDATE Statement



