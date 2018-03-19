# Playground 2 - Solução

## Comandos básicos

1. Verifique a versão do Docker.
```
$ docker version
```

2. Exiba as informações de instalação do Docker.
```
$ docker system info
```

3. Baixe a imagem `Alpine` do Docker Hub.
```
$ docker image pull alpine
```

4. Verifique as imagens Docker presentes no host.
```
$ docker image ls
```

5. Liste as informações referentes a imagem `Alpine`.
```
$ docker image inspect alpine
```

6. Verifique o histórico de criação da imagem `Alpine`.
```
$ docker image history alpine
```

7. Crie um contêiner a partir da imagem `Alpine` e execute o comando `uname -a` sem utilizar o modo interativo.
```
$ docker container run alpine uname -a
```

8. Crie um contêiner a partir da imagem `Alpine` e execute o comando `ls -l` sem utilizar o modo interativo. Esse contêiner deve ser removido automaticamente após a execução.
```
$ docker container run --rm alpine ls -l
```

9. Crie um contêiner a partir da imagem `Alpine` em modo desconectado, isto é, rodando em background.
```
$ docker container run -d -t alpine
```

10. Verifique se existe algum contêiner em execução. Verifique as informações de ID, IMAGE, STATUS e NAMES.
```
$ docker container ls
```

11. Crie um contêiner a partir da imagem `Debian` em modo desconectado e defina o nome do contêiner para "container1".
```
$ docker container run -d -t --name=container1 debian
```

12. Liste as informações referentes ao contêiner "container1".
```
$ docker container inspect container1
```

13. Liste todos os contêineres presentes na máquina, inclusive os que não estão em execução. Verifique as informações exibidas.
```
$ docker container ls -a
```

14. Execute o comando `hostname` no contêiner "container1" criado acima, mas sem acessar o terminal (tty) do contêiner.
```
$ docker container exec container1 hostname
```

15. Acesse a console (tty) do contêiner em execução criado a partir da imagem `Alpine`. Mova o diretório /bin para /bug e saia do terminal.
```
$ docker container exec -it <id do container> /bin/sh

# mv /bin /bug
# exit
```

16. Pare o contêiner indicado no exercício acima.
```
$ docker container stop <id do container>
```

17. Tente iniciar o referido contêiner. O que aconteceu?
```
$ docker container start <id do container>
```

18. Tente criar um novo contêiner com o nome "container1" com o comando executado anteriormente. Qual o erro apresentado?
```
$ docker container run -d -t --name=container1 debian
```

19. Remova o contêiner "container1". Em caso de erro, o que é necessário ser feito?
```
$ docker container rm container1

$ docker container stop container1

$ docker container rm container1

OU

$ docker container rm -f container1

```
 
20. Remova a imagem `Alpine`. Em caso de erro, o que é necessário fazer para removê-la?
 ```
$ docker image rm alpine

$ docker container stop $(docker container ls --filter ancestor=alpine -q)

$ docker container rm $(docker container ls --filter ancestor=alpine -qa)

$ docker image rm alpine

OU

$ docker container rm -f $(docker container ls --filter ancestor=alpine -qa)

$ docker image rm alpine

```

---
<p align="left">
<a href='../unidade2.md' id='unidade2' class='anchor' aria-hidden='true'><< Unidade 2 - Ciclo de vida de Contêineres</a></p>
<p align="right">
<a href='../../unidade3/unidade3.md' id='unidade3' class='anchor' aria-hidden='true'>Unidade 3 - Construindo Imagens >></a></p>