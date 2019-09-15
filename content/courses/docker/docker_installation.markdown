---
title: Docker Installation
linktitle: Docker Installation
toc: true
type: docs
date: "2019-08-01"
draft: false
menu:
  docker:
    parent: Tutorial Docker
    weight: 1

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 1
---


***
## Docker
***

A base do que iremos fazer depende do docker, então nessa etapa inicial ensinarei a instalar, dar alguns exemplos de chamadas e também como criar o Dockerfile (nada mais que uma receita de bolo para alguma coisa).

***
## Instalação docker
***

Para usá-lo é necessário instalá-lo. A instância que será utilizada na Amazon é um distribuição linux própria chamada **Amazon linux 2**, ela é baseada em CentOS 7. Os sabores do linux utilizado faz com que algumas coisas nos sistema variem. Por exemplo, o comando de instalação de pacotes, o caminho de arquivos de configurações do sistema podem mudar, assim como algumas "facilidades", comandos podem existir num sabor e não em outro. Então caso procure algum tutorial de instalação, ou de solução de problemas, procure soluções de **CentoOS 7** que serão compatíveis com a instância na Amazon.

Dito isso, vamos começar a instalar o Docker para um sabor baseado em CentOS7.

Para evitar problemas, vamos primeiro garantir que não haja nada de docker instalado na máquina.

```sh
sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

Este comando tem o seguinte significado:

- `sudo`: Tem o significado de "faça" (`do`) como "super usuário" (`su`). O super usuário nada mais é que o administrador da máquina, tem plenos poderes sobre ela. Normalmente quando executado alguma coisa como super usuário não há qualquer pergunta (caso execute o comando para deletar todo o sistema, não haverá qualquer pergunta, a execução será direta). Outra opção ao `sudo` seria logar como *SU* e executando o comando posterior (em distribuições como Debian isso é necessário, pois não há comando `sudo`).

- `yum`: Este é um programa que gerencia os pacotes instalados do CentOS. Caso fosse uma distribuição baseada em Debian como Ubuntu seria `apt`ou `apt-get`. Se fosse baseado em Arch seria `pacman`.

- `remove`: Este é um argumento para o programa `yum`. Este argumento informa que qualquer pacote descrito a frente deve ser removido.
- `\`: Esta barra invertida é apenas uma quebra de linha para facilitar a visualização. Normalmente o comando existe em apenas uma única linha.

Vamos instalar o Docker via repositório. O repositório é um "espaço" em que seu OS "confia" para buscar pacotes e instalá-los na máquina. Normalmente o OS linux quando instalado já possui uma lista de repositórios confiáveis e oficiais em que pode contar, cada distribuição possui sua própria coleção de repositórios padrão. Aqui vamos adicionar um repositório nesta lista, o do docker-ce. Para isso, inicialmente será instalado alguns programas auxiliares ao `yum`, para podermos adicionar o novo repositório.


```sh
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```

Agora adicionando o repositório:


```sh
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

Utilizando o programa auxiliar `yum-config-manager` adicionamos o repositório na lista do `yum`. O argumento `--add-repo` é um argumento de chamada para o programa, logo após com especificamos qual o valor deste argumento, o link para o repositório.

Agora que temos o repositório onde o docker-ce existe podemos instalá-lo da seguinte forma (semelhante ao que já utilizamos antes):


```sh
sudo yum install docker-ce docker-ce-cli containerd.io
```

O docker foi instalado, porém não iniciado, para isso:


```sh
sudo systemctl start docker
```

- `sudo`: Faça como super usuário

- `systemctl`: É o programa que organiza os processos do sistema

- `start`: É um argumento para o programa `systemctl`, dizendo para iniciar o programa de nome a frente.

Neste passo o Docker está iniciado. Mas há **2 poréns**, o primeiro é que sempre que a máquina reiniciar o processo ficará parado, o segundo é que apenas o super usuário possui permissão de executar o docker.

Resolvendo o primeiro porém:


```sh
sudo systemctl enable docker
```

Resolvendo o Segundo porém:

- Uma forma é sempre usar `sudo` para executar comandos docker

- A segunda é dar permissão ao seu usuário para executar comandos docker


```sh
sudo usermod -aG docker $USER
```

Normalmente já existe um grupo chamado "docker" criado na hora da instalação. O que fazemos nesse comando é adicionar o nosso usuário `$USER` no grupo docker. Eu prefiro utilizar sempre o comando `sudo` para executar os comandos docker.

Vamos testar o docker:


```sh
sudo docker run hello-world
```

Este comando executa uma imagem chamada `hello-world` (caso ela não exista no seu PC, o docker faz o download dela no repositório oficial do docker, o docker hub (não confundir com repositório de pacotes do sistema)).

***
## Instalação docker-compose
***

O docker-compose ajuda na orquestração de uma imagem docker. Para executar uma imagem docker há vários parâmetros a serem setados para um comando só (como veremos adiante). Com o docker-compose é possível reescrever todos esses comandos em um único arquivo que pode ser versionado. Em um único arquivo, também é possível 'levantar' várias imagens diferentes, com seus próprios argumentos e configuração de como essas imagens irão conversar.

A instalação do docker-compose é diferente das demais. Ainda não há um pacote fechado para ele, o que fazemos nada mais é do que fazer o download de seus binários para um local do sistema e dar permissões a esses arquivos.


```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

- `curl`: O curl é basicamente um navegador web para a linha de comando. Ele baixa o arquivo e o coloca no caminho especificado no output
- `$(uname -s)` e `$(uname -m)`: São variáveis do sistema, o primeiro retorna "Linux" o segundo  a arquitetura "x86_64".

Agora é necessário dar permissão de execução para o arquivo:


```sh
sudo chmod +x /usr/local/bin/docker-compose
```

- `chmod`: Este comando altera permissões
- `+x`: Este é um argumento para o `chmod` o `x` significa 'executável'.


Testando a instalação:


```sh
docker-compose --version
```

AEWW!!! Docker e Docker-compose instalados, agora vamos a exemplos:

***
## Exemplos: Docker
***

Vamos iniciar por exemplos simples, ver alguns comandos, subir algumas imagens simples e pequenas, subir alguma imagem de alguma aplicação visual (algo mais tátil), construir algum Dockerfile.

<br><br>

