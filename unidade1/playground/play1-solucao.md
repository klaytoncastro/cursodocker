# Playground 1 - Solução

## Instalação do Docker

1. Realize a instalação do Docker Engine na máquina virtual com o sistema operacional CentOS 7.


```
# Configuração do repositório

$ yum install -y yum-utils

$ yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo  

# Instalar Docker CE

$ yum install docker-ce

$ systemctl start docker

$ docker system info

# Testar a instalação

$ docker container run hello-world
```

2. Realize a instalação do Docker Toolbox.

- [Procedimento de instalação do Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)

---
<p align="left">
<a href='../unidade1.md' id='unidade1' class='anchor' aria-hidden='true'><< Unidade 1 - Conceitos e fundamentos</a></p>
<p align="left">
<a href='../../unidade2/unidade2.md' id='unidade2' class='anchor' aria-hidden='true'> Unidade 2 - Ciclo de vida de Contêineres >></a></p>