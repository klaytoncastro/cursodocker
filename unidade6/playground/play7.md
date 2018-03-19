# Playground 7

## Integrando Contêineres com Docker Compose


1. Realize a instalação do Docker Compose.

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

3. Inicialize os serviços em segundo plano.

4. Da sua máquina local acesse o endereço web referente ao host: `http:\\<IP>:8000`.

5. Verifique os serviços em execução.

6. Verifique os logs dos serviços.

7. Pare somente o serviço `wordpress`

8. Pare e remova todos os contêineres dos serviços

---
<p align="left">
<a href='../../unidade6.md' id='unidade6' class='anchor' aria-hidden='true'><< Unidade 6 - Integrando Contêineres com Docker Compose</a></p>
<p align="right">
<a href='play7-solucao.md' id='play7-solucao' class='anchor' aria-hidden='true'>Playground 7 - Solução >></a></p>