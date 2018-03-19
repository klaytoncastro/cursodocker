# Playground 7

## Integrando Contêineres com Docker Compose


1. Realize a instalação do Docker Compose.

    - Realize o download do binário a partir do repositório do GitHub:

    ```bash
    $ curl -L https://github.com/docker/compose/releases/download/1.19.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
    ```

    - Inclua permissão de execução para o arquivo:

    ```bash
    $ chmod +x /usr/local/bin/docker-compose
    ```

    - Teste a instalação:

    ```bash
    $ docker-compose --version
    ```

2. Crie um diretório chamado `wp` na sua máquina. Dentro do diretório criado, edite um compose-file para a integração dos serviços do **Wordpress** e **MySQL** conforme especificado abaixo:

    
- Serviço: `db`
- Imagem: `mysql:5.7`
- Monte o volume `dbvol`, a partir da pasta corrente do host (use a variável de ambiente `PWD`) para a pasta `/var/lib/mysql` do contêiner
- Defina as variáveis de ambiente com os seguintes valores:
    - MYSQL_ROOT_PASSWORD=mysql
    - MYSQL_DATABASE=wordpress
    - MYSQL_USER=wordpress
    - MYSQL_PASSWORD=wordpress
- Conecte o contêiner a rede `inet`
- Defina a política de inicialização para que o serviço reinicialize sempre

<br>

- Serviço: `wordpress`
- Imagem: `wordpress`
- Mapeie a porta `8000` do host para a porta `80` do contêiner
- Defina as variáveis de ambiente com os seguintes valores referente a conexão com o banco de dados em execução no contêiner `db`:
    - WORDPRESS_DB_HOST=db:3306
    - WORDPRESS_DB_USER=wordpress
    - WORDPRESS_DB_PASSWORD=wordpress
- Conecte o contêiner a rede `inet`
- Defina que esse contêiner depende do serviço `db`
- Defina a política de inicialização para que o serviço reinicialize sempre

```
$ mkdir wp && cd wp

$ vim docker-compose.yml
```
Conteúdo do arquivo `docker-compose.yml`:

```YAML
version: '3'

services:
   db:
     image: mysql:5.7
     volumes:
       - $PWD/dbvol:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: mysql
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress
     restart: always       
     networks:
       - inet

   wordpress:
     image: wordpress
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
     restart: always       
     networks:
       - inet

networks:
    inet:
```


3. Inicialize os serviços em segundo plano.

```
$ docker-compose up -d
```

4. Da sua máquina local acesse o endereço web referente ao host: `http:\\<IP>:8000`.

5. Verifique os serviços em execução.

```
$ docker-compose ps
```

6. Verifique os logs dos serviços.

```
$ docker-compose logs 
```

7. Pare somente o serviço `wordpress`

```
$ docker-compose stop wordpress
```

8. Pare e remova todos os contêineres dos serviços

```
$ docker-compose down
```

---
<p align="left">
<a href='../../unidade6.md' id='unidade6' class='anchor' aria-hidden='true'><< Unidade 6 - Integrando Contêineres com Docker Compose</a></p>
<p align="right">
<a href='../../unidade7/unidade7.md' id='unidade7' class='anchor' aria-hidden='true'>Unidade 7 - Introduzindo orquestração de contêineres >></a></p>