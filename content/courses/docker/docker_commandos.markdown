---
title: Docker Commands
linktitle: Docker Commands
toc: true
type: docs
date: "2019-08-01"
draft: false
menu:
  docker:
    parent: Tutorial Docker
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---

***
## Comandos: run
***


O comando run, quando não utilizado com outra ferramente de orquestração, como docker-compose, kubernets, ou outro, será o comando mais utilizado. Ele executa uma imagem, normalmente esta imagem executa um comando e morre. A imagem hello-world basicamente executa um print de texto no console e depois morre. Se subirmos uma API por exemplo, ela executaria todo o cálculo e morreria logo após. Se subirmos um serviço contínuo o comportamento ainda é o mesmo, porém a "sensação" é outra, por exemplo, subimos um serviço de banco de dados, o comando que a imagem terá é executar o banco, a imagem continuará ativa durante toda a "execução do comando", ou seja, a imagem só irá morrer caso o banco de dados morra por algum motivo.

Primeiro, há duas formas de executar uma imagem, no Foreground (seu console vai ficar preso no que for executado) e no Background (ou detached) a imagem será executada no plano de fundo.

Vamos subir uma imagem que seja um OS ubuntu em foreground:


```sh
sudo docker run -it ubuntu bash
```

- `run`: Comando que executa uma imagem
- `-it`: Argumentos para o comando run, `i` significa interativo, `t` ativa um tipo de buffer para entradas de texto
- `ubuntu`: O nome da imagem que será executado
- `bash`: O programa q será executado dentro da imagem, neste caso iniciaremos o console.

Note que, agora você está num ambiente ubuntu, dentro da imagem. Aqui você pode fazer qualquer coisa, inclusive deletar seu OS (pois quando restartar a imagem ela virá no mesmo formato de antes). **Faça alguns testes aqui a seu prazer**.

Para sair de dentro da imagem execute `exit`.

Vamos agora subir uma imagem mais "visual". Muitos estão falando sobre o Metabase, um visualizador gratuito. Então vamos subir uma aplicação em Foreground de Metabase para que possamos acessá-la.


```sh
sudo docker run -p 8787:3000 metabase/metabase 
```

- `-p`: Aqui temos um argumento novo, a porta. O app fica exposto numa porta, nesse caso na porta 3000. Porém essa porta é do container e nós acessamos a máquina (host), então é necessário que seja construído um caminho para a porta do container. Então, quando se acessa http://127.0.0.1:8787/setup você está acessando na verdade o container na porta 3000. Caso você esteja em alguma instância o endereço seria http://ip-da-instancia:8787/setup. (porta do host à esquerda e do container à direita)

- `metabase/metabase`: O nome da imagem, caso não tenha a imagem no PC o docker irá procurar em seu repositório

Observe seu terminal, veja que ele está preso à aplicação, caso seu terminal morra a aplicação também irá sofrer o mesmo destino. Vamos subir a aplicação no modo detached.


```sh
sudo docker run -d -p 8787:3000 metabase/metabase 
```

Observe agora que o terminal está livre e a aplicação continua rodando. E agora que a imagem já está no PC, observe a velocidade que a aplicação entra em operação.


Podemos ainda fazer:


```sh
sudo docker run  --rm -d -p 8787:3000 metabase/metabase 
```

- `--rm`: Este argumento diz que, quando o container morrer ou set parado, ele deve ser removido.


O comando run tem muitas outras opções, como por exemplo dar acesso a pastas do host para o container, declarar variáveis de ambiente, etc. Mas vamos deixar isso para depois, quando estivermos falando de docker-compose.

<br><br>

***
## Comandos: container
***


O comando container nos permite verificar informações referentes aos contêineres. O container é a aplicação/imagem que foi executada ou está em execução.

Primeiramente vamos verificar os contêineres ativos, aquelas imagens que estão em execução.


```sh
sudo docker container ls
```

- `ls`: O argumento ls significa listar:

Caso você tenha executado o exemplo detached do metabase da sessão anterior terá uma linha preenchida com as informações deste container. Verá que ele está ativo, em qual porta, qual o comando executado e a quanto tempo. 

Ao fazer:


```sh
sudo docker container ls -a
```

- `-a`: Este argumento representa "all"

É possível ver todos os contêineres que já foram executados.

Vamos então matar esse container ativo que não mais necessitamos. Ao executar `sudo docker container ls` podemos ver os ativos, observe que, cada container possui um CONTAINER ID, é por ele iremos matar o processo, basta fazer:


```sh
sudo docker container kill CONTAINER_ID
```

No meu caso o comando foi `sudo docker container kill 816e65266526`, observe com `sudo docker container ls` que o container não está mais de pé.

Há outros argumentos que podem ser utilizados junto com `container`, entre eles estão `exec`, `restart`, `pause`, `inspec`, `start`, `stop`, `unpause`, `logs`, etc. Normalmente eles estão ligados à manutenção de contêineres, sugiro que sejam estudados caso a necessidade (preciso visualizar as entranhas do meu container para observar algo, etc). Como nosso objetivo é criar uma API escalável não faremos nenhuma manutenção de container, pois eles irão nascer e morrer a todo momento.

<br><br>

***
## Comandos: image
***


Este comando tem o objetivo de gerir as imagens que existem. Inicialmente iremos utilizar bastante esse comando para poder baixar e subir imagens para o nosso repositório. Posteriormente até este processo será automático, nosso trabalho será somente versionar os scripts.

Vamos listar todas as imagens que estão salvas.


```sh
sudo image ls -a
```

Observe os tamanhos de cada imagem, todas essas imagens estão salvas no HD. A medida que são utilizadas o espaço do HD vai diminuindo. Vamos deletar a imagem do ubuntu


```sh
sudo docker rmi ubuntu
```
Caso tenha algum log de um container que a esteja utilizando podemos forçar a remoção


```sh
sudo docker rmi -f ubuntu
```

Vamos baixá-la novamente.


```sh
sudo docker image pull ubuntu
```

Falarei de como subir imagens somente após a construção da nossa primeira imagem.

<br><br>

***
## Comandos: prune
***


Este é um comando muito útil. Porém não iremos utilizá-lo em produção, apenas no nosso ambiente de Dev. Nós temos um histórico no docker, como por exemplo todos os contêineres já executados `sudo docker container ls -a`, e imagens que não mais utilizamos, ou que foram atualizadas continuam a existir. O prune tem como objetivo limpar todos esse lixo. Ele pode ser feito em etapas ou em todo o docker. Vamos por etapas.

**Container:**

Veja quantos container temos que estão parados:


```sh
sudo docker container ls -a
```

Podemos remover todos os contêineres parados com:


```sh
sudo docker container prune
```

Podemos ainda filtrar, aqui removemos todos os contêineres criados parados que possuem mais de 24 horas.

```sh
sudo docker container prune --filter="until24h"
```

**Images:**

Podemos remover imagens que são "zumbis". Essas imagens são aquelas que não são taggeadas ou não sejam referenciadas por nenhum container.


```sh
sudo docker image prune
```

Podemos remover todas as imagens que não estejam associadas a um container:

```sh
sudo docker image prune -a
```

E assim como no exemplo dos contêineres podemos utilizar filtros

```sh
sudo docker image prune --filter="until24h"
```

**Tudo:**

Podemos fazer um prune de tudo ao mesmo tempo incluindo **Networks (Não abordei networks nesse documento, mas acho pertinente fazer uma pesquisa em separa quando ir surgindo a necessidade de ligar com essas configurações, mas num resumo bem breve, podemos criar networks internas para os containers de modo que eles apenas se comuniquem num espaço fechado)**


```sh
sudo docker system prune
```

Execute esse comando e depois observe quantos contêineres você possui e quantas imagens salvas.




<br><br>

***
## Comandos: build
***

**VAMOS CONSTRUIR!!!!!**

Vamos construir nosso primeiro Dockerfile. O Dockerfile é uma receita de bolo, dizendo como será a imagem. Essa receita de bolo consiste em empilhar comandos do próprio Linux num arquivo interpretável. Existe um procedimento de como deixar as imagens menores e mais eficientes, mas isso não é importante no momento, apenas construa a seu prazer para testar as possibilidades.

O Dockerfile se baseia nos seguintes comandos:

- `FROM`: A imagem inicial que sua imagem irá se basear (normalmente algo pequeno, para um ambiente de DEV costuma ser um OS e para produção um OS mais capado possível)
- `RUN`: Aqui dizemos os comandos que serão executados antes da imagem estar pronta (instalar pacotes por exemplo)
- `MAINTAINER`: O autor do arquivo
- `COPY`: Copia algum arquivo para dentro da imagem (assim quando a imagem for iniciada ela já possui o arquivo)
- `USER`: Define qual será o usuário padrão para a imagem
- `ENTRYPOINT`: Define qual a aplicação do container. Normalmente é um arquivo em shell dentro da imagem com tudo para ser executado. É executado quanto a imagem é executada (na criação de um container).
- `CMD`: Semelhante ao ENTRYPOINT, pode executar um comando na execução da imagem, ou passar argumentos para o ENTRYPOINT. Porém o CMD pode ser "ofuscado" por algum outro comando na hora de executar a imagem, exemplo `sudo docker run --rm -it ubuntu /bin/bash`, aqui estamos executando a imagem ubuntu com o CMD `/bin/bash` que substitui qualquer outro CMD dentro da imagem.

Vamos preparar o terreno. Primeiro precisamos de uma área de trabalho para essa imagem que vamos criar. Então vamos criar um caminho de pastas para nosso Dockerfile ficar isolado e entrar nessas pasta.


```sh
mkdir teste_docker
cd teste_docker
```

- `mkdir`: Significa "make a directory", estamos criando uma pasta chamada teste_docker
- `cd`: Significa "change directory", vamos entrar na pasta teste_docker.

Estamos dentro da pasta de trabalho, agora iremos criar um arquivo de texto e começar a construir nossa imagem. Vamos utilizar o programa `nano`, caso não tenha instale:

Para CentOS

```sh
sudo yum install nano
```

Para ubuntu/debian

```sh
sudo apt-get install nano
```


Vamos criar um executável para passar para dentro da imagem.Observe que, após executar esse comando agora temos uma página em branco para ser preenchida. Para salvar o arquivo após algo escrito aperte CTRL+ O, para sair do editor nano e retornar ao terminal CTRL+X. Vamos preencher o documento.



```sh
nano text_print.sh
```

Coloque o seguinte texto:

```sh

#!/usr/bin/env bash

echo "ESTAMOS IMPRIMIIINDOO"
```


Vamos criar um arquivo chamado Dockerfile:


```sh
nano Dockerfile
```

Vamos colocar o seguinte texto:


```sh
FROM ubuntu # da imagem ubuntu

MAINTAINER eu_mesmo # autor

RUN apt-get update \
    && apt-get install -y mariadb-client \
    libmariadbclient-dev \
    libmariadb-client-lgpl-dev \
    && apt-get clean \
    && rm -rf /var/lib/apt/* \
    && rm -rf /tmp/*

COPY text_print.sh /text_print.sh

RUN chmod 755 /text_print.sh

ENTRYPOINT /text_print.sh
```

A seguir CTRL+O para salvar e CTRL+X para sair.

Acima temos:



- FROM: imagem ubuntu
- MAINTAINER: autor
- RUN: Executo uma série de comandos, primeiro dou update nos repositórios do apt-get (instalador de pacotes do ubuntu). Depois instalo 3 pacotes, cliente e bibliotecas do MariaDB. Após isso eu limpo o cache de instalação e deleto qualquer arquivo intermediário que possa ter ficado das instalações.
- COPY: Copio o arquivo que criamos anteriormente e coloco na raiz
- RUN: Dou permissão para esse arquivo ser executado
-ENTRYPOINT: Aponto o que será executado quando a subirmos o container.


Agora que temos a receita vamos construir a imagem "buildar". A pasta deve conter apenas dois arquivos, verifique:


```sh
ls
```

Deve retornar isso:

```sh
Dockerfile  text_print.sh
```

Então vamos buildar a imagem:


```sh
sudo docker build -t imagem_teste .
```

- `build`: é o comando para construir a imagem
- `-t`: é o argumento dizendo que vamos nomear a nossa imagem
- `imagem_teste`: O nome da nossa imagem
- `.`: O comando build precisa que você referencie a pasta onde está o Dockerfile e os arquivos necessários para a construção. O `.` referencia a pasta que você está no presente momento com o console.

Aguarde a imagem terminar de buildar e verifique:

```sh
sudo docker image ls
```

Caso você tenha uma conta no docker hub (repositório oficial) ou tenha algum repositório de imagens configurado (teremos um FUNCIONAL) você pode fazer o seguinte para subir a imagem para tal.


```sh
sudo docker tag imagem_teste seu_nome_usuario_repo/imagem_teste
sudo docker push seu_nome_usuario_repo/imagem_teste
```

Primeiro renomeia a imagem, colocando seu usuário como prefixo. Depois usar o push, para o upload da imagem, irá pedir um login e senha do repositório e irá iniciar o upload.

Vamos testar a imagem!!


```sh
sudo docker run imagem_teste
```

Imprimiu o que queríamos?

Vamos sair desta pasta que criamos e voltar para onde estávamos

```sh
cd ..
```

- `..`: Este símbolo significa subir uma pasta


<br><br>

