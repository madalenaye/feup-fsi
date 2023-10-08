
# SEED Labs – Environment Variable and Set-UID Program Lab

## Tarefa 1: Manipulação de Variáveis de Ambiente

* Ao usar os comandos ```printenv``` e ```env```, observamos que foram impressas as variáveis de ambiente. Para além disso, ao usar, por exemplo, ```printenv PWD``` ou ```env | grep PWD```, verificamos que obtivemos o mesmo valor que obteríamos com o comando ```pwd```, ou seja, o caminho da localização atual (na verdade, no caso de ```env | grep PWD```, obtivemos os valores de PWD E OLDPWD).

* Ao fazer ```unset PWD``` (seguido de ```env | grep PWD```), verificamos que a variável de ambiente PWD e o seu valor já não apareceram, mas apenas OLDPWD.


## Tarefa 2: Passagem de Variáveis de Ambiente do Processo Pai para o Processo Filho

* No passo 1, observamos que o ficheiro no qual se guardou o output do programa ```myprintenv.c``` continha as variáveis de ambiente do processo filho.

* No passo 2, comentamos a chamada à função ```printenv()``` no caso do processo filho e descomentamos a chamada à mesma função no caso do processo pai. Depois de compilar e correr, observamos que o novo ficheiro continha as variáveis de ambiente do processo pai, que pareciam ter os mesmos valores do processo filho.

* No passo 3, ao usar o comando ```diff``` para comparar os dois ficheiros obtidos anteriormente, verificamos que eles tinham exatamente o mesmo conteúdo, comprovando que as variáveis de ambiente foram herdadas do processo pai para o processo filho.


## Tarefa 3: Variáveis de Ambiente e ```execve()```

* No passo 1, compilando e correndo o programa, observamos que nada (nenhuma variável de ambiente) era impresso na consola, ou seja, o processo atual não tinha acesso às variáveis de ambiente, dado que o terceiro argumento da chamada à função ```execve()``` era NULL.

* Para as variáveis de ambiente serem impressas na consola, no passo 2, mudamos o terceiro argumento da chamada à função ```execve()``` para ```environ```: o apontador para o array das variáveis de ambiente do processo que chama a função. Assim, constatamos que já foram impressas as variáveis de ambiente pretendidas.

* Concluimos que o novo programa obtém as suas variáveis de ambiente através da passagem do valor da variável ```environ``` como terceiro parâmetro da chamada à função ```execve()```, ou seja, respondendo à questão inicial, as variáveis de ambiente **não** são automaticamente herdadas pelo novo programa.


## Tarefa 4: Variáveis de Ambiente e ```system()```

* Compilando e correndo o programa apresentado, verificamos que a chamada ```system("/usr/bin/env")``` fez com que as variáveis de ambiente do processo que a realizou passassem para o novo processo e fossem impressas. De facto, isto acontece porque o comando ```system()```, na verdade, cria uma *shell* na qual executa o comando passado como parâmetro.


## Tarefa 5: Variável de Ambiente e Programas ```Set-UID```

* Conforme esperado, o programa do passo 1 imprimiu todas as variáveis de ambiente do processo atual.

* No passo 2, corremos o comando ```sudo chown root filename``` para mudar o *owner* do ficheiro para *root* e ```sudo chmod 4755 filename``` para mudar as permissões do ficheiro e, neste caso em concreto, torná-lo um programa ```Set-UID```

* No passo 3, corremos os seguintes comandos: ```export PATH=$PATH:/home/seed/Desktop```, ```export LD_LIBRARY_PATH=caminho``` e ```export SEGURANCA=20```. Fazendo ```./filename | grep PATH```, observamos que o o valor da variável de ambiente PATH foi alterado. Correndo ```./filename | grep SEGURANCA```, verificamos que foi criada uma nova variável de ambiente SEGURANCA. Contudo, ao executar ```./filename | grep LD_LIBRARY_PATH``` não obtivemos nenhum resultado. Isto significa que, apesar de ser possível alterar o valor de determinadas variáveis de ambiente (ou criar novas), não foi possível fazê-lo para LD_LIBRARY_PATH. Essa possibilidade criaria uma vulnerabilidade de segurança, pois permitiria que um atacante direcionasse o programa para uma biblioteca maliciosa.


## Tarefa 6: A Variável de Ambiente PATH e Programas ```SET-UID```

* Nesta tarefa, criamos o programa pretendido, mudamos o seu *owner* para ```root``` e tornamo-lo um programa ```Set-UID```. A par disto, experimentamos criar um outro programa chamado "ls" no diretório atual com um código que simplemente imprimia "Hello, World" e alteramos o valor da variável PATH conforme sugerido para se iniciar pelo caminho do diretório atual. Assim, ao correr o programa criado inicialmente, verificamos que, em vez de ser apresentada a listagem dos conteúdos do diretório atual (como seria esperado numa chamada ao comando ```ls```), foi impresso "Hello, World". Ou seja, concluiu-se que é possível fazer com que um programa ```Set-UID``` corra código malicioso criado por um atacante e com privilégio *root*.


# CTF 4

* Ligando-nos ao servidor Linux a explorar, observamos o conteúdo do diretório ```/home/flag_reader``` e, em particular, os ficheiros *admin_note.txt*, *main.c* e *my_script.sh*

* Com a leitura do ficheiro *admin_note.txt*, pudemos deduzir que só teríamos permissões para criar novos ficheiros dentro do diretório ```tmp```

* Através da análise do ficheiro *main.c*, percebemos que a *flag* deveria encontrar-se em ```/flags/flag.txt``` e que a função ```access()``` era chamada com esse caminho como primeiro argumento

* Após vermos o ficheiro *my_script.sh* - que intuimos ser o script corrido regularmente no servidor e, como tal, corrido como *root*, com mais permissões do que o nosso utilizador *flag_reader* - compreendemos que este lia os valores das variáveis de ambiente do ficheiro ```home/flag_reader/env``` e, de seguida, corria o executável ```home/flag_reader/reader```, que depreendemos ser o ficheiro resultante da compilação do *main.c*

* Constatamos ainda (com ```ls -la```) que o ficheiro ```home/flag_reader/env``` era apenas um apontador para o ficheiro ```tmp/env```

* Confirmamos, então, que não tínhamos permissões para aceder a ```/flags/flag.txt```, mas que as tínhamos para criar e editar (ler e escrever) ficheiros no diretório ```tmp```

* Assim, a nossa abordagem passaria por: no diretório ```tmp```, criar um novo ficheiro *new* (```touch new```) com permissões para qualquer utilizador o ler, escrever e executar (```chmod 777 new```), no qual deveria ser escrita a *flag*

* Ora, com base no descoberto anteriormente e numa tentativa de aproveitar a chamada periódica do servidor à função ```access("/flags/flag.txt")```, pensamos em criar uma função ```access()``` com a mesma assinatura da função original, mas cujo objetivo (malicioso) seria ler a *flag* do ficheiro passado como parâmetro e escrevê-la no novo ficheiro *new* criado por nós

* Criamos um código C com esta finalidade e colocamo-lo (com ```echo```) num novo ficheiro *mylib.c*
'''csharp
echo '#include <stdio.h>
    int access(const char *pathname, int mode) {
        char flag[1000];
        FILE *r = fopen("/flags/flag.txt", "rb");
        fgets(flag, sizeof(flag), r);
        fclose(r);
        FILE *w = fopen("/tmp/new.txt", "wb");
        fputs(flag, w);
        fclose(w);
        return 0;
    }
' >> mylib.c
'''

* A vulnerabilidade a explorar para alcançar o nosso objetivo seria abusar da variável de ambiente LD_PRELOAD, isto é, editarmos o ficheiro *env* para conter LD_PRELOAD=..., sendo ... o nome de uma biblioteca partilhada criada por nós a partir do ficheiro *mylib.c* escrito anteriormente

* Isto faria com que a biblioteca partilhada criada e especificada por nós fosse carregada antes de todas as outras, substituindo a chamada à função ```access()``` original pela função ```access()``` em *mylib.c* e, com isso, resultaria no efeito desejado de leitura e escrita da *flag* através do processo que tinha permissões para tal, corrido pelo servidor

* Em particular, corremos ```gcc -fPIC -g -c mylib.c```, ```gcc -shared -o libmylib.so.1.0.1 mylib.o -lc``` e ```echo "LD_PRELOAD=/tmp/libmylib.so.1.0.1" >> env``` para criar a biblioteca partilhada e definir o valor pretendido para a variável de ambiente

* Assim que o script correu no servidor, o ficheiro *new* passou a conter **flag{01ef8d0277833b8af3ac65d5d182f146}** , conforme esperado

