# Playground 5

## Persistindo dados com Volumes

Para as questões desse playground será utilizada a imagem do banco de dados **MySQL** disponível no [Docker Hub](https://hub.docker.com/_/mysql/).


1. Crie um volume com o nome `wpdbvol`.

2. Liste os volumes presentes no host e verifique a criação do volume `wpdbvol`

3. Verifique as informações detalhadas do volume criado acima.

4. Execute um contêiner em segundo plano, a partir da imagem `mysql:5.7`, conforme as informações abaixo:

    - Nome: `wpdb`

    - Monte utilizando a sintaxe `--mount` o volume `wpdbvol` para a pasta `/var/lib/mysql` do contêiner

    - Defina as variáveis de ambiente com os seguintes valores:
        - MYSQL_ROOT_PASSWORD=mysql
        - MYSQL_DATABASE=wordpress
        - MYSQL_USER=wordpress
        - MYSQL_PASSWORD=wordpress

5. Verifique o funcionamento do container acessando a console do MySQL. 

    - Dica: `mysql -D wordpress -u wordpress -p -e 'show databases'`


---
<br>
<p align="left">
<a href='../unidade4.md' id='unidade4' class='anchor' aria-hidden='true'><< Unidade 4 - Persistindo dados com Volumes</a></p>
<p align="right">
<a href='play5-solucao.md' id='play5-solucao' class='anchor' aria-hidden='true'>Playground 5 - Solução >></a></p>