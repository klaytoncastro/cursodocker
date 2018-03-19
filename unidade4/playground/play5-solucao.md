# Playground 5 - Solução

## Persistindo dados com Volumes

Para as questões desse playground será utilizada a imagem do banco de dados **MySQL** disponível no [Docker Hub](https://hub.docker.com/_/mysql/).


1. Crie um volume com o nome `wpdbvol`.

```
$ docker volume create wpdbvol
```

2. Liste os volumes presentes no host e verifique a criação do volume `wpdbvol`

```
$ docker volume ls
```

3. Verifique as informações detalhadas do volume criado acima.

```
$ docker volume inspect wpdbvol
```

4. Execute um contêiner em segundo plano, a partir da imagem `mysql:5.7`, conforme as informações abaixo:

    - Nome: `wpdb`

    - Monte utilizando a sintaxe `--mount` o volume `wpdbvol` para a pasta `/var/lib/mysql` do contêiner

    - Defina as variáveis de ambiente com os seguintes valores:
        - MYSQL_ROOT_PASSWORD=mysql
        - MYSQL_DATABASE=wordpress
        - MYSQL_USER=wordpress
        - MYSQL_PASSWORD=wordpress


```
$ docker container run -d \
--mount source=wpdbvol,target=/var/lib/mysql \
-e MYSQL_ROOT_PASSWORD=mysql \
-e MYSQL_DATABASE=wordpress \
-e MYSQL_USER=wordpress \
-e MYSQL_PASSWORD=wordpress \
--name wpdb mysql:5.7
```

5. Verifique o funcionamento do container acessando a console do MySQL. 

    - Dica: `mysql -D wordpress -u wordpress -p -e 'show databases'`

```
$ docker container exec -it wpdb mysql -D wordpress -u wordpress -p -e 'show databases'
```

---
<p align="left">
<a href='../unidade4.md' id='unidade4' class='anchor' aria-hidden='true'><< Unidade 4 - Persistindo dados com Volumes</a></p>
<p align="right">
<a href='../../unidade5/unidade5.md' id='unidade5' class='anchor' aria-hidden='true'>Unidade 5 - Trabalhando com Redes >></a></p>