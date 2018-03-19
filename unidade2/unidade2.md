# Ciclo de vida de contêineres

[Comandos para manipulação de objetos Docker](#comandos-para-manipulação-de-objetos-docker)

[Execução de contêineres:](#execução-de-contêineres)

- [Parâmetros de criação](#parâmetros-de-criação)

- [Configurações de rede](#configurações-de-rede)

- [Limitações de recursos](#limitações-de-recursos)

- [Execução privilegiada](#execução-privilegiada)

- [Logging](#logging)

- [Monitoramento](#monitoramento)

## Comandos para manipulação de objetos Docker

### Comandos para manipulação de imagens

#### Baixar imagem do registry

```
$ docker image pull [OPÇÕES] [REGISTRO PRIVADO/] NOME [:TAG]
```

Exemplo:
```
# Baixa imagem padrão (tag latest)

$ docker image pull centos

# Baixa tag específica

$ docker image pull centos:6
```

#### Subir imagem para o registry

```
$ docker image push [OPÇÕES] [REGISTRO PRIVADO/] NOME [:TAG]
```

Exemplo:
```
$ docker image push imagemdeteste

```

#### Construir uma imagem a partir do Dockerfile

```
$ docker image build [-t NOME] .
```

Exemplo:
```
$ docker image build -t imagemdeteste .
```

#### Listar as imagens presentes no host

```
$ docker image ls

Opções:

  -a, --all             Exibe todas as imagens (padrão não exibe as imagens intermediárias)
  -f, --filter          Filtra a saída de acordo com condições definidas
  --format string       Formata saída usando um template Go
  --no-trunc            Não trunca a saída
  -q, --quiet           Mostra apenas IDs númericos

```

Exemplo:
```
# Lista as imagens do host

$ docker image ls

# Lista somente os IDs das imagens

$ docker image ls -q

# Formata a saída

$ docker image ls --format "{{.ID}}: {{.Repository}}"

$ docker image ls --format "table {{.ID}}\t{{.Repository}}\t{{.Tag}}"
```

#### Remover imagem do host

```
$ docker image rm [OPÇÕES] NOME/ID

Opções:

  -f      Força a remoção da imagem
```
Exemplo:
```
$ docker image rm centos:6
```
#### Exibir histórico de criação de imagem (camadas)

```
$ docker image history NOME/ID
```
Exemplo:
```
$ docker image history centos
```
#### Exibir informações detalhadas sobre as imagens

```    
$ docker image inspect NOME/ID
```
Exemplo:
```
$ docker image inspect centos
```

#### Remover imagens não utilizadas (não referenciadas e "dangling")

```    
$ docker image prune [OPÇÕES]

Opções:

  -a, --all             Remove todas as imagens não utilizadas, não somente as imagens dangling
      --filter          Permite que seja informado um filtro (p.ex. 'until=<timestamp>')
  -f, --force           Não solicita confirmação
```

- Imagens `dangling` são imagens presentes no host, mas que foram substituídas por uma versão mais recente com o mesmo nome e tag.;

- Imagens `dangling` não tem uso e com o tempo podem lotar o seu espaço de armazenamento;

- Imagens `dangling` aparecem com nome e tag igual a `<none>:<none>` quando executado o comando `docker image ls`. Essas imagens não devem ser confundidas com as imagens `<none>:<none>` que são mostradas na saída do comando `docker image ls -a`, nesse caso são imagens intermediárias das imagens útéis;

- Imagens não referenciadas são imagens presentes no host, mas que não estão vinculadas a nenhum contêiner em execução ou parado.

Exemplo:
```
# Remove somente as imagens "dangling"

$ docker image prune

# Remove todas as imagens não referenciadas e "dangling"

$ docker image prune -a

# Remove todas as imagens não referenciadas e "dangling" criadas a mais de 10 dias sem solicitar confirmação 

$  docker image prune -a --force --filter "until=240h"
```

### Comandos para utilização de registros Docker

#### Realizar login no registry privado

```
$ docker login [REGISTRO PRIVADO]
```

Exemplo:
```
$ docker login registry.stf.jus.br
```

#### Realizar logout no registry privado

```
$ docker logout [REGISTRO PRIVADO]
```

Exemplo:
```
$ docker logout registry.stf.jus.br
```

#### Pesquisar imagens no Docker Hub

```
$ docker search [OPÇÕES] TERMO

Opções:
  -f, --filter      Filtra a saída de acordo com condições definidas
                    Filtros:  stars (int) - número de estrelas da imagem
                              is-automated (boolean - true ou false) - se possui build automatizado
                              is-official (boolean - true ou false) - se a imagem é oficial
  --format string   Formata saída usando um template Go
  --limit int       Número máximo de resultados (default 25)
  --no-trunc        Não trunca a saída
```

Exemplo:
```
# Pesquisa imagem no Docker Hub com o termo java (nome da imagem ou descrição)

$ docker search java

# Pesquisa imagem no Docker Hub com o termo java e se é oficial

$ docker search -f is-official=true java

# Formata a saída

$ docker search --format "{{.Name}}: {{.StarCount}}" java
```

### Comandos para criação e operação de contêineres Docker

#### Executar o contêiner a partir da imagem

```
$ docker container run [OPÇÕES] [REGISTRO PRIVADO/] IMAGEM[:TAG] [COMANDO] [ARG...]

Opções:

  -d          Execução do container em background
  -i          Modo interativo. Mantém o STDIN aberto.
  -t          Aloca uma pseudo TTY
  –-rm        Remove o container após finalização
              (Não funciona com -d)
  –-name      Atribui um nome ao contêiner
```

Exemplo:
```
# Executa o contêiner

$ docker container run tomcat:8.0

# Executa o contêiner em background

$ docker container run -d tomcat:8.0

# Executa o contêiner em background e atribui um nome

$ docker container run -d --name jerry tomcat:8.0

# Executa o contêiner e o remove automaticamente na finalização

$ docker container run --rm tomcat:8.0

# Executa o contêiner e roda um comando

$ docker container run centos cat /etc/centos-release
```
#### Exibir os contêineres em execução

```    
$ docker container ls [OPÇÕES]

Opções:

  -a, --all             Exibe todos os contêineres (padrão exibe somente em execução)
  -f, --filter          Filtra a saída de acordo com condições definidas
  --format string       Formata saída usando um template Go
  -n, --last int        Mostra n últimos contêineres criados (inclui todos os estados) (default -1)
  -l, --latest          Mostra o último contêiner criado
  --no-trunc            Não trunca a saída
  -q, --quiet           Mostra apenas IDs númericos

```

Exemplo:
```
# Lista os contêineres em execução no host

$ docker container ls

# Lista somente os IDs dos contêineres

$ docker container ls -q

# Lista todos os contêineres do host (inclusive parados)

$ docker container ls -a

# Lista contêineres filtrando pelo nome

$ docker container ls --filter "name=jerry"

# Lista contêineres filtrando somente os finalizados sem erro

$ docker container ls -a --filter 'exited=0'

# Lista contêineres com status de pausado

$ docker container ls --filter status=paused

# Lista contêineres que foram criado a partir de uma imagem específica ou outras imagens que tiveram essa como base

$ docker container ls -a --filter ancestor=tomcat:8.0

# Formata a saída

$ docker container ls --format "{{.ID}}: {{.Command}}"

$ docker container ls --format "table {{.ID}}\t{{.Labels}}"
```
#### Inicializar e finalizar um contêiner

```
$ docker container start NOME/ID
    
$ docker container stop NOME/ID
```

Exemplo:
```
# Finaliza o contêiner de nome Jerry

$ docker container stop jerry

# Inicializa o contêiner pelo ID

$ docker container start 36cc6767b041

```
#### Exibir informações detalhadas sobre os contêineres

```    
$ docker container inspect NOME/ID
```

Exemplo:
```

$ docker container inspect 36cc6767b041

```

#### Remover contêineres do host

```
$ docker container rm [OPÇÕES] NOME/ID

Opções:

  -f, --force     Força a remoção de contêineres em execução (usa SIGKILL)
  -v, --volumes   Remove volumes associados ao contêiner
```

Exemplo:
```

$ docker container rm 36cc6767b041

```
#### Executar comandos em um contêiner

```
$ docker container exec [OPÇÕES] NOME/ID COMANDO [ARG...]

Opções:

  -d          Execução o comando em background
  -i          Modo interativo. Mantém o STDIN aberto
  -t          Aloca uma pseudo TTY
  -u          Define um usuário ou UID para execução
  -e          Define váriaveis de ambiente
```

Exemplo:
```
# Executa um comando touch em um contêiner em execução

$ docker container exec jerry touch /tmp/jerry.txt

# Inicializa uma sessão no contêiner em execução

$ docker container exec -i -t jerry /bin/bash

```

#### Remover contêineres parados

```    
$ docker container prune

Opções:

      --filter          Permite que seja informado um filtro (p.ex. 'until=<timestamp>')
  -f, --force           Não solicita confirmação
```


Exemplo:
```
# Remove todos os contêineres parados

$ docker container prune

# Remove todos os contêineres criados a mais de 5 minutos sem solicitar confirmação

$ docker container prune --force --filter "until=5m"
```

### Comandos de sistema para o Docker

#### Exibir informações de sistema do Docker

```
$ docker system info [OPÇÕES]

Opções:

  --format string   Formata saída usando um template Go
  
```

Exemplo:
```
# Exibe informações de sistema do Docker

$ docker system info

# Exibe informações de sistema do Docker em formato JSON

$ docker system info --format '{{json .}}'

```

#### Mostrar o uso de disco pelo Docker

```
$ docker system df [OPÇÕES]

Opções:

  --format string   Formata saída usando um template Go
  -v, --verbose     Mostra informações detalhadas de uso de espaço

```

Exemplo:
```
# Mostra o uso de disco pelo Docker, por padrão a informação é sintética

$ docker system df

# Mostra o uso de disco pelo Docker de forma detalhada

$ docker system df -v

```

#### Remover todos os dados não utilizados

```
$ docker system prune [OPÇÕES]

Opções:

--all , -a      Remove todas as imagens não utilizadas, não somente as imagens dangling
--filter        Permite que seja informado um filtro (p.ex. 'label==')
  -f, --force   Não solicita confirmação (e.g. ‘')
--volumes       Elimina volumes não utilizados por nenhum contêiner
```

- A opção `docker system prune` remove todos os contêineres parados, volumes (com a opção `--volumes`), redes (não referenciadas por nenhum contêiner), imagens não utilizadas - dangling e não referenciadas (com a opção `-a`) e dados do cache.

Exemplo:
```
# Remove todos os contêineres parados, redes (não referenciadas por nenhum contêiner) e imagens dangling

$ docker system prune

# Remove todos os dados

$ docker system prune -a --volumes

```

##### Para saber mais:

> [Documentação Docker - Use the Docker command line](https://docs.docker.com/engine/reference/commandline/cli/)

## Let's play! 
>[Playground 2](playground/play2.md)

# Execução de contêineres

## Parâmetros de criação

O comando `docker container run` em primeiro lugar cria uma camada de leitura para o contêiner sobre a imagem especificada, e em seguida, inicializa o comando especificado.

Segue a sintaxe desse comando:

```
$ docker container run [OPÇÕES] [REGISTRO PRIVADO/] IMAGEM[:TAG] [COMANDO] [ARG...]
```

Algumas opções já vistas estão presentes na tabela abaixo:

|OPÇÃO|FINALIDADE|
| --- | --- |
|-d |Execução do container em background|
|-i |Modo interativo. Mantém o STDIN aberto.|
|-t |Aloca uma pseudo TTY|
|- - rm |Remove o container após finalização|
|- - name| Atribui um nome ao contêiner|

Outras opções importantes serão apresentadas a seguir.

#### Publicar portas dos contêneires (-p)

A opção **-p** mapeia uma porta no contêiner, tornando-o acessível a partir do host.

```
$ docker container run -p <porta_host>:<porta_container> IMAGEM
``` 
Exemplo:
```
$ docker container run -it --rm -p 8888:8080 tomcat:8.0
```

- A interface do host, na qual a porta será exposta, também pode ser especificada (p.ex. 127.0.0.1:80:8080)

- Se a porta do host não for definida, uma porta aleatória de numeração alta será escolhida (pode ser descoberta pelo comando `docker container port <id|nome do contêiner>`);

- Não é possível acessar a porta 8080 no endereço IP do Docker host, pois essa porta está acessível apenas dentro do container que é isolada a nível de rede;

- Não é possível mapear a mesma porta do host em contêineres diferentes;

- É possível utilizar várias vezes a opção **-p** na execução do comando docker container run;

- A opção **-P** publica todas as portas expostas (por meio da instrução EXPOSE na imagem) no contêiner para o host.

#### Definir variáveis de ambiente (-e, –-env)

A opção **-e** ou **--env** é utiliza para definir variáveis de ambiente simples (não-array) dentro contêiner para execuçao.

```
$ docker container run -e VAR1=valor IMAGEM
``` 
Exemplo:
```
$ docker container run -e var1=val -e var2="val 2" debian env
```

- É possível substituir o valor de variáveis definidas originalmente na imagem;

- Caso o valor da variável não seja passado por meio do "=", o cliente Docker verifica se existe a variável no ambiente local do host e passa o valor para o contêiner;

- É possível utilizar várias vezes as opções **-e** e **--env** na execução do comando docker container run;

- A opção **--env-file** permite a passagem de variáveis via arquivo. Este arquivo deve usar a sintaxe <variável> = valor (que define a variável para o valor dado) ou <variável> (que assume o valor do ambiente local) e # para comentários;

- O Docker define automaticamente algumas variáveis de ambiente ao criar um contêiner Linux (isso não ocorre para os contêineres Windows). São configuradas as variáveis de ambiente HOME, HOSTNAME, PATH (com o caminho dos diretórios de binários mais comuns)  e TERM (xterm se ao contêiner é anexado a um pseudo-TTY).

#### Mapeamento de volumes (-v,--volume)

Volumes Docker são diretórios que não fazem parte da estrutura UnionFS do contêiner, isto é, são diretórios no host montados por vinculação dentro do contêiner.

- Os volumes são inicializado nos contêineres utilizando a opção `-v`;

```
docker container run -v [HOST_DIR:]CONTAINER_DIR IMAGEM
```

- Se o diretório do host (`HOST_DIR`) não é informado o volume é criado dentro da estrutura de metadados do Docker (`/var/lib/docker`). O conteúdo do diretório do diretório do contêiner é copiado para o volume;

- Quando o diretório do host é informado o diretório em questão é mapeado dentro do volume; Caso o diretório já exista no contêiner seu conteúdo será oculto pelo conteùdo do diretório do host;

Exemplo:

```
# Mapeia a pasta /tmp do contêiner para um volume (sem indicar o diretório do host)

$ docker container run -d -it -v /tmp --name=centvol centos bash

# Verificar para qual diretório foi mapeado o volume

$ docker container inspect -f {{.Mounts}} centvol

# Mapeia /tmp do contêiner para um diretório no host

$ docker container run -d -it -v /centvol:/tmp --name=centvol1 centos bash

# Cria arquivo no diretório do contêiner mapeado como volume

$ docker container exec centvol1 touch /tmp/1.txt

# Verificar o diretório no host

$ ls /centvol/
```

- Com a opção `--volumes-from` é possível montar volumes a partir de um outro contêiner.

Exemplo:

```
# Monta os volumes a partir de outro contêiner

$ docker container run -d -it --volumes-from centvol --name=centvol2 centos bash

# Cria arquivo no diretório do contêiner mapeado como volume

$ docker container exec centvol touch /tmp/teste1.txt

# Verificar o contéudo do diretório mapeado no outro contêiner

$ docker container exec centvol2 ls /tmp

```

#### Políticas de reinicialização (--restart)

Utilizando a opção **--restart** é possível definir a política de reinicialização do contêiner, ou seja, em que condições o contêiner deve ser iniciado novamente de forma automática quando encerrado.

```
$ docker container run --restart=<POLÍTICA> IMAGEM
``` 
Exemplo:
```
$ docker container run --restart=always redis
```

O Docker suporta as seguintes políticas:

|POLÍTICA|RESULTADO|
| :---: | --- |
| no | Não reinicia automaticamente o contêiner. Esse é o padrão|
| on&#8209;failure[:max&#8209;retries] | Reiniciará os contêineres *encerrados com status diferente de zero* e pode receber um argumento opcional especificando a quantidade de tentativas de reinicialização.|
| always | O contêiner sempre será reiniciado independente do status de encerramento.|
| unless&#8209;stopped | Sempre reinicia o contêiner, independentemente do status de saída, mas não o inicializa na inicialização do daemon se o contêiner tiver sido parado antes.|

## Configurações de Rede

Por padrão, todos os contêineres têm rede ativada e podem fazer qualquer conexão de saída.

Os tipos de rede suportados são:

|REDE|DESCRIÇÃO|
| :---: | --- |
| none | Nenhuma rede no contêiner|
| bridge (padrão) | Conecta o contêiner a rede bridge via interfaces veth (virtual ethernet)|
| host | Usa a pilha de rede do host dentro do contêiner|
| container:<nome\|id>  | Utiliza a pilha de rede de outro contêiner, especificado por seu nome ou identificação.|
| Definida pelo usuário | Conecta o contêiner a uma rede criada pelo usuário (usando o comando `docker network create`)|

### Modo de rede **none**

Com a rede definida para `none`, o contêiner ainda terá uma interface de loopback habilitada, mas não possui rotas para o tráfego externo.

Exemplo:

```
docker container run --network none alpine ifconfig
```
### Modo de rede **bridge**

A configuração de rede padrão do docker é a `bridge`. 

Uma interface bridge é configurada no host (`docker0`), e um par de interfaces veth serão criadas para o contêiner (uma dentro do contêiner (eth0) e outra ligada a bridge do host).

Um endereço IP será alocado para contêineres na rede bridge e o tráfego será encaminhado por essa interface para o contêiner.

<!--O net namespace permite que cada contêiner possua sua interface de rede e portas. Para que seja possível a comunicação entre os contêineres, é necessário criar dois net namespaces diferentes, um responsável pela interface do contêiner (normalmente utilizamos o mesmo nome das interfaces convencionais do Linux, por exemplo, a eth0) e outro responsável por uma interface do host, normalmente chamada de veth* (veth + um identificador aleatório). Essas duas interfaces estão conectadas por meio da bridge docker0 no host, que permite a comunicação entre os contêineres através de roteamento de pacotes. Uma virtual ethernet (veth) permite conectar um par de conexão, semelhante a um pipe (a saída de uma interface é a entrada da outra). A virtual ethernet funciona como um pipe, logo interligando as pontas é possível direcionar conteúdo.
-->

Exemplo:

```
docker container run alpine ifconfig
```

### Modo de rede **host**

Com a rede configurada para `host`, o contêiner compartilha a pilha de rede do host, dessa forma todas as interfaces do host estarão disponíveis para o contêiner.

Em comparação com o modo de `bridge` padrão, o modo `host` oferece um desempenho de rede significativamente melhor, pois ele usa a plataforma de rede nativa do host, enquanto que no modo `bridge` o tráfego de rede deve passar por um nível de virtualização através do daemon docker. 

Porém com esse modo de rede ativado, o contêiner tem acesso total aos serviços de rede do sistema local, podendo ser menos seguro.

Exemplo:

```
docker container run --network host alpine ifconfig
```

### Modo de rede **container**

A definição do modo de rede como `container`, um contêiner irá compartilhar a pilha de rede de outro contêiner. 

A opção deve indicar o nome ou id do contêiner do qual será utilizada a pilha de rede: `--network container:<name|id>`.

Exemplo:

```
$ docker container run -d -t  --name=alpine1 alpine

$ docker container run -d -t --name=alpine2 --network container:alpine1 alpine

$ docker container exec alpine1 ifconfig

$ docker container exec alpine2 ifconfig

# O que acontece se o contêiner que provê a pilha de rede for removido?

$ docker container rm -f alpine1

$ docker container exec alpine2 ifconfig
```
### Modo de rede definido pelo usuário

Será visto em detalhes na [Unidade 5](../unidade5/unidade5.md)

### Outras opções referentes a rede para utilização com **docker container run**

<!--

--dns=[]           : Set custom dns servers for the container
--add-host=""      : Add a line to /etc/hosts (host:IP)

As portas de publicação e a ligação a outros contêineres só funcionam com o padrão (ponte). O recurso de ligação é um recurso legado. Você sempre deve preferir usar os drivers de rede Docker sobre a ligação.

Seu contêiner usará os mesmos servidores de DNS que o host por padrão, mas você pode substituir isso com --dns.

Por padrão, o endereço MAC é gerado usando o endereço IP alocado ao contêiner. Você pode definir o endereço MAC do recipiente explicitamente fornecendo um endereço MAC através do parâmetro --mac-endereço (formato: 12: 34: 56: 78: 9a: bc). Lembre-se de que o Docker não verifica se os endereços MAC especificados manualmente são único.
-->

|OPÇÃO|DESCRIÇÃO|
| :---: | --- |
| -h,&#8209;&#8209;hostname | Define o hostname do contêiner|
| &#8209;&#8209;add&#8209;host="" | Adiciona uma linha ao arquivo /etc/hosts do contêiner|
| --dns | Define um servidor DNS específico para o contêiner (nameserver)|
| --dns-search | Define os nomes de domínio para pesquisa DNS (search)|

Exemplos:

Quando utilizada a opção `-h` ou `--hostname` o nome de host é gravado nos arquivos /etc/hostname e /etc/hosts do contêiner. Nesse último arquivo é adicionado o hostname e endereço IP do contêiner.

```
$ docker container run -h alpine0 alpine cat /etc/hosts
```

O uso da opção `--add-host` inclui uma entrada ao arquivo /etc/hosts do contêiner.

```
$ docker container run --rm --add-host dbserver:192.168.20.233 alpine cat /etc/hosts
```

Por padrão as entradas do arquivo /etc/resolv.conf do host são utilizadas para a criação desse arquivo para o contêiner na sua criação. 

Porém, é possível incluir um ou mais endereços IP como nameserver no arquivo /etc/resolv.conf do contêiner por meio da opção `--dns`. 

Os processos no contêiner utilizarão os IPs fornecidos como serviço de resolução de nomes.

```
$ docker container run --rm --dns=8.8.8.8 --dns=8.8.4.4 alpine cat /etc/resolv.conf
```

A opção `--dns-search` define os nomes de domínio no arquivo /etc/resolv.conf que são pesquisados ​​quando um nome de host não qualificado é usado dentro do contêiner. Por padrão os nameserver são copiados do host.

```
$ docker container run --rm --dns-search=rede.local.com.br alpine cat /etc/resolv.conf
```

##### Para saber mais

> [Referência do Docker container run - Network settings](https://docs.docker.com/engine/reference/run/#network-settings)

> [Documentação Docker - Configure container DNS](https://docs.docker.com/engine/userguide/networking/default_network/configure-dns/)

## Limitações de recursos

Utilizando Docker é possível restringir o uso de recursos de mémoria e CPU pelos contêineres em tempo de execução.

### Uso de memória

Para limitação de uso de mémoria, as opções abaixo podem ser utilizadas na execução do comando `docker container run`:

|OPÇÃO|DESCRIÇÃO|
| :---: | --- |
| -m, --memory="" | Limite de memória (formato: <número> [\<unidade>]). O número é um número inteiro positivo. A unidade pode ser uma de b, k, m ou g. Mínimo é 4M.|
| &#8209;&#8209;memory&#8209;swap="" | Limite de memória total (memória + swap, formato: <número> [\<unidade>]). O número é um número inteiro positivo. A unidade pode ser uma de b, k, m ou g.|
| &#8209;&#8209;memory&#8209;reservation="" | Limite soft de memória (formato: <número> [\<unidade>]). O número é um número inteiro positivo. A unidade pode ser uma de b, k, m ou g.|


Exemplos:

Quando não é definida uma limitação, o processo no contêiner pode utilizar a quantidade de memória RAM e swap que ele necessitar:

```
$ docker container run -it ubuntu:14.04 /bin/bash
```
Quando definido um limite de memória e o limite de memória swap é desabilitado (valor=-1), significa que os processos no contêiner podem usar a quantidade de memória RAM definida - que no caso do exemplo abaixo é de 300M - e a toda a memória swap que precisarem (se o host suportar a memória swap).

```
$ docker container run -it -m 300M --memory-swap -1 ubuntu:14.04 /bin/bash
```

Definindo somente o limite de memória, isso significa que os processos no contêiner podem usar a memória RAM definida, no caso de 300M, e a mesma quantidade de memória swap. Por padrão, a memória total utilizada (memória RAM + swap) deve ser o dobro do valor da memória RAM definida.

```
$ docker container run -it -m 300M ubuntu:14.04 /bin/bash
```

Quando definidos os valores de memória RAM e memória swap, os processos no contêiner podem usar a quantidade de memória RAM definida e mais a diferença de memória swap limitado ao valor total definido pelo valor do swap. Ou seja, no exemplo poderá ser utilizado 300M de memória RAM e 700M de memória swap.

```
$ docker container run -it -m 300M --memory-swap 1G ubuntu:14.04 /bin/bash
```

A reserva de memória é um tipo de limite de memória flexível (soft) que permite um maior compartilhamento de memória. Em circunstâncias normais, os contêineres podem usar memória conforme necessitam e são limitados apenas pelos limites rígidos definidos com a opção de memória `-m` . Quando a reserva de memória está configurada, o Docker detecta uma contenção da memória ou pouca memória e força os contêineres para restringir seu consumo a um limite de reserva.

- Defina sempre o valor da reserva de memória abaixo do limite rígido (`-m`), caso contrário, o limite rígido tem precedência;

- Uma reserva de 0 é a mesma que não definir nenhuma reserva. Por padrão, a reserva de memória é o mesmo valor do limite rígido de memória;

- A reserva de memória é um recurso de limite soft e não garante que o limite não seja excedido. Em vez disso, o recurso tenta garantir que, quando houver uma forte contenção de memória, a memória seja (re)alocada aos contêineres com base nesse parâmetro.

O exemplo a seguir limita a memória (`-m`) a 500M e configura a reserva de memória para 200M. Com essa configuração, quando o contêiner consome memória superior a 200M e inferior a 500M, na próxima recuperação de memória (memory reclaim) do sistema, o Docker tenta reduzir a memória do contêiner para abaixo de 200M.
```
$ docker container run -it -m 500M --memory-reservation 200M ubuntu:14.04 /bin/bash
```

O exemplo a seguir configura a reserva de memória para 1G sem um limite de memória rígida. O contêiner poderá utilizar quanta memória precisar, porém com a configuração de uma reserva de limite, o consumo de muita memória não será por muito tempo. Pois com a execução do memory reclaim, o sistema tentará reduzir o uso de memória ao limite de reserva.

```
$ docker container run -it -memory-reservation 1G ubuntu: 14.04 / bin / bash
```

- Por padrão, o kernel mata o contêiner se ocorrer um erro de falta de memória (OOM). A opção `--oom-kill-disable` desativa esse comportamento (o que é não é recomendável ser feito sem a definição de limite de uso de memória).

### Uso de CPU

Para limitação de uso de CPU, as opções abaixo podem ser utilizadas na execução do comando `docker container run`:

|OPÇÃO|DESCRIÇÃO|
| :---: | --- |
| --cpu-shares , -c | Compartilhamento de ciclos de CPU|
| --cpu-quota | Limite de quota de CPU &#8209; CFS (Completely Fair Scheduler)|
| --cpu-period | Limite de período de CPU &#8209; CFS (Completely Fair Scheduler)|
| --cpuset-cpus | Restringe os processadores utilizados para execução (0-3, 0,1)|

#### Compartilhamento de ciclos de CPU (-c, --cpu-shares)

Por padrão, todos os contêineres recebem a mesma proporção de ciclos de CPU. Essa proporção pode ser modificada alterando a peso relativo de compartilhamento de CPU do contêiner em relação à todos os outros contêineres em execução.

- Para modificar a proporção do padrão de 1024, é utilizada a opção `-c` ou `--cpu-shares`. Se a opção for configurada para 0, o sistema ignorará o valor e usará o padrão de 1024;

- A proporção só será aplicada quando os processos dos contêineres utilizarem 100% da CPU. A quantidade real de tempo de CPU variará de acordo com o número de contêineres em execução no sistema;

- Por exemplo, considere três contêineres, um tem um compartilhamento de CPU de 1024 e outros dois têm uma configuração de compartilhamento de CPU de 512. Quando os processos em todos os três contêineres tentam usar 100% da CPU, o primeiro contêiner receberia 50% do tempo total de CPU. Se for adicionado um quarto contêiner com um cpu-share de 1024, o primeiro contêiner só recebe 33% da CPU. Os contêineres restantes recebem 16,5%, 16,5% e 33% da CPU.

#### Limite de quota e período de CPU - CFS (Completely Fair Scheduler)

O controle de largura de banda CFS (Completely Fair Scheduler) é um recurso do *cgroups* que permite a especificação da largura de banda máxima da CPU disponível para um grupo ou hierarquia.

- A largura de banda permitida para um grupo é especificada usando uma quota e um período.

- Dentro de cada dado "período" (microssegundos), um grupo pode consumir apenas até sua "quota" em milisegundos de tempo de CPU. Quando o consumo de largura de banda da CPU de um grupo excede esse limite (para esse período), as tarefas pertencentes à sua hierarquia serão retirados da CPU e não é permitida a sua execução até o próximo período.

- O padrão CPU CFS (Completely Fair Scheduler) é de 100ms. É utilizado `--cpu-period` para definir o período de uso da CPU pelo contêiner. E `--cpu-quota` é o tempo total de execução disponível dentro de um período.

Exemplos:

```
$ docker container run -it --cpu-period = 50000 --cpu-quota = 25000 ubuntu: 14.04 / bin / bash
```

Havendo 1 CPU, isso significa que o contêiner pode obter 50% de tempo de execução na CPU a cada 50ms.

#### Restrição de processadores para execução (--cpuset-cpus)

Pode ser definido em quais CPUs o contêiner terá permitição para execução.

Exemplos:

Para definir que os processos de um contêiner só podem ser executados na CPU 1 e na CPU 3 do Host:

```
$ docker container run -it --cpuset-cpus = "1,3" ubuntu: 14.04 / bin / bash
```
Para definir que os processos do contêiner em questão podem ser executados nas CPU 0, CPU 1 e CPU 2.

##### Para saber mais

> [Referência do Docker container run - Runtime constraints on resources](https://docs.docker.com/engine/reference/run/#runtime-constraints-on-resources)

> [Resource management in Docker](https://goldmann.pl/blog/2014/09/11/resource-management-in-docker/)

## Execução privilegiada

- Para realizar verificações de permissão, as implementações UNIX tradicionais distinguem duas categorias de processos: processos privilegiados (cujo ID de usuário efetivo é 0, conhecido como superusuário ou root) e processos não privilegiados (cujo UID efetivo é diferente de zero);

- Os processos privilegiados ignoram todas as verificações de permissão do kernel, enquanto os processos não privilegiados estão sujeitos a verificação de permissão completa com base nas credenciais do processo (cgroups);

- Por padrão, os contêineres Docker são "não privilegiados". Porém é possível criar contêiners privilegiados com o uso da opção `--privileged`. Quando é utilizada essa opção o Docker habilitará o acesso a todos os dispositivos no host e permitirá que o contêiner tenha os privilégios que os processos privilegiados que rodam diretamente no host;

- Para limitar o acesso a dispositivos específicos, pode ser utilizada a opção `--device`. Ela permite que seja especificado um ou mais dispositivos que serão acessíveis dentro do contêiner;

Exemplo:

```
$ docker container run -it --rm  ubuntu fdisk /dev/sda1

$ docker container run -it --rm --privileged --device /dev/sda7:/dev/sda1 ubuntu fdisk /dev/sda1
```

- A partir do kernel 2.2, o Linux divide os privilégios associados ao superusuário em unidades distintas, conhecidas como capacidades (*capabilities*), que podem ser habilitadas e desabilitadas de forma independente;

- As opções `--cap-add` e `--cap-drop` permitem adicionar e remover capabilities para um contâiner Docker;

- O Docker possui uma lista de recursos (capabilities) que são mantidos por padrão em um contêiner, sendo que outras capabilities não são concedidas por padrão e podem ser adicionados ao contêiner. Nessa [página](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities) podem ser encontradas essas listas.

Exemplo:

```
# Criar interface de rede sem adicionar a capability 

$ docker container run -it --rm ubuntu:14.04 ip link add dummy0 type dummy

# Criar interface de rede adicionando a capability NET_ADMIN 

$ docker container run -it --rm --cap-add=NET_ADMIN ubuntu:14.04 ip link add dummy0 type dummy
```
##### Para saber mais

> [Referência do Docker container run - Runtime privilege and Linux capabilities](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)

> [CAPABILITIES(7) - Linux Programmer's Manual](http://man7.org/linux/man-pages/man7/capabilities.7.html)

## Logging

O Docker possui vários mecanismos de registro de logs para  obter informações de contêineres em execução. Esses mecanismos são chamados de **logging drivers**;

- Cada daemon do Docker possui um driver de logging padrão, que cada contêiner usa, a menos que seja configurado um driver de logging diferente;

- Além dos logging drivers incluídos no Docker, também é possível usar plugins de logging drivers. Os plugins estão disponíveis a partir da versão 17.05 do Docker;

### Modificar o logging driver padrão

- Para modificar o logging driver padrão a ser utilizado pelo daemon do Docker é necessário definí-lo no arquivo `daemon.json`, que está localizado em `/etc/docker/` em hosts Linux ou `C:\ProgramData\docker\config\` em hosts Windows;

- O driver de logging padrão é o `json-file`;

O exemplo a seguir define explicitamente o logging driver padrão para o `syslog`:

```
{
  "log-driver": "syslog"
}
```

- Se o driver de logging tiver opções configuráveis, você pode configurá-las no arquivo daemon.json por meio da chave `log-opt`;

- Para encontrar o driver de logging padrão utilizado pelo daemon do Docker, execute o comando `docker system info`:

```
$ docker system info |grep 'Logging Driver'
```

- Quando um contêiner é inicializado, ele pode ser configurado para usar um driver de logging diferente do padrão do daemon do Docker, usando a opção `--log-driver`;

Exemplo:

O exemplo abaixo inicia um contêiner com o logging driver `none`:

```
$ docker container run -it --log-driver none centos echo "Docker!"
```

### Comando para visualização dos logs

- O comando `docker container logs` exibe os logs de um contêiner;

- A saída inclui tudo o que foi gravado nas saídas padrão (*STDOUT*) e de erros (*STDERR*) pelo contêiner;

- As opções abaixo podem ser utilizadas na execução do comando `docker container logs`:

|OPÇÃO|DESCRIÇÃO|
| :---: | --- |
| --details | Mostra detalhes extras fornecidos pelos logs|
| --follow , -f | Acompanha a saída do log|
| --since | Mostrar logs desde **timestamp** (por exemplo, 2013-01-02T13:23:37) ou **relativo** (por exemplo 42m para 42 minutos)|
| --tail | Número de linhas a mostrar a partir do final do log|
| &#8209;&#8209;timestamps,&#8209;t | Mostrar timestamps|

Exemplos:

```
# Exibe os logs do contêiner

$ docker container logs e64f8a7b2dd2

# Exibe os logs do contêiner e acompanha a saída dos logs 

$ docker container logs -f jerry

# Exibe os logs do contêiner a partir dos últimos 5 minutos

$ docker container logs --since 2m e64f8a7b2dd2

# Exibe as últimas 10 linhas do logs do contêiner

docker container logs --tail 10 jerry
```
### Logging drivers suportados

|Driver	|Descrição|
|:---:|---|
|none	|Nenhum log estará disponível para o contêiner e o  `docker container logs` não retornarão qualquer saída.|
|json-file|	Os logs são formatados como JSON.|
|syslog	|Grava mensagens de log para o `syslog`. O daemon syslog deve estar em execução na máquina host.|
|journald	|Grava mensagens de log para o `journald`. O daemon journald deve estar em execução na máquina host.|
|gelf	|Grava mensagens de log para o Graylog Extended Log Format (GELF) endpoint como Graylog ou Logstash.|
|fluentd	|Grava mensagens de log para fluentd (forward input). O daemon fluentd deve estar em execução na máquina host.|
|awslogs |Grava mensagens de log para Amazon CloudWatch Logs.|
|splunk	|Escreve mensagens de log para splunk usando o coletor de eventos HTTP.|
|etwlogs |Grava mensagens de log como eventos Event Tracing for Windows (ETW). Apenas disponível em plataformas Windows.|
|gcplogs |Grava mensagens de log para Google Cloud Platform (GCP) Logging.|

##### Para saber mais:

> [Referência do Docker container logs](https://docs.docker.com/engine/reference/commandline/logs/)

> [Documentação Docker - Configure logging drivers](https://docs.docker.com/engine/admin/logging/overview/)

## Monitoramento

O Docker possui uma ferramenta básica que retorna um fluxo dinâmico de uso de recursos - `docker container stats`.

- As estatísticas abordam o uso de CPU, memória e utilização de rede;

- O comando recebe o nome ou ID de um ou mais contêineres separados por espaço;

- Para obter informações de forma programática é possível utilizar o endpoint da API do Docker - `/containers/(id)/stats`.

```
$ docker container stats [OPÇÕES] [CONTAINER...]
```

|OPÇÃO|DESCRIÇÃO|
| :---: | --- |
| --all , -a | Exibe todos os contêineres (default mostra apenas os contêineres em execução)|
| --format | Acompanha a saída do log|
| --no-stream | Desabilita o fluxo dinâmico e mostra apenas o primeiro resultado|

Exemplos:

```
# Exibe as estatísticas dos contêineres em execução

$ docker container stats

# Exibe as estatísticas dos contêineres recebendo como parâmetro seus nomes

$ docker container stats $(docker container inspect -f {{.Name}} $(docker container ls -q))
```

##### Para saber mais:

> [Referência do Docker container stats](https://docs.docker.com/engine/reference/commandline/stats/)

> [Documentação Docker - Runtime metrics](https://docs.docker.com/engine/admin/runmetrics/)

## Let's play! 
>[Playground 3](playground/play3.md)

---
<p align="left">
<a href='../unidade1/unidade1.md' id='unidade1' class='anchor' aria-hidden='true'><< Unidade 1 - Conceitos e fundamentos</a></p>
<p align="right">
<a href='../unidade3/unidade3.md' id='unidade3' class='anchor' aria-hidden='true'>Unidade 3 - Construindo Imagens >></a></p>
