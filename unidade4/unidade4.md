# Persistindo dados com Volumes

[Conceito e utilização](#conceito-e-utilização)

[Tipos de volumes](#tipos-de-volumes)
 
[Trabalhando com Volumes](#trabalhando-com-volumes)

## Conceito e utilização

Como já visto, o Docker faz uso do Unionfs para prover um sistema de arquivos virtual empilhável (conjunto de diretórios hierárquicos que são apresentados aos usuários como um único diretório virtual).

A utilização da tecnologia Copy-on-write (CoW), provida pelo Unionfs, é utilizada na cópia dos arquivos para a camada de leitura e escrita do contêiner, quando é necessário que eles sejam alterados. Esse comportamento pode ter impacto notável no desempenho do contêiner, especialmente se os arquivos copiados forem grandes e estejam localizados abaixo de várias camadas da imagem.

Uma outra desvantagem é que esse sistema de arquivos está vinculado ao contêiner em execução, ou seja, os dados relacionados são perdidos quando o contêiner é removido.

Para solucionar essas desvantagens, o Docker provê a solução de **volumes**.

Ao utilizar volumes, o Docker monta a pasta relacionada em um nível imediatamente inferior ao do contêiner, o que permite o
acesso rápido ao dado armazenado resolvendo o problema de performance.

Além disso, o conteúdo do volume existe fora do ciclo de vida de um determinado contêiner e também não aumenta o tamanho dos contêineres que o utilizam.

Os volumes são divididos em três categorias de montagem:

- Gerenciados pelo Docker (volumes);
- Mapeados no host (bind mount);
- Temporários (tmpfs mounts).

<p align="center">
<img src="images/types-of-mounts-volume.png" width=90% title="Categorias de montagem de Volumes">
</p>


## Tipos de volumes

### Gerenciados pelo Docker

Os volumes são o mecanismo preferido para a persistência de dados gerados e usados pelos contêineres Docker.

O uso dos volumes gerenciados pelo Docker têm várias vantagens:

- Podem ser gerenciados usando os comandos do Docker CLI ou a API Docker;
- Funcionam nos contêineres Linux e Windows;
- Podem ser compartilhados de forma mais segura entre vários contêineres;
- Os drivers de volume permitem o armazenamento de volumes em hosts remotos ou fornecedores de nuvem, a criptografia do seu conteúdo e outras funcionalidades;
- O conteúdo de um novo volume pode ser pré-preenchido por um contêiner.

### Mapeamento de pastas no host (bind mount)

Quando o mapeamento no host (bind mount) é utilizado, um arquivo ou diretório na máquina host é montado em um contêiner. O arquivo ou diretório é referenciado por seu caminho completo ou relativo na máquina host.

O arquivo ou diretório não precisa existir no host, ele é criado sob demanda se ainda não existir.

Os volumes montados no host são mais performáticos, porém dependem do sistema de arquivos da máquina host.

Não é possível utilizar os comandos do Docker CLI para gerenciar diretamente os volumes montados no host.

### Temporários (tmpfs mounts)

Volumes gerenciados pelo Docker e mapeados no host são, por padrão, montados no sistema de arquivos do contêiner e seus conteúdos são armazenados na máquina host.

Pode haver casos em que você não deseja armazenar os dados de um contêiner na máquina host, mas você também não deseja escrever os dados na camada gravável do contêiner, por razões de desempenho ou segurança. Um exemplo pode ser uma senha temporária que o aplicativo do contêiner cria e usa conforme necessário.

Para essa situação é recomendado a utilização de volumes temporários (tmpfs mounts), que permite o acesso do contêiner aos dados sem gravá-lo de forma permanente. Um volume temporário é armazenado na memória da máquina host (principal ou swap).

Quando o contêiner é finalizado, o volume temporário é removido.

Os volumes temporários não podem ser compartilhados entre contêineres e só funcionam em contêineres Linux.

## Trabalhando com Volumes

<!--Criação, exclusão, identificação, mapeamento e compartilhamento de volumes em contêineres-->

### Criação e gerenciamento de volumes

O Docker possibilita que os volumes gerenciados por ele sejam criados, excluídos e consultados por meio de seu cliente, utilizando o comando `docker volume`.

#### Criação de volumes

Sintaxe

```
$ docker volume create <NOME-DO-VOLUME>
``` 

Exemplo

```
$ docker volume create meuVolume
```

Também é possível criar volumes utilizando um driver de Volume provido por plugins de outros fabricantes. Para isso, basta incluir a opção `--driver` no comando de criação do volume. Essa [página](https://docs.docker.com/engine/extend/legacy_plugins/#volume-plugins) apresenta uma lista de diversos plugins suportados pelo Docker. 

Exemplo

```
$ docker volume create --driver=flocker volumename
```

#### Listagem de volumes

Sintaxe

```
$ docker volume ls
``` 

#### Inspecionar um volume

Sintaxe

```
$ docker volume inspect <NOME-DO-VOLUME>
``` 

Exemplo

```
$ docker volume inspect meuVolume
```

#### Remover um volume

Sintaxe

```
$ docker volume rm <NOME-DO-VOLUME>
``` 

Exemplo

```
$ docker volume rm meuVolume
```

Os volumes sem uso podem ser removido com a opção `prune`.

Exemplo

```
$ docker volume prune
```

### Mapeamento de volumes em contêineres

Conforme visto na [Unidade 2](../unidade2/unidade2.md), o mapeamento de volumes em contêineres pode ser feito com a utilização da opção `-v` ou `--volume` do comando `docker container run`. Porém a partir da versão 17.06 do Docker é possível utilizar a opção `--mount`, que originalmente era utilizada somente pelos serviços swarm.

A diferença entre elas é que a sintaxe com `-v` combina todas as opções juntas em um único campo, enquanto que `--mount` separa essas opções de forma mais explícita.

#### -v ou --volume

A sintaxe da opção `-v` ou `--volume` é posicional separada pelo caracter dois pontos (`:`) e cada campo varia de significado dependendo do tipo de volume que se quer utilizar.

Sintaxe para volumes gerenciados pelo Docker
```
docker container run -v [NOME-DO-VOLUME:]CONTAINER_DIR[:<OPÇÕES>]

NOME-DO-VOLUME      Indica o nome atribuído ao volume, que deve ser único (opcional)
CONTAINER_DIR       Caminho onde o arquivo ou diretório será montado no contêiner
OPÇÕES              [rw|ro]  leitura e escrita | somente leitura
                    [z|Z]    compartilhado | privado
                    [nocopy] desativa a cópia automática do contéudo do contêiner para o volume
```

Exemplo
```
$ docker container run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

Sintaxe para volumes montados no host
```
docker container run -v HOST_DIR:CONTAINER_DIR[:<OPÇÕES>]

HOST_DIR            Caminho do arquivo ou diretório na máquina host
CONTAINER_DIR       Caminho onde o arquivo ou diretório será montado no contêiner
OPÇÕES              [rw|ro]  leitura e escrita | somente leitura
                    [z|Z]    compartilhado | privado
                    [shared|slave|private...] propagação de montagem (bind propagation) 
```

Exemplo
```
$ docker container run -d \
  -it \
  --name devtest \
  -v "$(pwd)"/target:/app \
  nginx:latest
```

Sintaxe para volumes temporários
```
docker container run --tmpfs CONTAINER_DIR

CONTAINER_DIR       Caminho onde o diretório será montado no contêiner
OPÇÕES              Não permite o uso de opções de configuração
```

Exemplo
```
$ docker container run -d \
  -it \
  --name tmptest \
  --tmpfs /app \
  nginx:latest
```

#### --mount

A opção `--mount` consiste de vários pares de chave-valor (`<chave>=<valor>`), separados por vírgulas e cuja ordem das chaves não é significativa. A utilização da opção `--mount` é preferível por ser mais explícita e de fácil compreensão.

Sintaxe Geral
```
$ docker container run --mount type=<valor>,source=<valor>,target=<valor>,<OPÇÕES>

OPÇÕES   Variam de acordo com o tipo de montagem. 
```

|Chaves|Descrição|
|---|---|
|type| Define o tipo de montagem do volume. Pode assumir os valores: `volume`, `bind` e `tmpfs`.|
|source|Define a origem da montagem, o que pode ser um diretório da máquina host ou o nome de um volume|
|target|Define o diretório do contêiner onde o volume será montado|

Segue abaixo lista de opções possíveis:

|Opções|Descrição|
|---|---|
|readonly|Define o volume como somente leitura.|
|volume-opt|Define outras opções para a montagem,normalmente é utilizado para passagem de parâmetros para o driver de volume.|
|bind-propagation|Modifica o bind propagation. Pode assumir os valores `rprivate`, `private`, `rshared`, `shared`, `rslave`, `slave`.|
|tmpfs-size|O tamanho em bytes do tmpfs (por padrão é ilimitado)|
|tmpfs-mode|O modo de permissão do tmpfs em octal (por exemplo, 700 ou 0770). O modo padrão é 1777 (acesso e escrita universal)|
 
Exemplo para volumes gerenciados pelo Docker
```
$ docker container run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

Exemplo para volumes montados no host
```
$ docker container run -d \
  -it \
  --name devtest \
  --mount type=bind,source="$(pwd)"/target,target=/app,readonly \
  nginx:latest
```

Diferentemente da opção `-v`, com o uso da opção `--mount`, caso o diretório do host a ser mapeado não exista, o Docker **não o criará**.


Exemplo para volumes temporários
```
$ docker container run -d \
  -it \
  --name tmptest \
  --mount type=tmpfs,destination=/app \
  nginx:latest
```

##### Para saber mais

> [Documentação Docker - Manage data in Docker](https://docs.docker.com/storage/)

> [Documentação Docker - Use volumes](https://docs.docker.com/storage/volumes/)

> [Documentação Docker - Use bind mounts](https://docs.docker.com/storage/bind-mounts/)

> [Documentação Docker - Use bind mounts](https://docs.docker.com/storage/tmpfs/)

## Let's play! 
>[Playground 5](playground/play5.md)

---
<p align="left">
<a href='../unidade3/unidade3.md' id='unidade3' class='anchor' aria-hidden='true'><< Unidade 3 - Construindo Imagens</a></p>
<p align="right">
<a href='../unidade5/unidade5.md' id='unidade5' class='anchor' aria-hidden='true'>Unidade 5 - Trabalhando com Redes >></a></p>