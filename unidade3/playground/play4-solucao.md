# Playground 4 - Solução

## Construindo Imagens

Para as questões desse playground será utilizada a imagem do aplicativo **Nginx** disponível no [Docker Hub](https://hub.docker.com/_/nginx/). O Nginx é um servidor proxy reverso de código aberto, balanceador de carga, cache HTTP e servidor web.

O propósito desse playground é criar uma nova imagem, tendo como base uma imagem de servidor web, que hospede uma página estática. Instanciaremos um contêiner a partir da imagem criada para acesso a referida página.

#### Preparação do ambiente

1. Crie na sua máquina um diretório com o nome `oladocker`

2. Crie o arquivo `ola_docker.html` com o seguinte conteúdo:

```HTML
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
</head>
<body bgcolor=BGCOLOR>
<img src="imagens/baleia.png" alt="Oi!" align=left width=30% height="30%" hspace="20">
<h1>Olá ALUNO!</h1>
<p>Essa página está rodando a partir de um container <b>Docker</b></p>
</body>
</html>
```

3. Dentro desse diretório, crie outro diretório com o nome `imagens` e copie para ele o arquivo [baleia.png](images/baleia.png).

4. Crie um arquivo chamado `start.sh` (na raiz da pasta `oladocker`) com o seguinte conteúdo:

```bash
#!/bin/bash
sed -i -e s/ALUNO/"$ALUNO"/g index.html && \
sed -i -e s/BGCOLOR/"$BGCOLOR"/g index.html && \
nginx -g 'daemon off;'
```


#### Criação do Dockerfile e construção da imagem

Será utilizada a imagem do aplicativo **Nginx** disponível no [Docker Hub](https://hub.docker.com/_/nginx/). O Nginx é um servidor proxy reverso de código aberto, balanceador de carga, cache HTTP e servidor web.

1. Crie o arquivo com o nome `Dockerfile` dentro do diretório `oladocker` com as instruções referentes aos passos abaixo:

    a. Utilizar a imagem do aplicativo Nginx como base

    b. Copie o arquivo `ola_docker.html` para o diretório `/usr/share/nginx/html/` da imagem

    c. Renomeie o arquivo copiado no passo acima para `index.html`

    c. Copie o diretório `imagens` para o diretório `/usr/share/nginx/html/` da imagem

    d. Defina váriaveis de ambiente `ALUNO` e `BGCOLOR` dentro da imagem

    e. Defina o diretório `/usr/share/nginx/html/` da imagem como o diretório de trabalho

    f. Copie o arquivo `start.sh` para a pasta corrente da imagem

    g. Inclua permissão de execução para o arquivo `start.sh` copiado para a imagem

    h. Defina o comando de inicialização do contêiner para o script `start.sh`

```Dockerfile
FROM nginx
COPY ola_docker.html /usr/share/nginx/html/
RUN cd /usr/share/nginx/html/ && mv ola_docker.html index.html
COPY imagens/ /usr/share/nginx/html/imagens
ENV ALUNO ""
ENV BGCOLOR ""
WORKDIR /usr/share/nginx/html
COPY start.sh .
RUN chmod +x start.sh
CMD ./start.sh
```

2. Em seguida construa a imagem com o nome `oladocker`.

```bash
$ docker image build -t oladocker .
```

3. Execute um contêiner a partir da imagem criada e mapeie a porta `8080` do host para a porta `80` do contêiner. Inclua valores para as variáveis `ALUNO` e `BGCOLOR`.

```Bash
$ docker container run -d -p 8080:80 -e BGCOLOR=DodgerBlue -e ALUNO=Flavio oladocker
```

4. Da sua máquina local acesse o endereço web referente ao host: `http:\\<IP>:8080`

#### Push da imagem para o Docker Hub

1. Identifique a imagem com o seu usuário do Docker Hub.

```
$ docker image tag oladocker flaviohrocha/oladocker
```

2. Execute o comando `docker login` e informe suas credenciais.

```
$ docker login
```

3. Realize o push da sua imagem.

```
$ docker image push flavioh/oladocker
```

---
<p align="left">
<a href='../unidade2/unidade2.md' id='unidade2' class='anchor' aria-hidden='true'><< Unidade 2 - Ciclo de vida de Contêineres</a></p>
<p align="right">
<a href='../../unidade4/unidade4.md' id='unidade4' class='anchor' aria-hidden='true'>Unidade 4 - Persistindo dados com Volumes >></a></p>