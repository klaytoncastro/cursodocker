# Playground 3

## Execução de contêineres

Para as questões de 1 a 7 desse playground será utilizada a imagem do aplicativo **Nginx** disponível no [Docker Hub](https://hub.docker.com/_/nginx/). O Nginx é um servidor proxy reverso de código aberto, balanceador de carga, cache HTTP e servidor web.

Para as demais questões será utilizada a imagem do aplicativo **Elasticsearch** disponível no site da [Elastic](https://www.docker.elastic.co/). Elasticsearch é um mecanismo de busca e análise distribuída, baseado em JSON, desenvolvido para escalabilidade horizontal, máxima confiabilidade e fácil gerenciamento.


1. Crie um contêiner a partir da imagem do Nginx e mapeie a porta `8080` do host para a porta `80` do contêiner. Da sua máquina local acesse o endereço web referente ao host: `http:\\<IP>:8080` .

2. Acesse o endereço acima várias vezes e acompanhe o log do contêiner.

3. Crie um contêiner, a partir da imagem do Nginx, que exponha automaticamente as portas definidas na imagem pela instrução `EXPOSE`. Verifique em qual porta o serviço está rodando e acesse o endereço correspondente.

4. Verifique os contêineres em execução. Remova o contêiner criado no exercício 1 e o recrie com a opção de reinicialização automática. Em seguida reinicie o daemon do Docker e confira novamente os contêineres em execução.

5. Remova o contêiner criado no exercício anterior. Crie no host Docker o arquivo `/tmp/html/index.html` com o seguinte conteúdo:

```html
<!DOCTYPE html>
<html>
<body>
<h1>Olá Docker!</h1>
</body>
</html>
``` 

Em seguida recrie o contêiner e mapeie a pasta criada para a pasta `/usr/share/nginx/html/` do contêiner. Acesse novamente o endereço web correspondente.

6. Inspecione as informações do contêiner acima e verifique os dados sobre montagem dos volumes.

7. Crie um novo contêiner, a partir da imagem do Nginx, que utilize a rede do host e mapeie diretamente a porta 80. Em seguida acesse o endereço correspondente.

8. Crie um contêiner baseado na imagem do elasticsearch (versão 6.2.2). Mapeie as portas 9200 e 9300 para as portas correspondentes do host. Indique variável de ambiente com o valor `discovery.type=single-node`. Também indique a variável `cluster.name` com um nome à sua escolha. Acesse o endereço `http://<IP>:9200` e verifique as informações do serviço (inclusive o nome do cluster). 

9. Monitore o uso de memória do contêiner (use a opção de estatísticas do Docker).

10. Elimine o contêiner criado no exercício 8. Recrie o contêiner limitando a memória a 500MB e desabilite o uso de swap. Acompanhe novamente o uso de memória do contêiner.

---
<p align="left">
<a href='../unidade2.md' id='unidade2' class='anchor' aria-hidden='true'><< Unidade 2 - Ciclo de vida de Contêineres</a></p>
<p align="right">
<a href='play3-solucao.md' id='play3-solucao' class='anchor' aria-hidden='true'>Playground 3 - Solução >></a></p>