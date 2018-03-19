# Integrando Contêineres com Docker Compose
[Funcionamento e características](#funcionamento-e-características)

[Instalação do Docker Compose](#instalação-do-docker-compose)

[Compose files](#compose-files)

<!--p align="center">
<img src="images/docker-compose.png" width=30% title="Docker Compose">
</p-->

## Funcionamento e características

Docker Compose é uma ferramenta para definição e execução de múltiplos containers Docker, que funcionem de forma integrada para compor uma aplicação.

O Docker Compose se utiliza de um arquivo de definição em formato YAML (`docker-compose.yaml`) para configuração dos parâmetros necessários para a execução de contêineres, sejam eles informações de construção, exposição de portas, variáveis de ambientes, etc.

Os contêineres são especificados como **serviços** que funcionam de forma integrada, sendo possível gerenciá-los em conjunto pela linha de comando.

O Docker Compose permite o gerenciamento do ciclo de vida da aplicação por meio do comando `docker-compose`. Dentre às operações possíveis, temos:

- Iniciar, parar e reconstruir serviços
- Exibir o status dos serviços em execução
- Transmitir a saída de log da execução de serviços
- Executar um comando único em um serviço


O Docker Compose permite a configuração do ambiente de execução desses serviços, incluindo rede, volumes e dependências entre eles.

### Vantagens do Docker Compose

#### Múltiplos ambientes isolados em um único host

O Compose permite a criação de diversos ambientes para uma aplicação no mesmo host.

Esses ambientes são isolados uns dos outros, o que possibilita:

- Isolar aplicações que utilizem os mesmos nomes de serviços;
- A execução de diferentes versões de uma mesma aplicação em um mesmo host;
- Em um servidor de integração contínua, para evitar que as construções interfiram entre si.

Esses ambientes são definidos por meio do nome do projeto. O nome padrão do projeto é o nome do seu diretório.

Um nome de projeto personalizado pode ser feito utilizando a opção `-p` do comando `docker-compose` ou a variável de ambiente `COMPOSE_PROJECT_NAME`.

### Preservação de dados de volume quando os contêineres são criados

O Docker Compose preserva todos os volumes usados ​​por seus serviços. 

Quando o comando `docker-compose up` é executado, caso existam contêineres executados anteriormente, os volumes do contêiner antigo são copiados para o novo. Esse processo garante que os dados criados nos volumes não sejam perdidos.

### Recriação somente de contêineres modificados

O Compose armazena em cache a configuração usada para criar um contêiner. 

Quando um serviço que não sofreu mudanças é reiniciado são reutilizados os contêineres existentes. Essa forma de funcionamento permite a realização de mudanças mais rápidas no ambiente.

### Utilização de variáveis na composição de ambientes

O arquivo de definição do Docker Compose suporta a utilização de variáveis. 

Essa variáveis permitem personalizar configurações de uma aplicação (arquivos, usuários, senhas,...) para execução em diferentes ambientes, tais como homologação e testes.

Além disso, o Docker Compose possibilita que o arquivo de configurações seja estendido, o que torna possível dividir as configurações em múltiplos arquivos.

##### Para saber mais

> [Documentação Docker - Overview of Docker Compose](https://docs.docker.com/compose/overview/)

## Instalação do Docker Compose

O Docker Compose pode ser utilizado nos sistemas operacionais macOS, Windows e Linux 64-bit.

### Linux

1. Realize o download do binário a partir do repositório do GitHub:

```bash
$ curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
```

2. Inclua permissão de execução para o arquivo:

```bash
$ chmod +x /usr/local/bin/docker-compose
```

3. Teste a instalação:

```bash
$ docker-compose --version
```

É possível a instalação do command completion para bash, siga as instruções presentes [aqui](https://docs.docker.com/compose/completion/)

##### Para saber mais

> [Documentação Docker - Install Docker Compose](https://docs.docker.com/compose/install/)

## Comandos e utilização

O binário do Docker Compose provê uma interface, via linha de comando, com a seguinte sintaxe:

```bash
docker-compose [opções] [COMANDO] [ARGUMENTOS...] [SERVIÇOS ...]

Opções:
  -f, --file ARQUIVO          Especifica um arquivo de configuração alternativo (default: docker-compose.yml)
  -p, --project-name NOME     Especifica um nome alternativo de projeto (default: nome do diretório)
  --verbose                   Mostra informações detalhadas
  -v, --version               Exibe a versão
  -H, --host HOST             Socket do Daemon para conexão remota
```


|Comando|Descrição|Opções comuns|
|---|---|---|
  |build|Constrói ou reconstrói serviços|`--no-cache` : Não utilizar o cache na construção das imagens|
  |config|Valida e visualiza o arquivo de configuração| |
  |down|Pára e remove contêineres, redes, imagens e volumes| `-v`, `--volumes` : Remove os volumes|
  |exec|Executa comandos em um contêiner em execução| |
  |images|Lista imagens| |
  |kill|Interrompe contêineres| |
  |logs|Exibe os logs dos contêineres| `-f` : atualiza a saída do log continuamente|
  |ps|Lista os contêineres| |
  |pull|Realiza o pull das imagens dos serviços| |
  |push|Realiza o push das imagens dos serviços| |
  |restart|Reinicializa os serviços| |
  |rm|Remove contêineres parados|`-f`, `--force` : Não pede confirmação para remoção |
  |start|Inicializa serviços| |
  |stop|Pára serviços| |
  |top|Exibe os processos em execução| |
  |up|Cria e inicia os contêineres| `-d` : Executa contêineres em background <br> `--force-recreate` : Recria os contêineres nos casos em que não houve alteração na configuração ou imagem <br>`--build` : Constrói as imagens antes de iniciar os contêineres|
  |version|Exibe a informação de versão do Docker Compose| |

##### Para saber mais

> [Documentação Docker - Compose command-line reference](https://docs.docker.com/compose/reference/)

> [Documentação Docker - Frequently asked questions](https://docs.docker.com/compose/faq/)



## Compose Files

### Criação e utilização

Um arquivo de configuração do Docker Compose (**compose file**) é um arquivo no padrão YAML que define o comportamento dos contêineres Docker para que funcionem em conjunto para entrega de uma aplicação.

O padrão YAML utiliza a indentação como separador dos
blocos de códigos das definições, por conta disso, caso o compose file não seja escrito corretamente, o comando `docker-compose` falhará em sua execução.

O caminho padrão para um compose file é o diretório corrente (`./docker-compose.yml`).


Exemplo de Compose file:

```YAML
version: "3"

services:
  our-app:
    build: .
    environment:
      - MODE=dev
      - ELASTICSEARCH_HOST=127.0.0.1
      - ELASTICSEARCH_PORT=9200
    volumes:
      - .:/code
    depends_on:
      - elasticsearch
      - rabbitmq
    ports:
      - "9091:9090"
    networks:
      - appnet
      
  rabbitmq:
    image: rabbitmq:3.6.1-management
    environment:
      - RABBITMQ_DEFAULT_USER=admin
      - RABBITMQ_DEFAULT_PASS=admin
    ports:
      - "8023:15672"
      - "8024:5672"
    networks:
      - appnet

  elasticsearch:
    image: elasticsearch:5-alpine
    restart: unless-stopped
    command: elasticsearch -Etransport.host=127.0.0.1
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - ../data:/usr/share/elasticsearch/data
    networks:
      - appnet
networks:
  appnet:
```

O compose file é iniciado com a indicação da versão da especificação a ser utilizada, seguido pela seção de definição dos serviços:

```YAML
version: "3"

services:
  our-app:
...
```

Indentado dentro da seção do serviço correspondente está a sua configuração:

```YAML
services:
  our-app:
    build: .
    environment:
      - MODE=dev
      - ELASTICSEARCH_HOST=127.0.0.1
      - ELASTICSEARCH_PORT=9200
    volumes:
      - .:/code
    depends_on:
      - elasticsearch
      - rabbitmq
    ports:
      - "9091:9090"
    networks:
      - appnet
```

Uma definição de serviço contém a configuração que é aplicada a cada contêiner iniciado para esse serviço, assim como é feito com o uso do comando `docker container run` . Da mesma forma, as definições de rede e de volume são análogas à `docker network create` e `docker volume create`.

As opções especificadas no Dockerfile, como `CMD`, `EXPOSE`, `VOLUME` e `ENV`, são respeitadas por padrão - não é necessário especificá-las novamente no `docker-compose.yml`.

Nos arquivos de configuração podem ser utilizadas variáveis de ambiente em valores de configuração com uma sintaxe `${VARIÁVEL}` semelhante ao Bash do Linux.

O Docker Compose utiliza os valores das variáveis definidos no ambiente shell no qual o comando `docker-compose` está em execuçao. Caso a variável de ambiente não esteja atribuída, o valor vazio é repassado para a variável no compose-file.

### Opções de configuração

#### build

Configuração da construção do serviço.

|Subopções|Descrição|
|---|---|
|context|Contexto alternativo de construção. Por padrão é o diretório corrente do `docker-compose.yml`|
|dockerfile|Caminho de Dockerfile alternativo para construção do serviço|

Exemplo:

```YAML
version: '3'
services:
  webapp:
    build:
      context: ./dir
      dockerfile: Dockerfile-alternate
```    


Caso a opção `image` seja informada, no mesmo nível de `build`, o nome informado (no formato `NOMEIMAGEM:TAG`)será atribuído à imagem que será construída.

#### depends_on

Expressa a dependência entre serviços. 

Quando é executado o comando `docker compose up`, no caso da utilização de `depends_on`, os serviços serão inicializados na ordem de dependência.

Exemplo:

```YAML
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

A utilização de `depends_on` apenas verifica a dependência para inicialização, essa opção não verifica se o serviço já está disponível (ready).

#### environment

Define variáveis de ambiente e seus valores. Similar a opção `-e` ou `--env` do comando `docker container run`.

Exemplo:

```YAML
services:
  our-app:
    build: .
    environment:
      - MODE=dev
      - ELASTICSEARCH_HOST=127.0.0.1
      - ELASTICSEARCH_PORT=9200
```

#### image

Especifica a imagem da qual o contêiner será inicializado.

```YAML
version: '3'
services:
  influxdb:
    image: tutum/influxdb:latest
  db:
    image: postgres
```

#### networks

Definem as redes às quais o serviço será conectado. 

Um serviço pode se conectar a mais de uma rede. 

Essa opção referencia as redes definidas pelo usuário no nível mais alto do compose-file (chave `networks`).

Exemplo:

```YAML
services:
  web:
    build: ./web
    networks:
      - new

  worker:
    build: ./worker
    networks:
      - legacy

  db:
    image: mysql
    networks:
      new:
        aliases:
          - database
      legacy:
        aliases:
          - mysql

networks:
  new:
  legacy:
```

É possível a definição da subopção `aliases` para atribuir um  nome adicional a um serviço na rede.

#### ports

Expõe e vincula portas de um serviço. Similar a opção `-p` do comando `docker container run`.

É possível especificar o mapeamento das portas do host e contêiner (sintaxe `HOST:CONTÊINER`), ou apenas a porta do contêiner que será mapeada para uma porta alta do host automaticamente.

```YAML
services:
  our-app:
    ports:
      - "9091:9090"
```

#### volumes

A opção `volumes` permite montar volumes nomeados ou diretórios do host. Similar a opção `-v` do comando `docker container run`.

A definição do mapeamento dos volumes a serem utilizados pelo serviço pode ser feita diretamente na seção correspondente no compose-file sem a necessidade de referenciar a seção `volumes` em nível mais alto.

Nos casos em que se necessite compartilhar o mesmo volume entre serviços, além de indicar o volume na seção referente ao serviço, é necessário definir o volume nomeado na seção `volumes` no nível mais alto do compose-file.

Exemplo

```YAML
  elasticsearch:
    image: elasticsearch:5-alpine
    volumes:
      - ../data:/usr/share/elasticsearch/data
```

#### restart

Essa opção define a política de reinicialização dos serviços. Similar a opção `--restart` do comando `docker container run`.

As opções são: `no`, `always`, `on-failure` e `unless-stop`.

```YAML
version: "3"

services:

  elasticsearch:
    image: elasticsearch:5-alpine
    restart: unless-stopped
    command: elasticsearch -Etransport.host=127.0.0.1
    environment:
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - ../data:/usr/share/elasticsearch/data
    networks:
      - appnet
networks:
  appnet:
```

### Substituição e extensão de configurações

#### Substituição

Por padrão, o Docker Compose lê dois arquivos, o arquivo padrão `docker-compose.yml` e um arquivo opcional chamado `docker-compose.override.yml`. 

Por convenção, o `docker-compose.yml` contém a configuração base dos serviços. O arquivo de substituição, como o próprio nome indica, sobrescreve configuração para serviços existentes ou define serviços novos.

Se um serviço for definido em ambos os arquivos, o Compose mescla as configurações.

É possível utilizar um ou mais arquivos de substituição, basta para isso indicar o nome dos arquivos utilizando a opção `-f`. O Docker Compose mescla os arquivos na ordem em que são especificados na linha de comando.

Exemplo:

```
$ docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

docker-compose.yml

```YAML
web:
  image: example/my_web_app:latest
  links:
    - db
    - cache

db:
  image: postgres:latest

cache:
  image: redis:latest
```

docker-compose.prod.yml

```YAML
web:
  ports:
    - 80:80
  environment:
    PRODUCTION: 'true'

cache:
  environment:
    TTL: '500'
```

Quando são utilizados vários arquivos de configuração, todos os caminhos devem ser definidos em relação ao arquivo base.

#### Extensão

O compartilhamento de configurações entre diferentes arquivos, ou mesmo projetos diferentes, é possível por meio da opção `extends` fornecida pelo Docker Compose.

A extensão de serviços é útil caso existam vários serviços que reutilizam um conjunto comum de opções de configuração.

Exemplo

docker-compose.yml

```YAML
web:
  extends:
    file: common-services.yml
    service: webapp
```

common-services.yml

```YAML
webapp:
  build: .
  ports:
    - "8000:8000"
  volumes:
    - "/data"
```

##### Para saber mais

> [Documentação Docker - Compose file version 3 reference](https://docs.docker.com/compose/compose-file/)

> [Documentação Docker - Variable substitution](https://docs.docker.com/compose/compose-file/#variable-substitution)

> [Documentação Docker - Control startup order in Compose](https://docs.docker.com/compose/startup-order/)

> [Documentação Docker - Share Compose configurations between files and projects](https://docs.docker.com/compose/extends/)

## Let's play! 
>[Playground 7](playground/play7.md)

---
<p align="left">
<a href='../unidade5/unidade5.md' id='unidade5' class='anchor' aria-hidden='true'><< Unidade 5 - Trabalhando com Redes</a></p>
<p align="right">
<a href='../unidade7/unidade7.md' id='unidade7' class='anchor' aria-hidden='true'>Unidade 7 - Introduzindo orquestração de contêineres >></a></p>