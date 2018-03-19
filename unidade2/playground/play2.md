# Playground 2

## Comandos básicos

1. Verifique a versão do Docker.

2. Exiba as informações de instalação do Docker.

3. Baixe a imagem `Alpine` do Docker Hub.

4. Verifique as imagens Docker presentes no host.

5. Liste as informações referentes a imagem `Alpine`.

6. Verifique o histórico de criação da imagem `Alpine`.

7. Crie um contêiner a partir da imagem `Alpine` e execute o comando `uname -a` sem utilizar o modo interativo.

8. Crie um contêiner a partir da imagem `Alpine` e execute o comando `ls -l` sem utilizar o modo interativo. Esse contêiner deve ser removido automaticamente após a execução.

9. Crie um contêiner a partir da imagem `Alpine` em modo desconectado, isto é, rodando em background.

10. Verifique se existe algum contêiner em execução. Verifique as informações de ID, IMAGE, STATUS e NAMES.

11. Crie um contêiner a partir da imagem `Debian` em modo desconectado e defina o nome do contêiner para "container1".

12. Liste as informações referentes ao contêiner "container1".

13. Liste todos os contêineres presentes na máquina, inclusive os que não estão em execução. Verifique as informações exibidas.

14. Execute o comando `hostname` no contêiner "container1" criado acima, mas sem acessar o terminal (tty) do contêiner.

15. Acesse a console (tty) do contêiner em execução criado a partir da imagem `Alpine`. Mova o diretório /bin para /bug e saia do terminal.

16. Pare o contêiner indicado no exercício acima.

17. Tente iniciar o referido contêiner. O que aconteceu?

18. Tente criar um novo contêiner com o nome "container1" com o comando executado anteriormente. Qual o erro apresentado?

19. Remova o contêiner "container1". Em caso de erro, o que é necessário ser feito?
 
20. Remova a imagem `Alpine`. Em caso de erro, o que é necessário fazer para removê-la?

---
<p align="left">
<a href='../unidade2.md' id='unidade2' class='anchor' aria-hidden='true'><< Unidade 2 - Ciclo de vida de Contêineres</a></p>
<p align="right">
<a href='play2-solucao.md' id='play2-solucao' class='anchor' aria-hidden='true'>Playground 2 - Solução >></a></p>