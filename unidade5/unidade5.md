# Trabalhando com Redes

[Funcionamento da rede do Docker](#funcionamento-da-rede-do-docker)

[Redes definidas pelo usuário](#redes-definidas-pelo-usuário)

- [Tipos de redes definidas por usuário](#tipos-de-redes-definidas-por-usuário)

- [Gerenciamento de redes definidas por usuário](#gerenciamento-de-redes-definidas-por-usuário)



## Funcionamento da rede do Docker

Conforme visto na [Unidade 2](../unidade2/unidade2.md), o Docker provê três redes por padrão: `bridge`, `host` e `none`. Essas redes oferecem configurações específicas para gerenciamento do tráfego de dados:

|REDE|DESCRIÇÃO|
| :---: | --- |
| none | Nenhuma rede no contêiner|
| bridge | Conecta o contêiner a rede bridge via interfaces veth (virtual ethernet)|
| host | Usa a pilha de rede do host dentro do contêiner|

O gerenciamento das redes do Docker pode ser feito por meio do comando `docker network`.

Para visualizar as redes disponíveis utilize a opção `ls`:

```
$ docker network ls
``` 

O modo de rede `bridge` é a rede padrão para os contêineres criados, a menos que seja escolhida outra rede por meio da opção `--network` do comando `docker container run`.

À essa rede é associada uma interface local do host chamada `docker0`, que funcionará como gateway, e definida com a configuração default da sub-rede `172.17.0.0/16`.

Para visualizar as informações detalhadas de uma rede pode ser utilizada o comando `docker network inspect NETWORK`.

Exemplo:

```
$ docker network inspect bridge
```

Os contêineres conectados à rede bridge padrão podem se comunicar entre si por endereço IP.

Não é possível, na rede bridge, a resolução dos IPs dos contêineres pelos seus nomes.

Antigamente o reconhecimento entre dois contêineres pelo nome era feita utilizando uma ligação entre eles, por meio da opção `--link` na inicialização dos contêineres. Essa opção não é mais recomendada e foi depreciada.

Exemplo da utilização de `--link`

```Bash
# Criar um contêiner com o nome "bd" a partir da imagem oficial do MySQL
$ docker container run -d --name bd -e MYSQL_ROOT_PASSWORD=123  mysql

# Criar contêiner com o nome "web" a partir da imagem oficial do PHP
$ docker container run -d --name web -p 80:80 php:7.0-apache

# Verificar que não é possível a resolução de nomes
$ docker container exec -it web ping -c 3 bd

# Apagar o contêiner "web"
$ docker container rm -f web

# Recriar o contêiner "web" com um link para "bd"
$ docker container run -d --name web --link bd -p 80:80 php:7.0-apache

# Testar a resolução do nome de "bd" por "web"
$ docker container exec -it web ping -c 3 bd
```

A ligação entre contêineres é uma opção muito limitada quando é necessário lidar com mais de dois contêineres. Como solução o Docker permite a criação das redes próprias ou definidas pelo usuário (user-defined).

##### Para saber mais

> [Documentação Docker - Network overview](https://docs.docker.com/network/)

> [Documentação Docker - Legacy container links](https://docs.docker.com/network/links/)

## Redes definidas pelo usuário

O Docker possibilita a criação de redes próprias ou definidas pelo usuário, além das redes padrão já vistas.

Podem ser criadas quantas redes forem necessárias e adicionar contêineres a uma ou mais dessas redes.

É importante notar que os contêineres podem se comunicar com os outros quando pertencem à mesma rede, mas a comunicação entre redes não é permitida. Para os casos em que um contêiner necessita trocar informações com um contêiner em outra rede, é possível adicioná-lo também àquela rede.

As redes definidas são criadas a partir de `drivers`. Os drivers podem ser desenvolvidos por terceiros ou escolhidos, a partir das opções disponibilizadas na instalação do Docker. São elas:

- Bridge
- Overlay
- MACVLAN


### Tipos de redes definidas por usuário

#### Bridge

Uma rede bridge é o tipo de rede mais comum usado no Docker. 

<p align="center">
<img src="images/bridge.png" width=80% title="Esquema de funcionamento da rede bridge definida por usuário">
</p>

As redes bridge são semelhantes à rede "bridge" padrão. A mais importante diferença é que, nas redes bridge definidas pelo usuário, os contêineres podem utilizar o DNS interno do Docker para resolução de nomes e respectivos IPs dos membros dessa rede.

Os contêineres iniciados nesta rede devem residir no mesmo host Docker. A comunicação entre os contêineres da mesma rede é imediato, porém a rede isola os contêineres de redes externas.

<p align="center">
<img src="images/network_access.png" width=80% title="Esquema de funcionamento da rede bridge definida por usuário">
</p>


#### Overlay

As redes overlay conectam múltiplos daemons Docker juntos e permitem que os serviços swarm se comuniquem uns com os outros.

As redes overlay podem ser utilizadas para facilitar a comunicação entre um serviço swarm e um contêiner autônomo, ou entre dois contêineres independentes em diferentes daemons do Docker. Essa estratégia remove a necessidade de fazer o roteamento no nível do sistema operacional entre esses contêineres.

Caso o Docker Engine não esteja no modo swarm, a rede overlay requer um serviço de armazenamento de chave-valor válido. Os serviços de chave-valor suportados incluem Consul, Etcd e ZooKeeper. O Docker em modo swarm não é compatível com armazenamento de chave-valor externo. Esta maneira de usar redes overlay não é recomendada.

<p align="center">
<img src="images/overlay.png" width=80% title="Esquema de funcionamento da rede overlay definida por usuário">
</p>

#### MACVLAN

Algumas aplicações, especialmente aplicativos legados ou aplicativos que monitoram o tráfego de rede, esperam estar diretamente conectados à rede física.

Neste tipo de situação, pode-se usar o driver de rede macvlan para atribuir um endereço MAC à interface de rede virtual de cada contêiner, fazendo com que pareça ser uma interface de rede física diretamente conectada à rede física. Neste caso, é preciso designar uma interface física no seu host Docker para usar no Macvlan, bem como a sub-rede e o gateway do Macvlan.

<p align="center">
<img src="images/macvlan.png" width=80% title="Esquema de funcionamento da rede macvlan definida por usuário">
</p>

### Gerenciamento de redes definidas por usuário

#### Criar uma rede 

A criação de uma rede é feita por meio do comando `docker network create`.

Sintaxe
```
docker network create [OPÇÕES] NETWORK

Opções:

--driver, -d	Driver para gerenciar a rede (padrão: bridge)
```

Exemplo
```
$ docker network create -d bridge br0
```

#### Excluir uma rede

A exclusão de uma ou mais redes é feita por meio do comando `docker network rm`.

Sintaxe
```
docker network rm NETWORK [NETWORK...]
```

Exemplo
```
$ docker network rm my-network
```

#### Conectar um contêiner a uma rede

Para conectar um contêiner a uma rede na sua criação é utilizada a opção `--network` do comando `docker container run`.

Exemplo
```Bash
$ docker container run -itd --network=br0 --name c1 alpine
```

Para conectar um contêiner já existente a uma rede utilize o comando `docker network connect`

Sintaxe
```Bash
$ docker network connect NETWORK CONTAINER
```

Exemplo
```Bash
$ docker network connect br0 bd
```

#### Desconectar um contêiner de uma rede

Para desconectar um contêiner de uma rede utilize o comando `docker network disconnect`

Sintaxe
```Bash
$ docker network disconnect [OPÇÕES] NETWORK CONTAINER

Opções:

--force , -f		Força a desconexão de um contêiner de uma rede
```

Exemplo
```Bash
$ docker network disconnect br0 bd
```

#### Remover redes não utilizadas

Para remover redes não utilizadas use o comando `docker network prune`

Sintaxe
```Bash
$ docker network prune [OPÇÕES] 

Opções:

--filter		 Possibilita filtros (p.ex. 'until=')
--force , -f	 Não solicita confirmação da operação
```

Exemplo
```Bash
$ docker network prune
```

##### Para saber mais

> [Documentação Docker - Docker container networking](https://docs.docker.com/engine/userguide/networking/)

> [Documentação Docker - Use bridge networks](https://docs.docker.com/network/bridge/)

> [Documentação Docker - Use Macvlan networks](https://docs.docker.com/network/macvlan/)

> [Documentação Docker - Use overlay networks](https://docs.docker.com/network/overlay/)

> [Documentação Docker - Networking with overlay networks](https://docs.docker.com/network/network-tutorial-overlay/)

> [UNDERSTANDING DOCKER NETWORKING DRIVERS AND THEIR USE CASES](https://blog.docker.com/2016/12/understanding-docker-networking-drivers-use-cases/)

## Let's play! 
>[Playground 6](playground/play6.md)
---
<p align="left">
<a href='../unidade4/unidade4.md' id='unidade4' class='anchor' aria-hidden='true'><< Unidade 4 - Persistindo dados com Volumes</a></p>
<p align="right">
<a href='../unidade6/unidade6.md' id='unidade6' class='anchor' aria-hidden='true'>Unidade 6 - Integrando Contêineres com Docker Compose >></a></p>