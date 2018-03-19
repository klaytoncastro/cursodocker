# Construindo Imagens

[Estrutura das imagens](#estrutura-das-imagens)

[Dockerfile: instruções, escrita e melhores práticas](#dockerfile)

[Trabalhando com Registros](#trabalhando-com-registros)

[Outros comandos úteis para manipulação de imagens](#outros-comandos-úteis-para-manipulação-de-imagens)

## Estrutura das imagens

Conforme visto na [Unidade 1](../unidade1/unidade1.md), Uma imagem Docker é construída a partir de uma série de camadas (layers), que são combinadas em um único filesystem consistente por meio da tecnologia UnionFS.

Essa estrutura apresenta as seguintes características:

- As camadas são empilhadas, sendo que cada camada é um conjunto das diferenças da camada anterior;

- Quando um novo contêiner é criado, é adicionada uma nova camada gravável sobre as camadas que compõem a imagem original. Essa camada é chamada de "camada de contêiner";

- Todas as alterações feitas no filesystem do contêiner em execução (escrita, alteração e exclusão) são escritas nesta camada de contêiner gravável;

<p align="center">
<img src="images/container-layers.jpg" width=100% title="Estrutura de camadas em imagens Docker">
</p>

- Quando o contêiner é excluído, a camada gravável também é excluída. A imagem subjacente permanece inalterada;

- Como cada contêiner possui sua própria camada de contêiner gravável e todas as alterações são armazenadas nessa camada, vários contêineres podem compartilhar o acesso à mesma imagem subjacente e ainda assim, possuem seu próprio estado de dados;

<p align="center">
<img src="images/sharing-layers.jpg" width=100% title="Compartilhamento das camada de uma imagem Docker">
</p>

- A estratégia utilizada para o compartilhamento e cópia dos arquivos entre as camadas é chamado de **Copy-on-write (CoW)**. 

- O Copy-on-write funciona da seguinte forma:

    - Se um arquivo ou diretório existir em uma camada inferior dentro da imagem, e outra camada (incluindo a camada gravável) necessita de acesso de leitura, ela apenas usa o arquivo existente;

    - Quando um arquivo precisa ser alterado, o arquivo é copiado para a camada gravável, modificado e se torna a única versão visível pelo contêiner.

<p align="center">
<img src="images/overlay_constructs.jpg" width=100% title="Copy-on-write utilizando o Overlay">
</p>

- O **driver de armazenamento (storage driver)** é o componente de software responsável por manipular a forma como as camadas de uma imagem interagem entre si. Estão disponíveis diferentes drivers de armazenamento, que apresentam vantagens e desvantagens em diferentes situações;

- A disponibilidade dos drivers de armazenamento no Linux dependem da distribuição do Linux e versão do Kernel. São exemplos de drivers de armazenamento: `overlay`, `overlay2`, `devicemapper`, `aufs`, `zfs` e `btrfs`;

- O padrão sugerido pelo Docker é o `overlay2` por ser mais performático;

- As camadas da imagem quando baixadas são armazenadas separadamente na pasta `/var/lib/docker/<nome do driver>` em hosts Linux;


##### Para saber mais

> [Documentação Docker - About images, containers, and storage drivers](https://docs.docker.com/engine/userguide/storagedriver/imagesandcontainers/)

> [Documentação Docker - Select a storage driver](https://docs.docker.com/engine/userguide/storagedriver/selectadriver/)

> [Documentação Docker - Use the OverlayFS storage driver](https://docs.docker.com/engine/userguide/storagedriver/overlayfs-driver/#how-the-overlay-driver-works)

## Dockerfile

- Toda imagem é construída a partir de uma imagem base. Essas imagens base podem ser construídas a partir de uma imagem vazia (scratch) ou de outras imagens pré-construídas;

- A solicitação de criação de uma imagem é feita a partir de um `Dockerfile`, via cliente do Docker, com o comando `docker image build`:

```
$ docker image build [-t NOME] .
```

- No comando apresentado acima, após a palavra chave `build` é informado o parâmetro `-t`, no qual é indicado o nome da imagem e em seguida o contexto de construção;

- O **contexto de construção** é o diretório da máquina física do qual o Docker iniciará a construção, e onde estão os arquivos que serão utilizados nela (inclusive o  Dockerfile), nesse caso, é o diretório corrente (indicado pelo ".");

- Na construção da imagem, o comando build permite a utilização do parâmetro `-f` para que seja indicado o Dockerfile com outro nome ou localizado em outro lugar do sistema de arquivos.

```
$ docker image build -f /caminho/para/o/Dockerfile .
```

- Assim, a primeira coisa que um processo de construção faz é enviar todo o contexto (recursivamente) para o daemon. Antes que o daemon Docker execute as instruções do  Dockerfile, ele executa uma validação preliminar e retorna um erro se a sintaxe estiver incorreta;

- O daemon do Docker executa as instruções no Dockerfile uma a uma, gravando o resultado de cada instrução em uma nova imagem intermediária e, ao final, informar o identificador da sua nova imagem;

- Sempre que possível, o Docker irá reutilizar as imagens intermediárias (também chamadas de `cache`), para acelerar, significativamente, o processo de criação da imagem;

- Para aumentar o desempenho da compilação, exclua arquivos e diretórios adicionando um arquivo `.dockerignore` ao diretório de contexto;

- Antes que o cliente do Docker envie o contexto para o daemon docker, ele procura o arquivo chamado `.dockerignore` no diretório raiz do contexto. Se esse arquivo existir, o cliente modifica o contexto para excluir arquivos e diretórios que combinam com os padrões definidos nele;

Exemplo

```bash
# comentário
*/temp*
*/*/temp*
temp?
```

- O cliente considera a raiz do contexto para procurar os padrões definidos;

### Escrita de Dockerfile

- O Dockerfile é um arquivo texto simples, normalmente nomeado de `Dockerfile` e localizado na raiz do contexto de construção.

- Um Dockerfile contém as instruções executadas pelo daemon do Docker para a criação de uma imagem.

- As instruções são executadas uma por vez na ordem em que são escritas no Dockerfile.

Exemplo

```Dockerfile
# Imagem base
FROM alpine:3.5

# Instalação python and pip
RUN apk add --update py2-pip

# upgrade pip
RUN pip install --upgrade pip

# instalação dos módulos Python necessários para a aplicação
COPY requirements.txt /usr/src/app/
RUN pip install --no-cache-dir -r /usr/src/app/requirements.txt

# cópia dos arquivos requeridos para a execução da aplicação
COPY app.py /usr/src/app/
COPY templates/index.html /usr/src/app/templates/

# informa o número da porta do contêiner que deve ser exposta
EXPOSE 5000

# executa a aplicação
CMD ["python", "/usr/src/app/app.py"]
```

### Instruções

- Como já visto, as **instruções** são ações para a composição de uma imagem, tais como, executar um comando de linha, adicionar um arquivo ou pasta, criar uma variável de ambiente;

- Cada instrução cria uma nova camada e a adiciona à imagem;

- As instruções em um Dockerfile são escritas uma por linha, sendo que primeiro deve estar a instrução, e em seguida seus argumentos;

```Dockerfile
#Comentários
RUN echo "Esse é um exemplo de instrução"
```
### Instruções Relevantes para Composição de Dockerfiles

<!--|Instrução|Objetivo|
| --- | --- |
|FROM|A instrução FROM define a imagem base a partir da qual a nova imagem será criada. Essa deve ser a primeira instrução no Dockerfile. |
|RUN|A instrução RUN irá executar quaisquer comandos em uma nova camada criada na parte superior da imagem atual e gravar o resultado final. A imagem resultante será usada pela próxima instrução do Dockerfile.|
|CMD|O objetivo principal da instrução CMD é fornecer o processo padrão para um contêiner de execução. O CMD define o comando a ser processado ao executar a imagem. Esse processo padrão pode ser um software executável. Caso o contêiner execute o mesmo executável sempre, é recomendado utilizar a instrução ENTRYPOINT em combinação com CMD. Se o usuário especificar argumentos para a execução do Docker, eles substituirão o padrão especificado no CMD.|
|EXPOSE|A instrução EXPOSE informa ao Docker que o contêiner escuta nas portas de rede especificadas em tempo de execução. Essa instrução não torna as portas do contêiner acessíveis ao host. Para fazer isso,  deve-se usar o sinalizador `-p` para publicar um intervalo de portas ou o sinalizador `-P` para publicar todas as portas expostas. Pode ser exposto um número de porta e publicá-lo externamente em outro número.|
|ENV|A instrução ENV define uma variável de ambiente por meio de um nome e um valor. Este valor estará disponível no ambiente para todos as instruções subsequentes, Dockerfiles que tiverem essa imagem como base e no contêiner executado a partir dela.  Os valores dessas variáveis podem substituídos em tempo de construção e de execução.|
|ADD/COPY|As instruções ADD e COPY copiam novos arquivos ou diretórios e os adiciona ao sistema de arquivos da imagem no caminho especificado. Vários recursos podem ser especificados, mas se forem arquivos ou diretórios, eles devem ser relativos ao diretório de origem que está sendo construído (o contexto de compilação). O que diferencia as duas instruções é que o COPY é mais performático e o ADD pode também fazer cópias a partir de URLs de arquivos remotos.|
|ENTRYPOINT|Uma instrução ENTRYPOINT permite configurar um contêiner que será executado como um executável. Os argumentos passados para o comando `docker container run <imagem>` serão anexados após todos os elementos do ENTRYPOINT e substituirão todos os elementos especificados usando CMD. Isso permite que os argumentos sejam passados para o executável definido no ENTRYPOINT. A instrução ENTRYPOINT pode ser substituído usando a opção `--entrypoint` na inicialização do contêiner. Somente a última instrução ENTRYPOINT no Dockerfile terá efeito.|
|VOLUME|A instrução VOLUME cria um ponto de montagem com o nome especificado e marca-o como um volume que pode ser montado externamente no host e por outros contêineres.|
|USER|A instrução USER define o nome de usuário ou identificação de usuário (UID) para execução de quaiquer instruções RUN, CMD e ENTRYPOINT que virão a seguir no  Dockerfile. Esse será o usuário de execução do contêiner.
|WORKDIR|A instrução WORKDIR define o diretório de trabalho para quaisquer instruções RUN, CMD, ENTRYPOINT, COPY e ADD que o sigam no Dockerfile. Se o WORKDIR não existir, ele será criado.|-->

<!--Quando uma imagem é atualizada ou recriada, somente as camadas afetadas são atualizadas. Esse comportamento se repete quando as imagens são distribuídas, ou seja, é necessário transferir somente as camadas atualizadas de uma imagem já presente em um {\it host}. O mecanismo do Docker determina quais camadas precisam ser atualizadas em tempo de execução.-->


#### FROM

A instrução `FROM` define a imagem base a partir da qual a nova imagem será criada. Essa deve ser a primeira instrução no Dockerfile.

Não é necessário realizar o download da imagem indicada no `FROM`, o daemon do Docker ao identificar que a imagem não existe realizará o pull automaticamente.

Sintaxe
```Dockerfile
FROM <image>[:<tag>]
```

Exemplo
```Dockerfile
FROM alpine:3.5
```

#### RUN

A instrução `RUN` irá executar quaisquer comandos em uma nova camada criada na parte superior da imagem atual e gravar o resultado final. A imagem resultante será usada pela próxima instrução do Dockerfile.

Sintaxe
```Dockerfile
#Shell form (executado pelo shell /bin/sh -c no Linux ou cmd /S /C no Windows)
RUN <comando>

#Exec form (não é processado pelo shell, portanto não faz substituição de variáveis)
RUN ["executavel", "param1", "param2"]
```
Exemplo
```Dockerfile
RUN wget -O - https://www.site
```

Para tornar o Dockerfile mais legível, compreensível e sustentável, divida instruções `RUN` longas ou complexas em várias linhas separadas com barras invertidas (\).

Exemplo
```Dockerfile
RUN apt-get update && apt-get install -y \
    package-bar \
    package-baz \
    package-foo
```

#### CMD

O objetivo principal da instrução `CMD` é fornecer o processo padrão para a execução de um contêiner.

O `CMD` define o comando a ser processado ao executar a imagem. Esse processo padrão pode ser um software executável.

Caso o contêiner execute o mesmo executável sempre, é recomendado utilizar a instrução `ENTRYPOINT` em combinação com `CMD`.

Se o usuário especificar argumentos para a execução do Docker, eles substituirão o padrão especificado no `CMD`.

Sintaxe
```Dockerfile
#Exec form (não é processado pelo shell)
CMD ["executavel","param1","param2"]

#Shell form (executado por /bin/sh -c)
CMD comando param1 param2

#Define parâmetros default para o ENTRYPOINT
CMD ["param1","param2"]
```

Exemplo
```Dockerfile
CMD ["python", "/usr/src/app/app.py"]

#ou

CMD apache2 -DFOREGROUND
```
#### ENTRYPOINT

Uma instrução `ENTRYPOINT` permite configurar um contêiner que será executado como um executável.

Os argumentos passados para o comando `docker container run <imagem>` serão anexados após todos os elementos do `ENTRYPOINT` e substituirão todos os elementos especificados usando `CMD`. Isso permite que os argumentos sejam passados para o executável definido no `ENTRYPOINT`.

A instrução `ENTRYPOINT` pode ser substituído usando a opção `--entrypoint` na inicialização do contêiner. Somente a última instrução `ENTRYPOINT` no Dockerfile terá efeito.

Sintaxe
```Dockerfile
#Exec form
ENTRYPOINT ["executavel", "param1", "param2"]

#Shell form
ENTRYPOINT comando param1 param2
```

Exemplo
```Dockerfile
#Exec form
ENTRYPOINT ["top", "-b"]

#Shell form
ENTRYPOINT exec top -b
```

Na forma de shell o `ENTRYPOINT` é iniciado utilizando o subcomando de `/bin/sh -c`, o que não permite o recebimento de sinais pelo sistema operacional. Dessa forma, o processo não receberá o sinal de finalização (`SIGTERM`) do comando `docker container stop <container>`.

Nessa forma shell, também, serão ignoradas quaisquer instruções `CMD` ou argumentos passados via comando `docker container run`.

#### Como o `CMD` e o `ENTRYPOINT` interagem

As instruções CMD e ENTRYPOINT definem qual comando é executado ao executar um contêiner. Existem poucas regras que descrevem sua cooperação.

Até este ponto, discutimos como usar ENTRYPOINT ou CMD para especificar o executável padrão da sua imagem. No entanto, existem alguns casos em que faz sentido usar ENTRYPOINT e CMD juntos.

A combinação de ENTRYPOINT e CMD permite que você especifique o executável padrão para sua imagem ao mesmo tempo que fornece argumentos padrão para aquele executável que pode ser substituído pelo usuário. Vejamos um exemplo:

- O Dockerfile deve especificar pelo menos um dos comandos `CMD` ou `ENTRYPOINT`;

- A instrução `ENTRYPOINT` deve ser definida ao usar o contêiner como um executável;

- O `CMD` deve ser usado como uma forma de definir argumentos padrão para um comando `ENTRYPOINT` ou para executar um comando ad-hoc em um contêiner;

- O `CMD` será substituído ao executar o contêiner com argumentos alternativos.

<!--Exemplo
```Dockerfile
FROM ubuntu:trusty
ENTRYPOINT ["/bin/ping","-c","3"]
CMD ["localhost"]
```-->
**Combinações entre as formas das instruções `CMD` e `ENTRYPOINT`**

|Dockerfile|Resultado|Funciona|
|---|---|---|
|ENTRYPOINT /bin/ping -c 3 <br> CMD localhost|/bin/sh -c '/bin/ping -c 3' /bin/sh -c localhost|NÃO|
|ENTRYPOINT ["/bin/ping","-c","3"] <br> CMD localhost|/bin/ping -c 3 /bin/sh -c localhost|NÃO|
|ENTRYPOINT /bin/ping -c 3 <br> CMD ["localhost"]|/bin/sh -c '/bin/ping -c 3' localhost|NÃO|
|ENTRYPOINT ["/bin/ping","-c","3"] <br> CMD ["localhost"]|/bin/ping -c 3 localhost|SIM|

<!--docker inspect $(docker ps -lq)
cd /flavioh/git/exemplos/entrycmd-->

A instrução `ENTRYPOINT` pode ser usada em combinação com um script auxiliar, quando for necessário executar múltiplos passos para iniciar o contêiner.

Exemplo

```Bash
# Script utilizado no Entrypoint pela imagem oficial do Postgres

#!/bin/bash
set -e

if [ "$1" = 'postgres' ]; then
    chown -R postgres "$PGDATA"

    if [ -z "$(ls -A "$PGDATA")" ]; then
        gosu postgres initdb
    fi

    exec gosu postgres "$@"
fi

exec "$@"
```



```Dockerfile
# Uso do script no Entrypoint
ENTRYPOINT ["/docker-entrypoint.sh"]
```

#### EXPOSE

A instrução `EXPOSE` informa ao Docker que o contêiner escuta nas portas de rede especificadas em tempo de execução.

Essa instrução não torna as portas do contêiner acessíveis ao host. Para fazer isso,  deve-se usar o sinalizador `-p` para publicar um intervalo de portas ou o sinalizador `-P` para publicar todas as portas expostas.

Pode ser exposto um número de porta e publicá-lo externamente em outro número.

Sintaxe
```Dockerfile
EXPOSE <porta>
```

Exemplo
```Dockerfile
EXPOSE 5432
```

#### ENV

A instrução `ENV` define uma variável de ambiente por meio de um nome e um valor.

Este valor estará disponível no ambiente para todos as instruções subsequentes, Dockerfiles que tiverem essa imagem como base e no contêiner executado a partir dela.

Os valores dessas variáveis podem substituídos em tempo de construção e de execução.

Sintaxe
```Dockerfile
ENV <chave> <valor>

#ou

ENV <chave>=<valor> ...
```

Exemplo
```Dockerfile
ENV myName="John Doe" myCat=fluffy

#ou

ENV myName John Doe
ENV myCat fluffy
```

#### COPY

A instrução `COPY` copia novos arquivos ou diretórios e os adiciona ao sistema de arquivos do contêiner no caminho especificado.

Sintaxe
```Dockerfile
COPY <origem>... <destino>

#ou 

COPY ["<origem>",... "<destino>"]
```

Exemplo
```Dockerfile
COPY requirements.txt /usr/src/app/
```

O `<destino>` é um caminho absoluto, ou um caminho relativo ao `WORKDIR`, no qual o arquivo de origem será copiado dentro do contêiner de destino.

Exemplo
```Dockerfile
COPY teste diretorioRelativo/   # adiciona o arquivo "teste" para `WORKDIR`/diretorioRelativo/
COPY teste /diretorioAbsoluto/  # adiciona "teste" to /diretorioAbsoluto/
```

A instrução `COPY` obedece as seguintes regras:

- O caminho `<origem>` deve estar dentro do contexto de construção. Os caminhos de arquivos e diretórios serão interpretados como relativos a partir da raiz do contexto de construção;

- Se `<origem>` for um diretório, todo o conteúdo do diretório será copiado, incluindo metadados do sistema de arquivos. O próprio diretório não é copiado, apenas seu conteúdo;

- Se `<origem>` for qualquer outro tipo de arquivo, ele é copiado individualmente juntamente com seus metadados. Nesse caso, se `<destino>` terminar com uma barra diagonal (`/`) será considerado um diretório e `<origem>` será copiado dentro dessa pasta;

- A `<origem>` pode ser definida utilizando caracteres curingas (`*`,`?`,`[`,`]`);

- Se vários recursos `<origem>` forem especificados, diretamente ou devido ao uso de um curinga, então `<destino>` deve ser um diretório, e ele deve terminar com uma barra diagonal (`/`);

- Se `<destino>` não terminar com uma barra diagonal, ele será considerado um arquivo regular e o conteúdo de `<origem>` será escrito em `<destino>`;

- Se `<destino>` não existir, ele é criado juntamente com todos os diretórios que faltam em seu caminho.


Todos os novos arquivos e diretórios são criados com um UID e GID de 0, a menos que seja utilizada a opção `--chown=<user>:<group>`. Essa opção especifica um determinado nome de usuário e grupo ou UID / GID para definir a propriedade do conteúdo copiado.

Se um nome de usuário ou grupo for fornecido, os arquivos `/etc/passwd` e `/etc/group` do sistema de arquivos do contêiner serão usados para executar a tradução dos nomes para os UID ou GID, respectivamente. 

Exemplo
```Dockerfile
COPY --chown=elasticsearch:elasticsearch arquivos* /usr/share/elasticsearch/

#ou

COPY --chown=501:501 standalone-ha.xml $JBOSS_HOME/standalone/configuration
```

#### ADD

A instrução `ADD` copia novos arquivos, diretórios ou URLs de arquivos remotos de `<origem>` e os adiciona ao sistema de arquivos da imagem no caminho `<destino>`.

Sintaxe
```Dockerfile
ADD <src>... <dest>

#ou

ADD ["<src>",... "<dest>"]
```

Exemplo
```Dockerfile
ADD hom* /dir/

ADD http://example.com/foobar /
```

Observações sobre o uso da instrução `ADD`:

- O `ADD` segue as mesmas regras definidas para o `COPY`;

- Se um arquivo compactado for adicionado a partir de um caminho local, ele será descompactado automaticamente. Arquivos obtidos de URLs remotas não são descompactados;

- Por ser mais performático é mais indicada a utilização da instrução `COPY` para a cópia de arquivos e diretórios do contexto de construção e instruções `RUN` com `curl` ou `wget` para baixar recursos remotos (o que permite processar e excluir o download na mesma instrução).

#### VOLUME

A instrução `VOLUME` cria um ponto de montagem com o nome especificado e marca como um volume. Esse volume pode ser montado externamente no host e por outros contêineres.

O comando `docker container run` inicializa o volume recém-criado com todos os dados que existem no local especificado na imagem de base.

Sintaxe
```Dockerfile
VOLUME <diretório>
```

Exemplo
```Dockerfile
VOLUME ["/data"]
```

#### USER

A instrução `USER` define o nome de usuário ou identificação de usuário (UID) para execução de quaiquer instruções `RUN`, `CMD` e `ENTRYPOINT` que virão a seguir no  Dockerfile. Esse será o usuário de execução do contêiner.

Sintaxe
```Dockerfile
USER <usuário>[:<grupo>]

#ou

USER <UID>[:<GID>]
```

Exemplo
```Dockerfile
USER elasticsearch
```

#### WORKDIR

A instrução `WORKDIR` define o diretório de trabalho para quaisquer instruções `RUN`, `CMD`, `ENTRYPOINT`, `COPY` e `ADD` que o sigam no Dockerfile. 

Se o `WORKDIR` não existir, ele será criado mesmo que não seja usado em nenhuma instrução posterior do Dockerfile.

A instrução `WORKDIR` pode ser usada várias vezes em um Dockerfile. Se for fornecido um caminho relativo, será relativo ao caminho da instrução `WORKDIR` anterior.

A instrução `WORKDIR` pode resolver variáveis de ambiente previamente definidas usando o `ENV`. Somente podem ser utilizadas variáveis de ambiente explicitamente configuradas no Dockerfile.

Sintaxe
```Dockerfile
WORKDIR /<diretório>
```

Exemplo
```Dockerfile
WORKDIR /usr/local
```

### Exemplo

O propósito desse exemplo é criar uma nova imagem de uma aplicação desenvolvida em **Python** e **Flask** (micro-framework web para Python). Será utilizada como base a imagem oficial do [Python](https://hub.docker.com/_/python/). 

#### Preparação do ambiente

1. Crie na sua máquina um diretório com o nome `mensagem`

2. Crie o arquivo `requirements.txt` com o seguinte conteúdo:

```python
Flask==0.11.1
pyfiglet==0.7.5
```

3. Dentro desse diretório, crie outro diretório com o nome `app` e crie o arquivo `main.py` com o seguinte conteúdo:

```python
from pyfiglet import Figlet
import os
from flask import Flask

app = Flask(__name__)

font = Figlet(font="starwars")

@app.route("/")
def main():
    # get the message from the environmental variable $MESSAGE
    # or fall back to the string "no message specified"
    message = os.getenv("MESSAGE", "no message specified")

    # render plain text nicely in HTML
    html_text = font.renderText(message)\
            .replace(" ","&nbsp;")\
            .replace(">","&gt;")\
            .replace("<","&lt;")\
            .replace("\n","<br>")

    # use a monospace font so everything lines up as expected
    return "<html><body style='font-family: mono;'>" + html_text + "</body></html>"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=80)
```


#### Criação do Dockerfile e construção da imagem

1. Crie o arquivo com o nome `Dockerfile` dentro do diretório `mensagem` com o seguinte conteúdo:

```Dockerfile
FROM python:2.7
# copia o arquivo requirements.txt
COPY requirements.txt /tmp/

# executa o upgrade do pip e instala os pacotes python necessários
RUN pip install -U pip
RUN pip install -r /tmp/requirements.txt

# copia a pasta com o código da aplicação
COPY ./app /app

# define a variável de ambiente MESSAGE que a aplicação vai utilizar e mostrar
ENV MESSAGE "hello from Docker"

CMD python app/main.py

```

2. Em seguida construa a imagem com o seguinte comando:

```bash
$ docker image build -t mensagem .
```

3. Execute um contêiner a partir da imagem criada passando um valor para a variável de ambiente `MESSAGE` e acesse o endereço `http://<IP>`:

```bash
$ docker container run -p 80:80 -e MESSAGE="Docker em ação!" mensagem
```

##### Para saber mais

> [Referência do Dockerfile](https://docs.docker.com/engine/reference/builder/)


### Melhores práticas

#### Os contêineres devem ser descartáveis

O contêiner produzido pela imagem que seu Dockerfile define deve ser tão descartável quanto possível. Entenda por "descartável" que o contêiner pode ser interrompido e destruído e um novo construído e colocado no lugar, com um mínimo absoluto de configuração.

#### Utilize imagens-base mínimas

A estratégia de imagem mínima em imagens-base de sistemas operacionais indica que se mantenha por padrão, em imagens dessa natureza,somente bibliotecas estritamente necessárias para o funcionamento do sistema. O uso dessa estratégia visa diminuir ao máximo o tamanho da imagem do sistema operacional e, consequentemente, o tamanho das imagens construídas a partir dela.

Tendo em vista esse princípio, tornou-se muito popular a utilização da imagem do sistema operacional **Alpine Linux**`. 

Alpine Linux é uma distribuição Linux completa baseada no BusyBox. A imagem tem apenas 5 MB de tamanho e possui seu próprio gerenciador de pacotes (`apk`). O Alpine se destaca também pelo amplo repositório de pacotes e segurança.

#### Evitar instalação de pacotes desnecessários

Para reduzir a complexidade, as dependências, o tempo de construção e o tamanho da imagem, deve se evitar a instalação de pacotes extras ou desnecessários para o propósito da imagem. Por exemplo, instalar um cliente de email em uma imagem de banco de dados.

#### Cada contêiner com apenas um propósito

Desacoplar aplicações em vários contêineres torna muito mais fácil escalar horizontalmente e reutilizar contêineres. Por exemplo, uma pilha de aplicativos da Web pode consistir em três contêineres separados, cada um com sua própria imagem única, para gerenciar a aplicação web, banco de dados e um cache na memória de forma desacoplada. Utilize a rede do Docker para estabelecer a comunicação entre os contêineres.

#### Utilizar um arquivo `.dockerignore`

Conforme já visto, a utilização de um arquivo `.dockerignore` para omitir o envio de arquivos desnecessários do contexto de construção, permite minimizar o tempo de compilação e tamanho final da imagem.

Observe a mensagem na saída de console, quando executado o comando `docker image build`, para conferir o tamanho do contexto de construção:

```
Sending build context to Docker daemon  187.8MB
```

#### Utilizar o cache

Durante o processo de construção de uma imagem, o daemon do Docker percorre as instruções do Dockerfile executando cada uma delas na ordem especificada. Para cada instrução examinada, o daemon do Docker procura uma imagem existente em seu cache que pode ser reutilizada, em vez de criar uma nova imagem duplicada.

As regras utilizadas são as seguintes:

- Iniciando com a imagem principal que já está no cache, a próxima instrução é comparada com todas as imagens secundárias derivadas dessa imagem base para identificar se alguma foi criada com a mesma instrução. Em caso positivo a imagem em cache é utilizada, caso contrário, o cache é invalidado e uma nova imagem é construída a partir da instrução;

- Para as instruções `ADD` e `COPY`, o conteúdo do(s) arquivo(s) na imagem é examinado e uma soma de verificação é calculada para cada arquivo. Durante a pesquisa do cache, a soma de verificação é comparada com a soma de verificação nas imagens existentes. Se algo mudou no(s) arquivo(s), como o conteúdo e os metadados, o cache é invalidado;

- Uma vez que o cache é invalidado, todos as instruções subseqüentes do Dockerfile geram novas imagens e o cache não é usado.

Para que o cache não seja utilizado na construção, a opção `--no-cache=true` deve ser utilizada no comando de construção.

##### Para saber mais

> [Melhores práticas para escrita de Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/)


## Trabalhando com Registros

Conforme visto na [Unidade 1](../unidade1/unidade1.md), um Registro Docker (registry) é um repositório para armazenamento e distribuição de imagens previamente criadas.

Os Registros Docker são usados para integrar o armazenamento e distribuição de imagens no fluxo de trabalho interno de desenvolvimento e operacionalizar o processo de entrega contínua das aplicações em contêineres.

O download e upload dessas imagens pode ser realizado utilizando os comandos `docker image pull` e `docker image push`, conforme abaixo:

Sintaxe
```
#Baixa imagem do registry
$ docker image pull [OPÇÕES] [REGISTRO PRIVADO/] NOME [:TAG]

# Sobe imagem para o registry
$ docker image push [OPÇÕES] [REGISTRO PRIVADO/] NOME [:TAG]
```

Há um sistema hierárquico para o armazenamento de imagens:

- **Registro:** serviço responsável por hospedar e distribuir imagens;

- **Repositório:** conjunto de imagens relacionadas (geralmente fornecendo diferentes versões do mesmo aplicativo ou serviço);

- **Tag:** identificador alfanumérico anexado às imagens de um repositório (por exemplo, `14.04` ou `stable`). Caso não seja atribuída nenhuma tag, será atribuída a tag padrão `latest`.


O registro padrão é o **Docker Hub**, serviço público fornecido pela Docker Inc. Para realizar o push das imagens para esse registro é necessário realizar um cadastro no serviço.

Os registros podem ser criados dentro de um ambiente tecnológico privado. A própria Docker fornece uma [imagem de registry](https://hub.docker.com/_/registry/) que pode ser utilizada com esse propósito, bem como uma solução corporativa, na sua versão Enterprise, chamada [Docker Trusted Registry](https://docs.docker.com/datacenter/dtr/2.4/guides/).

Outras empresas também fornecem registros para imagens Docker, tais como o [CoreOS Quay](https://coreos.com/quay-enterprise/), [GitLab Container Registry](https://docs.gitlab.com/ce/user/project/container_registry.html), [Google Cloud Plataform](https://cloud.google.com/container-registry/?hl=pt-br) e [Amazon Elastic Container Registry
](https://aws.amazon.com/pt/ecr/).

Para realizar o push da imagem é necessário identificar a imagem em um repositório nomeado apropriadamente e usar o comando `docker image push` para fazer o upload no registro.

Essa identificação pode ser feita por meio da opção `-t` do comando `docker image build` ou do comando `docker image tag`, conforme especificado abaixo.

Sintaxe
```
$ docker image tag IMAGEM_ORIGEM[:TAG] [REGISTRO PRIVADO/] IMAGEM [:TAG]
```

Exemplo
```
$ docker image tag nginx:1.13.8-alpine registry.stf.jus.br/nginx:1.13.8
```

Com o comando `docker image tag`, a imagem de origem pode ser referenciada pelo seu nome ou ID.

### Exemplo

Nesse exemplo, realizaremos o push da imagem `mensagem`, criada no exemplo anterior, para o Docker Hub.

1. Crie um Docker ID, caso ainda não possua no [Docker Hub](https://hub.docker.com/).

2. Execute o comando `docker login` e informe suas credenciais.

3. Identifique a imagem com o repositório (que no caso do Docker Hub é o seu próprio nome de usuário).

```
$ docker image tag mensagem flavioh/mensagem
```

4. Realize o push da imagem para o Docker Hub

```
$ docker image push flavioh/mensagem
```

##### Para saber mais

> [About Docker Registry](https://docs.docker.com/registry/introduction/)

> [Documentação Docker - docker image tag](https://docs.docker.com/engine/reference/commandline/image_tag/)

## Outros comandos úteis para manipulação de imagens

### Criação de imagem a partir de um contêiner

O comando `docker container commit` permite criar uma imagem a partir de um contêiner já instanciado.

Embora essa operação possa ser útil, é preferível a utilização de Dockerfiles, pois torna a criação e manutenção de imagens mais repetível, documentada e sustentável.

Por padrão os contêineres são pausados antes da criação da imagem, isso reduz a probabilidade de corrupção de dados durante o processo de commit. A operação não incluirá quaisquer dados contidos nos volumes montados dentro do contêiner.

A opção `--change` permite aplicar novas configurações à imagem que será criada. As instruções de Dockerfile suportadas são: `CMD`, `ENTRYPOINT`, `ENV`, `EXPOSE`,`LABEL`,`ONBUILD`, `USER`,`VOLUME` e `WORKDIR`.

```
$ docker container commit [OPÇÕES] CONTAINER [IMAGEM[:TAG]]

Opções:

  --change , -c   Aplica mudanças de instruções Dockerfile na imagem criada
  --message , -m  Mensagem de commit
  --pause , -p    Pausa o contêiner durante o commit (default true)

```

Exemplo:
```
# Cria nova imagem a partir de um contêiner em execução

$ docker container commit c3f279d17e0a  imagemteste:v3

# Cria nova imagem de um contêiner criando uma nova variável de ambiente

docker container commit --change "ENV DEBUG true" c3f279d17e0a  imagemteste:v3

# Cria nova imagem de um contêiner modificando as instruções CMD e EXPOSE

$ docker container commit --change='CMD ["apachectl", "-DFOREGROUND"]' -c "EXPOSE 80" c3f279d17e0a imagemteste:v4

```

### Salvar e carregar imagens utilizando arquivos tar

O comando `docker image save` salva uma ou mais imagens em um arquivo tar (por default a saída é direcionada para STDOUT). A opção `-o` ou `--output` serve para indicar um arquivo para gravação.

Sintaxe
```
$ docker image save [--output|-o] IMAGEM [IMAGEM...]
```

Exemplo
```
$ docker image save -o /tmp/redis.tar redis:latest
```

O arquivo tar resultante pode ser carregado com o comando `docker image load`.

Sintaxe
```
$ docker image load [--input|-i]
```

Exemplo
```
$ docker image load -i /tmp/redis.tar
```

##### Para saber mais

> [Documentação Docker - docker container commit](https://docs.docker.com/engine/reference/commandline/container_commit/)

> [Documentação Docker - docker image save](https://docs.docker.com/engine/reference/commandline/image_save/)

> [Documentação Docker - docker image load](https://docs.docker.com/engine/reference/commandline/image_load/)

## Let's play! 
>[Playground 4](playground/play4.md)

---
<p align="left">
<a href='../unidade2/unidade2.md' id='unidade2' class='anchor' aria-hidden='true'><< Unidade 2 - Ciclo de vida de Contêineres</a></p>
<p align="right">
<a href='../unidade4/unidade4.md' id='unidade4' class='anchor' aria-hidden='true'>Unidade 4 - Persistindo dados com Volumes >></a></p>