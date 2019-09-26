---
title: Strata Data Conference Securing data lakes for heavy privacy regulation
author: Navarro Rosa
date: '2019-09-24'
slug: strata-data-conference-regulation
categories:
  - evento
  - nyc
  - strata
tags: []
subtitle: ''
summary: ''
authors: []
lastmod: '2019-09-24T01:40:50Z'
featured: no
image:
  caption: ''
  focal_point: ''
  preview_only: no
projects: []
---


<br><br>

Oláá, olá a todos, espero que estejam ótimos!

Trago um resumo de uma sessão que assisti no Strata Data Conference NY. O nome da sessão é: Securing data lakes for heavy privacy regulation. Foi apresentado por:

- Ifigeneia Derekli
- Mark Donsky
- Michael Ernest
- Lars George

Tudo que aqui escrevo é o meu entendimento sobre o que foi apresentado.



Em todo o mundo está surgindo regulações em relação aos dados de pessoas. De fato isso é importante para proteção dos indivíduos em relação como suas informações são acessadas, utilizadas e vendidas.

Estas regulações tem um forte viés na segurança, evitando que os dados sensíveis dos usuários vazem. Também na liberdade do indivíduo poder escolher se sua informação poderá ou não ser utilizada.

Há formatos em que para alguém utilizar a informação da pessoa é necessário uma permissão prévia, tendo ela o poder de suspender a qualquer momento. Outros já permitem que os dados sejam utilizados sem a permissão prévia, mas guardando o direito do usuário suspender as atividades.

Estas regulações virão cada vez mais, com diversas abrangências e diversos requisitos. Mas se tomarmos boas práticas de agora podemos deixar a estrutura pronta para se adequar a essas novas normas. Basicamente devemos assegurar que nossos serviços tenham:

- segurança: controle de acesso, criptografia, autenticação
- whitelist: poder filtrar os dados somente para aquelas pessoas que deram permissão de uso

São coisas simples, mas que podem ser quebradas nos quatro tópicos a seguir:

- Autenticação
- Autorização
- Criptografia
- Regulação 

## Autenticação

A ideia aqui é descobrir quem você é! O Usuário precisar provar sua identidade para o sistema.
Em tempos de home office essa parte é muito importante, pois é uma porta de entrada. Esta porta deve ser aberta somente para pessoas autorizadas.

As pessoas autorizadas estão numa "lista" seleta, o usuário precisa então provar sua identidade. A identidade é um objeto único identificatório, um username, um e-mail, etc. Mas uma identidade não significa conta, uma conta pode possuir mais de uma identidade.

Agora a prova fornecida para atestar a veracidade da pessoa em relação a identidade pode ser de três tipos:

- Coisas que você sabe: uma senha.
- Coisas que você possui: um smart card, um app de tokens, etc.
- Coisas que você é: sua Iris, digitais, DNA, etc.

Quanto mais dessas acumuladas melhor para a segurança. Por isso sempre use MFA.

Está etapa é extremamente sensível, cheia de pormenores, é recomendado que não se faça uma aplicação para isso, mas sim utilizar algo já pronto e testado, um software ou serviço. Há vários serviços e programas que fazem essa parte, o importe é utilizar algo que se encaixe no que você precisa. Que seja um serviço que tenha uma equipe dando suporte, corrigindo bugs e fazendo melhorias constantemente. 

Há gerenciadores em filesystem, como o login de linux ou até mesmo no Windows. Mas isso não foi feito para ser utilizado em larga escala. Existem métodos de onde a autenticação da identidade é feita por um agente externo como LDAP. Hoje, também temos a possibilidade de sistema de autenticação como serviço, a exemplo do AWS cognito, Azure active directory, Auth0, etc.  Eles podem usar diferentes tipos de tecnologia por trás, OAuth, OpenID,  SAML. Esses caras como serviço facilitam o transito das identidades entre os recursos de uma forma fácil e segura, onde tokens são gerados na hora, com curto tempo de validade, etc. Sem decorar a porrada de senhas. É fácil identificar o acesso de algo e cortar os acessos como um todo.

## Autorização

Uma vez que o sistema sabe quem eu sou a pergunta aqui agora muda. Se eu sou quem eu sou, o que eu posso fazer?

O objetivo dessa parte é colocar cada um no seu quadrado. Há diferentes tipos de usuários. Com isso há diferentes tipos de permissões para cada usuário. Tem aqueles que podem tomar ações, enquanto outros devem apenas consumir algo, também temos o que fazem os dois. Em cima disso tudo ainda há uma questão de zonas. Em quais dados / zonas os usuários poderão consumir/ tomar ações. É necessário manter controle de todo esse  "movimento". 

Existem diversas formas de realizar esse controle, os palestrantes citaram os seguintes exemplos: 

- ACL (Access control lists): ACL é usado diretamente no objeto. S3, HDFS, Hbase, ADLS Gen2, GCs, etc. PROS: simples, da conta do recado, bem comum no mercado. CONS: baixo nível. Não pode ser aplicado holisticamente, tem que ser manejado sistema por sistema, arquivo a arquivo.

- RBAC (Role based access control): Role é uma função que tem significado para uma empresa ou grupo. Usuários ou grupos recebem uma ou mais funções. Cada Função é associada com certas permissões (políticas). As funções são extraídas da identidade, feitas depois da autenticação, permitindo que elas sejam manejadas separadamente. Exemplo: AWS IAM e google IAM. POS: simples, fácil entendimento, mais eficiência, compliance. CONS: não é dinâmico o suficiente, não é muito granular (pode ser feito, mas exige mais esforço), no final pode haver muitas funções para manejar.


- ABAC (Attribute based access control): Atributo é algo que descreve o usuário/grupo/dados/contexto. É Mais dinâmico que o RBAC, se o atributo muda as permissões o segue. POS: dinâmico, granularidade baixa, eficiência, compliance. CONS: bem mais complexo que RBAC, pode continuar com muitas políticas.


O importante é poder ter ao mesmo tempo granularidade nas permissões e controle sobre as permissões. Isto para que se tenha um sistema flexível e seguro. Caso tenha uma falha nos acessos que seja possível identificá-la de forma rápida e resolvê-la sem grandes efeitos colaterais.

As políticas podem cuidar disso, mas as permissões de visualização deve descer ao nível da linha do dado. Imagine que, uma pessoa permita que você possa utilizar seus dados, mas não sua filial na Europa. Então as visualizações "europeias" não devem receber esses dados em específico. 

Numa arquitetura de banco de dados, é uma boa prática nunca exibir as tabelas bases para ninguém. O usuário/analista/cientista de dados/executivo que for acessar estes dados devem o fazer via uma camada que verifica seus acessos. Assim a pessoa que irá trabalhar com estes dados irá ver uma versão da tabela base filtrada por uma tabela de controle de acesso. 

## Criptografia

Confesso que este foi o ponto da palestra em que menos me sinto confortável para falar. Por isso irei apenas passar por cima, num modo mais conceitual.

A pergunta aqui não é "Devemos criptografar os dados?", mas sim "Quando criptografar os dados?". A criptografia oferece proteção e confidencialidade aos nossos dados, dados criptografados mesmo que vazados estão seguros (desde que capturou não tenha a chave de decodificação). É necessário manter integridade dos dados (evitando perdas acidentais) e a confidencialidade. Com a técnica apropriada é possível manter os dois. 

Quando transferimos dados de um lado para outro nós estamos utilizando fios, literalmente. Caso estes fios estejam numa rede pública, na internet por exemplo, eles são acessíveis para outras pessoas. Caso não seja utilizado algum tipo de criptografia em trânsito os dados estão sendo transferidos abertamente. Qualquer um com acesso nesta rede pública pode capturar estes dados sendo transferidos e observar o que eles realmente são. Agora, utilizando algum tipo de criptografia de transferência os dados ainda podem ser "capturados", porém não são recuperáveis, com exceção das pontas (aquele que enviou o dados e aquele que deve receber). Basicamente o que é capturado é uma colação de bytes aleatórios. 

Quando os dados estão parados (at rest) também é importante criptografar os dados. Pois isso protege os dados de pessoas que já estão na rede, pode ser um usuário interno mal intencionado, ou até mesmo uma invasão.

Porém tudo isso vem a um custo. Para podermos manter os dados criptografados é necessário que seja feito um manejamento das chaves de criptografia. São por estas que é possível criptografar e descriptografar os dados. E isto é de extrema importância, veja, se alguém tiver acesso às chaves ele terá acesso aos dados. Caso alguém se atrapalhe e as chaves sejam perdidas você nunca mais terá acesso a aqueles dados. Então é necessário que tenha um processo de governança sobre estas chaves, deixando elas num local diferentes dos dados, com expiração e rotação, com backups, etc.

## Regulação

Não é possível prever todos os detalhes de um nova regulação. Porém com boas práticas podemos estar preparados para elas. Fornecendo segurança, controle de acesso linha a linha para usuários são exemplos disso.

E necessário conhecer conhecer quais dados você possui no seu data lake. Quais são os dados sensíveis, quais conjunto de dados, mesmo que não sejam sensíveis possam identificar um indivíduo. Crie catálogos de todos os dados, crie tags para cada coluna, metadados EVERYWHERE. 

Como os dados estão sendo usados? Por quem os dados estão sendo usados.

Saber a linha temporal dos dados. De quando ele entra até quando ele sai para o usuário final, quais são as etapas de transformações. Guarde o estado de cada cálculo, entrada e saída, hoje o custo de armazenamento de dados é o mais baixo da história. Podemos guardar todo o histórico de transformação. Está é uma etapa necessária, já que muitas das regulações que estão surgindo ao redor do mundo exigem que isto seja feito e por um longo período de tempo.


SEMPRE devemos pensar na segurança por default. Hoje, muitos sistema são construídos para então serem adicionados etapas de segurança. Isto deve ser invertido, devemos pensar nos sistemas e nas soluções de forma a fornecer o máximo de segurança. Encriptar dados, controle de acesso, anonimização pseudoanonimização (xuberar), acesso fino aos dados.

Sempre implementar mecanismos de validar o consentimento das pessoas que de fato são os donos de suas informações pessoais. Toda aplicação deve receber os dados após a validação do consentimento, nunca dos dados base.
