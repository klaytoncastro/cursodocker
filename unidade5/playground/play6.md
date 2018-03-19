# Playground 6

## Trabalhando com Redes

Para as questões desse playground serão utilizadas as imagens do banco de dados **MySQL** disponível no [Docker Hub](https://hub.docker.com/_/mysql/) e **Wordpress**, uma ferramenta de gerenciamento de contéudo e blog largamente utilizada também disponível no [Docker Hub](https://hub.docker.com/_/wordpress/).


1. Confirme que o contêiner criado no [Playground 5](../../unidade4/playground/play5.md) esteja em execução.

2. Crie uma rede definida pelo usuário com o nome de `wpnet`.

3. Liste as redes presentes no host e verifique a criação da rede `wpnet`

4. Conecte o contêiner `wpdb` a rede `wpnet`

5. Exiba as informações detalhadas da rede `wpnet` e verifique se o contêiner `wpdb` está conectado a ela.

6. Execute um contêiner em segundo plano, a partir da imagem `wordpress`, conforme as informações abaixo:

    - Nome: `wp`

    - Mapeie a porta `8000` do host para a porta `80` do contêiner

    - Conecte o contêiner a rede `wpnet`

    - Defina as variáveis de ambiente com os seguintes valores referente a conexão com o banco de dados em execução no contêiner `wpdb`:
        - WORDPRESS_DB_HOST=wpdb:3306
        - WORDPRESS_DB_USER=wordpress
        - WORDPRESS_DB_PASSWORD=wordpress

7. Da sua máquina local acesse o endereço web referente ao host: `http:\\<IP>:8000` e execute os passos para finalização da instalação do Wordpress. Por fim, acesse a aplicação e crie o seu primeiro post.

8. Remova os contêineres `wp` e `wpdb`.

9. Recrie os contêineres `wpdb` e `wp`. Ao recriar o contêiner `wpdb` já o inclua na rede `wpnet`

10. Acesse novamente o endereço remoto do Wordpress `http:\\<IP>:8000` e verifique que os dados foram preservados no volume.


---
<p align="left">
<a href='../../unidade5/unidade5.md' id='unidade5' class='anchor' aria-hidden='true'><< Unidade 5 - Trabalhando com Redes</a></p>
<p align="right">
<a href='play6-solucao.md' id='play6-solucao' class='anchor' aria-hidden='true'>Playground 6 - Solução >></a></p>