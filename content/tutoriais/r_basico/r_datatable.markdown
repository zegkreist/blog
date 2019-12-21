---
title: R Data Table
linktitle: R - Data table 
toc: true
type: docs
date: "2019-12-20"
draft: false
menu:
  r_basico:
    parent: R Básico
    weight: 2

# Prev/next pager order (if `docs_section_pager` enabled in `params.toml`)
weight: 2
---





# R - Data.Table 
***

O data.table nada mais é do que uma reimaginação do data.frame do R. Ele é escrito primariamente em C. Possui uma série de otimizações focando em velocidade e economia de memória. Temos a liberdade de fazer operações inplace, evitando cópias desnecessárias que agridem a memória e demandam tempo.

Seja `dados` um objeto data.table. Nosso acesso as manipulações ocorrem pela função `[]`, temos a seguinte estrutura:


![Slots](data.table.png)

Temos 3 slots:

- Primeiro, `i`: É a região dos filtros
- Segundo,`j`: É a região do que iremos fazer. Criar colunas? Dropar colunas? Agregação?
- Terceiro, `by`: É a região dos grupos, mas algumas vezes pode servir como filtro também, como num caso de JOIN.

Seria algo com `dados[where, select|update|do, by]`. Vamos começar com a parte do `where`, para isso vamos introduzir a função `setkey`.

***
## Filtro
***

Já que iremos filtrar, vamos filtrar de forma eficiente.
A função `setkey` tem como objetivo estabelecer um índice. A serventia é a mesma que num banco de dados, isto é feito para que, numa busca seja feito busca binária e não vector scan. **Um ponto de atenção é que o objeto data.table é ordenado de acordo com o índice**, dependendo do que está sendo feito isto pode ser um problema.

Vamos criar um dado sintético para trabalhar de tamanho razoável.


```r
dados <- data.table(categoria = sample.int(4, 10000000, replace = T),
                    valor     = rnorm(10000000, mean = 1000, sd = 20),
                    sexo      = sample.int(2, 10000000, replace = T)
                    )
```


Temos um conjunto de dados de 10 milhões de linhas. Criamos dados fake, categoria, valor e sexo. Vamos criar um objeto que receba apenas os dados da categoria 1 e 2. Mas primeiro criaremos um índice.


```r
setkey(dados,categoria)
```

Vamos comparar o filtro utilizando a chave e sem.


```r
rbenchmark::benchmark(
  "sem_key" = {
    filtro1 <- dados[categoria == 1 | categoria == 2]
    },
  "com_key" ={
    filtro2 <- dados[.(c(1,2))]
    },
  order = "relative",
  replications = 1,
  columns = c("test", "replications", "relative", "elapsed")
  )
```

```
##      test replications relative elapsed
## 2 com_key            1    1.000   0.101
## 1 sem_key            1    1.842   0.186
```
São iguais?


```r
all(filtro1 == filtro2)
```

```
## [1] TRUE
```

A versão sem chave é mais lenta. Mas repare a velocidade com que filtramos uma tabela de 10 milhões de linhas e copiamos para um objeto novo em ambas as versões.

**Agora vamos explicar melhor.**

Quando não temos uma chave criada, filtramos o conjunto de dados utilizando vetores lógicos.


```r
dados[categoria == 1]
```

```
##          categoria     valor sexo
##       1:         1  983.1805    2
##       2:         1 1020.7571    2
##       3:         1 1041.2756    1
##       4:         1  984.5988    2
##       5:         1  981.9217    2
##      ---                         
## 2501239:         1 1018.7187    2
## 2501240:         1 1028.5804    1
## 2501241:         1 1045.4142    1
## 2501242:         1  993.8039    2
## 2501243:         1  998.1015    2
```

Perceba que, eu faço referência ao nome da coluna de forma direta internamente. Podemos compor este vetor lógico de outras formas, utilizando outras colunas também, veja.


```r
dados[categoria == 1 & valor >= 1000 & sexo == 1]
```

```
##         categoria    valor sexo
##      1:         1 1041.276    1
##      2:         1 1012.897    1
##      3:         1 1010.089    1
##      4:         1 1018.702    1
##      5:         1 1020.278    1
##     ---                        
## 624581:         1 1025.785    1
## 624582:         1 1016.047    1
## 624583:         1 1020.169    1
## 624584:         1 1028.580    1
## 624585:         1 1045.414    1
```

Porém, quando assim fazemos, não nos utilizamos da busca binária. Para usá-la precisamos criar as chaves e usar uma lista para passar os argumentos.


```r
setkey(dados, categoria, sexo)
```

Não irei fazer busca num range de valor na busca binária, mas chegarei no mesmo resultado. Iremos fazer o mesmo filtro.


```r
dados[.(1,1)][valor >= 1000]
```

```
##         categoria    valor sexo
##      1:         1 1041.276    1
##      2:         1 1012.897    1
##      3:         1 1010.089    1
##      4:         1 1018.702    1
##      5:         1 1020.278    1
##     ---                        
## 624581:         1 1025.785    1
## 624582:         1 1016.047    1
## 624583:         1 1020.169    1
## 624584:         1 1028.580    1
## 624585:         1 1045.414    1
```

Vejamos a diferença:


```r
rbenchmark::benchmark(
  "sem_key" = {
    filtro1 <- dados[categoria == 1 & valor >= 1000 & sexo == 1]
    },
  "com_key" ={
    filtro2 <- dados[.(1,1)][valor >= 1000]
    },
  order = "relative",
  replications = 1,
  columns = c("test", "replications", "relative", "elapsed")
  )
```

```
##      test replications relative elapsed
## 2 com_key            1     1.00   0.028
## 1 sem_key            1     5.75   0.161
```

São iguais?


```r
all(filtro1 == filtro2)
```

```
## [1] TRUE
```

Observe que não estamos contando o tempo necessário para se criar as chaves. Porém quase sempre é melhor fazer a chave, pois podemos pensar numa chave que favoreça o maior número possível de filtros que será feito.

***
## Agregações / Mutate
***

Agora vamos trabalhar na segunda parte do `data.table`, no "o que iremos fazer". Aqui vamos fazer agregações. Temos 3 colunas, duas que indicam categorias e uma que indica um valor. Então, faremos alguns cálculos utilizando a coluna valor. Vamos crar um novo objeto `data.table` que tenha a média e desvio padrão deste valor.


```r
dados1 <- dados[, .(media_valor         = mean(valor),
                    desvio_padrao_valor = sd(valor)
                    )
                ]
dados1
```

```
##    media_valor desvio_padrao_valor
## 1:    1000.004            20.00367
```

O que vemos nesta expressão?

- `dados[,`: Não queremos fazer nenhum filtro, então deixamos seu respectivo espaço em branco.
- `.(`: Está função `.` é um alias para a função `list`. Estamos passando uma lista com as expressões que são feitas.
- `media_valor = mean(valor)`: Aqui nomeio a nova coluna à esquerda da igualdade. No lado direito escrevo a expressão, aqui faço a média da coluna `valor`.

Vamos fazer o mesmo procedimento, só que para valores maiores que 1000.


```r
dados1 <- dados[valor > 1000, .(media_valor         = mean(valor),
                    desvio_padrao_valor = sd(valor)
                    )
                ]
dados1
```

```
##    media_valor desvio_padrao_valor
## 1:    1015.969            12.05818
```

Ao contrário da expressão anterior, adicionamos um filtro no espaço destinado a ele. Podemos reescrever isso de outra forma, já mostrada anteriormente, fazendo primeiro um `data.table` filtrado e continuando os cálculos. Isto pode ser feito com o pipe ou usando a função `[`, a segunda é mais recomendada pois não gera cópias.


```r
# com pipe
library(magrittr)
dados1A <- dados[valor > 1000] %>% 
  .[, .(media_valor         = mean(valor),
        desvio_padrao_valor = sd(valor)
        )
    ]

#Com os []

dados1B <- dados[valor > 1000
                 ][
                   , .(media_valor         = mean(valor),
                       desvio_padrao_valor = sd(valor)
                       )
                 ]

all(dados1A == dados1B)
```

```
## [1] TRUE
```


Perceba que, criamos um novo objeto com o resultado. Porém, muitas vezes, se faz necessário adicionar essa informação como uma nova coluna, mesmo que ela seja redundante. Para isso, utilizaremos um outra função para escrever as expressões, a função `:=`. Veja.


```r
dados[, media_valor := mean(valor)]
dados[, desvio_padrao_valor := sd(valor)]
dados
```

```
##           categoria     valor sexo media_valor desvio_padrao_valor
##        1:         1 1041.2756    1    1000.004            20.00367
##        2:         1  989.1302    1    1000.004            20.00367
##        3:         1  978.7430    1    1000.004            20.00367
##        4:         1  998.9086    1    1000.004            20.00367
##        5:         1 1012.8971    1    1000.004            20.00367
##       ---                                                         
##  9999996:         4 1018.1498    2    1000.004            20.00367
##  9999997:         4  981.9318    2    1000.004            20.00367
##  9999998:         4 1000.6837    2    1000.004            20.00367
##  9999999:         4  987.9402    2    1000.004            20.00367
## 10000000:         4 1038.9378    2    1000.004            20.00367
```

A função `:=` também é utilizada para deleter colunas.


```r
dados[, c("media_valor", "desvio_padrao_valor") := NULL]
```

Para utilizarmos uma mesma chamada para criar várias colunas novas, devemos utilizar a seguinte forma.


```r
dados[, `:=`(media_valor         = mean(valor),
             desvio_padrao_valor = sd(valor)
             )
      ]
head(dados)
```

```
##    categoria     valor sexo media_valor desvio_padrao_valor
## 1:         1 1041.2756    1    1000.004            20.00367
## 2:         1  989.1302    1    1000.004            20.00367
## 3:         1  978.7430    1    1000.004            20.00367
## 4:         1  998.9086    1    1000.004            20.00367
## 5:         1 1012.8971    1    1000.004            20.00367
## 6:         1 1010.0885    1    1000.004            20.00367
```

A função `:=` altera o `data.table` inplace, então tome cuidado quando for alterar colunas já existentes. A exemplo, vamos alterar uma coluna já existente utilizando de filtros.


```r
dados[valor > 1000,
      `:=`(media_valor         = mean(valor),
           desvio_padrao_valor = sd(valor)
           )
      ]
head(dados)
```

```
##    categoria     valor sexo media_valor desvio_padrao_valor
## 1:         1 1041.2756    1    1015.969            12.05818
## 2:         1  989.1302    1    1000.004            20.00367
## 3:         1  978.7430    1    1000.004            20.00367
## 4:         1  998.9086    1    1000.004            20.00367
## 5:         1 1012.8971    1    1015.969            12.05818
## 6:         1 1010.0885    1    1015.969            12.05818
```

Observou que alguns dos dados da coluna alterada aparentam serem os mesmos? A coluna só foi alterada onde a condição do filtro é satisfeita, que é `valor > 1000`. Veja:


```r
setorder(dados, valor)
dados[c(1:5, (nrow(dados)-5):nrow(dados))]
```

```
##     categoria     valor sexo media_valor desvio_padrao_valor
##  1:         2  900.4804    2    1000.004            20.00367
##  2:         3  902.8411    1    1000.004            20.00367
##  3:         1  903.1211    1    1000.004            20.00367
##  4:         2  903.4796    1    1000.004            20.00367
##  5:         4  903.6676    1    1000.004            20.00367
##  6:         4 1100.1116    1    1015.969            12.05818
##  7:         4 1100.5020    1    1015.969            12.05818
##  8:         2 1101.6412    1    1015.969            12.05818
##  9:         4 1102.5581    1    1015.969            12.05818
## 10:         3 1103.1713    2    1015.969            12.05818
## 11:         3 1105.0339    1    1015.969            12.05818
```

Vamos criar agora uma label para a variável `sexo`. Veremos o que acontecerá.


```r
setkey(dados, sexo)
dados[.(sexo = 1), SexoBeneficiario := "Masculino"]
dados[c(1,500,10000000)]
```

```
##    categoria     valor sexo media_valor desvio_padrao_valor
## 1:         3  902.8411    1    1000.004            20.00367
## 2:         3  926.0478    1    1000.004            20.00367
## 3:         3 1103.1713    2    1015.969            12.05818
##    SexoBeneficiario
## 1:        Masculino
## 2:        Masculino
## 3:             <NA>
```




Eu solicitei a impressão das linhas 1, 500 e 10000000. Veja a variável que criamos, observe que para onde as condições do filtro não foram satisfeitas não foi aplicado nenhum valor para a nova variável `SexoBeneficiario`.


```r
dados[.(sexo = 2), SexoBeneficiario := "Feminino"]
dados[c(1,500,10000000)]
```

```
##    categoria     valor sexo SexoBeneficiario
## 1:         3  902.8411    1        Masculino
## 2:         3  926.0478    1        Masculino
## 3:         3 1103.1713    2         Feminino
```

Pronto, podemos fazer isso de outra forma, veremos qual será mais rápido, para isso iremos deletar esta coluna.


```r
dados[, SexoBeneficiario := NULL]
```


```r
options(datatable.verbose=F, datatable.auto.index = F)
rbenchmark::benchmark(
  "sem_key_ifelse" = {
    dados[, SexoBeneficiario1 := ifelse(sexo == 1, "Masculino", "Feminino")]
    },
  "sem_key_data.table"= {
    dados[sexo == 1, SexoBeneficiario2 := "Masculino"]
    dados[sexo == 2, SexoBeneficiario2 := "Feminino"]
  },
  "com_key" ={
    dados[.(sexo = 1), SexoBeneficiario3 := "Masculino"]
    dados[.(sexo = 2), SexoBeneficiario3 := "Feminino"]

    },
  order = "relative",
  replications = 1,
  columns = c("test", "replications", "relative", "elapsed")
  )
```

```
##                 test replications relative elapsed
## 2 sem_key_data.table            1    1.000   0.209
## 3            com_key            1    1.033   0.216
## 1     sem_key_ifelse            1    8.321   1.739
```

São iguais?


```r
all(dados$SexoBeneficiario2 == dados$SexoBeneficiario3)
```

```
## [1] TRUE
```


Agora vamos ativar a função de auto index do `data.table`. O que ela faz? Basicamente observar quais filtros irá fazer e cria um índice por trás dos panos. Porém ele não reordena o objeto em si, é criado uma coluna com esse índice nos atributos do objeto. Ainda é mais lento do que criar uma Chave, mas já ajuda bastante, veja:


```r
options(datatable.verbose=F, datatable.auto.index = T)

rbenchmark::benchmark(
  "sem_key_ifelse" = {
    dados[, SexoBeneficiario1 := ifelse(sexo == 1L, "Masculino", "Feminino")]
    },
  "sem_key_data.table"= {
    dados[sexo == 1L, SexoBeneficiario2 := "Masculino"]
    dados[sexo == 2L, SexoBeneficiario2 := "Feminino"]
  },
  "com_key" ={
    dados[.(sexo = 1L), SexoBeneficiario3 := "Masculino"]
    dados[.(sexo = 2L), SexoBeneficiario3 := "Feminino"]

    },
  order = "relative",
  replications = 1,
  columns = c("test", "replications", "relative", "elapsed")
  )
```

```
##                 test replications relative elapsed
## 3            com_key            1    1.000   0.165
## 2 sem_key_data.table            1    1.073   0.177
## 1     sem_key_ifelse            1    9.945   1.641
```

O index apenas funciona para `==` e `%in%`. Uma coisa que devemos ter cuidado é sempre utilizar os tipos certos para se fazer comparação e para atribuir valores, conversões saem caras em questão de performance, assim sempre que possível devemos atribuir as coisas de forma correta, como em `sexo == 1L` aqui eu especifico com o `L` que o número à esquerda é um inteiro.

***
## Agrupamento
***

Nós já fizemos agregações na sessão anterior. Iremos fazer a exata mesma coisa, porém agora podemos agrupar os cálculos em grupos (basicamente o `GROUP BY` de qualquer SQL).

Vamos utilizar o mesmo mesmo exemplo anterior, só que agora agrupando por `categoria`

Limpe seus dados (caso ainda o tenha) e libere a memória.

```r
rm(dados)
gc(reset = T)
```

```
##           used (Mb) gc trigger  (Mb) max used (Mb)
## Ncells  599594 32.1    1069260  57.2   599594 32.1
## Vcells 3693424 28.2   90661241 691.7  3693424 28.2
```

Vamos declarar o dados novamente


```r
dados <- data.table(categoria = sample.int(4, 10000000, replace = T),
                    valor     = rnorm(10000000, mean = 1000, sd = 20),
                    sexo      = sample.int(2, 10000000, replace = T)
                    )
```


Iremos calcular a média e desvio padrão para cada nível da variável `categoria`


```r
dados[, .(media_valor         = mean(valor),
          desvio_padrao_valor = sd(valor)
          ),
      by = .(categoria)
      ]
```

```
##    categoria media_valor desvio_padrao_valor
## 1:         4    999.9855            19.99456
## 2:         2    999.9867            20.00855
## 3:         1   1000.0115            20.00871
## 4:         3    999.9952            19.99853
```

Esta foi uma agregação, um resumo, vamos criar as mesmas duas métricas para cada categoria, um mutate.


```r
dados[, `:=`(media_valor = mean(valor),
             desvio_padrao_valor = sd(valor)
             ),
      by = .(categoria)
      ]

head(dados)
```

```
##    categoria     valor sexo media_valor desvio_padrao_valor
## 1:         4 1006.5045    2    999.9855            19.99456
## 2:         4  999.9418    2    999.9855            19.99456
## 3:         2  984.4775    2    999.9867            20.00855
## 4:         1 1030.6272    2   1000.0115            20.00871
## 5:         1  995.2280    1   1000.0115            20.00871
## 6:         2 1000.1660    1    999.9867            20.00855
```


Também podemos fazer agrupamentos por mais de uma variável. Aqui será feito com `categoria` e `sexo`.


```r
dados[, .(media_valor         = mean(valor),
          desvio_padrao_valor = sd(valor)
          ),
      by = .(categoria, sexo)
      ]
```

```
##    categoria sexo media_valor desvio_padrao_valor
## 1:         4    2    999.9882            19.98116
## 2:         2    2    999.9859            20.01013
## 3:         1    2   1000.0132            20.00674
## 4:         1    1   1000.0099            20.01069
## 5:         2    1    999.9875            20.00697
## 6:         4    1    999.9828            20.00795
## 7:         3    1    999.9711            19.98177
## 8:         3    2   1000.0192            20.01526
```


***
## JOINss
***

O `data.table`tem seu modo de realizar JOINS e JOINS inplace. É eficiente, mas pode confundir um pouco.

Primeiro iremos criar as tabelas, `A` será um índice, contendo 1 e 2. `B` será uma índice com uma label.


```r
A <- data.table(id_A = sample.int(2, size = 10000,replace = T))
A[id_A == 1, label_A := "A"]
A[id_A == 2, label_A := "B"]

B <- data.table(id_B    = 1:2,
                label_B = c("Masculino", "Feminino")
                )

head(A)
```

```
##    id_A label_A
## 1:    2       B
## 2:    1       A
## 3:    2       B
## 4:    1       A
## 5:    2       B
## 6:    1       A
```

```r
B
```

```
##    id_B   label_B
## 1:    1 Masculino
## 2:    2  Feminino
```

Os JOINS podem ser definidos pela seguinte forma

### LEFT JOIN


```r
### LEFT JOIN EM A com B
B[A, on =c("id_B" = "id_A")] # LEFT JOIN COM COPIA
```

```
##        id_B   label_B label_A
##     1:    2  Feminino       B
##     2:    1 Masculino       A
##     3:    2  Feminino       B
##     4:    1 Masculino       A
##     5:    2  Feminino       B
##    ---                       
##  9996:    1 Masculino       A
##  9997:    1 Masculino       A
##  9998:    1 Masculino       A
##  9999:    1 Masculino       A
## 10000:    2  Feminino       B
```

```r
### LEFT JOIN EM A com B
A[B, label_B := label_B, on =c("id_A" = "id_B")] # LEFT JOIN SEM COPIA
A
```

```
##        id_A label_A   label_B
##     1:    2       B  Feminino
##     2:    1       A Masculino
##     3:    2       B  Feminino
##     4:    1       A Masculino
##     5:    2       B  Feminino
##    ---                       
##  9996:    1       A Masculino
##  9997:    1       A Masculino
##  9998:    1       A Masculino
##  9999:    1       A Masculino
## 10000:    2       B  Feminino
```

```r
### LEFT JOIN EM B com A
B[A, label_A := label_A, on =c("id_B" = "id_A")] # LEFT JOIN SEM COPIA
B
```

```
##    id_B   label_B label_A
## 1:    1 Masculino       A
## 2:    2  Feminino       B
```

Para colocar várias colunas da tabela à direita para a direita num **LEFT JOIN SEM COPIA** é necessário um pouco mais de trabalho. Veja, é só um pouco mais de trabalho.

Primeiro, crio esta função auxiliar, ela irá construir o código responsável pelo `lhs := rhs` de várias colunas e retorno como uma expressão do R. Após isso eu avalio essa expressão dentro do `data.table`


```r
pull_right <- function(columns,exceptions){
  columns <- columns[!columns %in% exceptions]
  string <- paste( "`:=`(",paste(columns, columns, sep = "=", collapse = ","), ")")
 
  return(parse(text =string))
  }

pull_right(columns = c("label_A", "label_B"), exceptions = c("id"))
```

```
## expression(`:=`(label_A = label_A, label_B = label_B))
```

Vejamos o exemplo anterior, utilizando desta função:

```r
# DECLARANDO AS MATRIZES NOVAMENTE
A <- data.table(id_A = sample.int(2, size = 10000,replace = T))
A[id_A == 1, label_A := "A"]
A[id_A == 2, label_A := "B"]

B <- data.table(id_B    = 1:2,
                label_B = c("Masculino", "Feminino")
                )
#
#
#
# LEFT JOIN IN PLACE
A[B,
  eval(pull_right(columns = "label_B",
                  exceptions = "id_B")
       ),
  on = .(id_A == id_B)
  ]
A
```

```
##        id_A label_A   label_B
##     1:    2       B  Feminino
##     2:    1       A Masculino
##     3:    2       B  Feminino
##     4:    2       B  Feminino
##     5:    1       A Masculino
##    ---                       
##  9996:    1       A Masculino
##  9997:    1       A Masculino
##  9998:    2       B  Feminino
##  9999:    1       A Masculino
## 10000:    2       B  Feminino
```

### RIGHT JOIN

Reimagine um RIGHT JOIN como um LEFT JOIN e faz como no passo anterior

### INNER JOIN

Vamos criar datasets de exemplo.


```r
A <- data.table(id_A = 1:10,
                label_A = "alguma coisa"
                )
B <- data.table(id_B = 1:4,
                label_B = "MATCH")
```

Para o INNER JOIN FAZEMOS


```r
A[B, on = c("id_A" = "id_B"), nomatch = 0]
```

```
##    id_A      label_A label_B
## 1:    1 alguma coisa   MATCH
## 2:    2 alguma coisa   MATCH
## 3:    3 alguma coisa   MATCH
## 4:    4 alguma coisa   MATCH
```

### NOT JOIN



Vamos criar datasets de exemplo.


```r
A <- data.table(id_A = 1:10,
                label_A = "alguma coisa"
                )
B <- data.table(id_B = 1:4,
                label_B = "MATCH")
```

Para o NOT JOIN temos


```r
A[!B, on = c("id_A" = "id_B")]
```

```
##    id_A      label_A
## 1:    5 alguma coisa
## 2:    6 alguma coisa
## 3:    7 alguma coisa
## 4:    8 alguma coisa
## 5:    9 alguma coisa
## 6:   10 alguma coisa
```

### NON EQUI JOIN

Este JOIN serve para quando queremos fazer um match baseado em desigualdade. Vamos a criação do dataset e ao exemplo.



```r
A <- data.table(id_A = 1:20,
                label_A = "alguma coisa"
                )
B <- data.table(id_B = 15:25,
                label_B = "MATCH")
```

Vejamos A e B


```r
A
```

```
##     id_A      label_A
##  1:    1 alguma coisa
##  2:    2 alguma coisa
##  3:    3 alguma coisa
##  4:    4 alguma coisa
##  5:    5 alguma coisa
##  6:    6 alguma coisa
##  7:    7 alguma coisa
##  8:    8 alguma coisa
##  9:    9 alguma coisa
## 10:   10 alguma coisa
## 11:   11 alguma coisa
## 12:   12 alguma coisa
## 13:   13 alguma coisa
## 14:   14 alguma coisa
## 15:   15 alguma coisa
## 16:   16 alguma coisa
## 17:   17 alguma coisa
## 18:   18 alguma coisa
## 19:   19 alguma coisa
## 20:   20 alguma coisa
```

```r
B
```

```
##     id_B label_B
##  1:   15   MATCH
##  2:   16   MATCH
##  3:   17   MATCH
##  4:   18   MATCH
##  5:   19   MATCH
##  6:   20   MATCH
##  7:   21   MATCH
##  8:   22   MATCH
##  9:   23   MATCH
## 10:   24   MATCH
## 11:   25   MATCH
```

Vamos fazer o match onde id_B é menor que id_A


```r
B[A, on = .(id_B < id_A)]
```

```
##     id_B label_B      label_A
##  1:    1    <NA> alguma coisa
##  2:    2    <NA> alguma coisa
##  3:    3    <NA> alguma coisa
##  4:    4    <NA> alguma coisa
##  5:    5    <NA> alguma coisa
##  6:    6    <NA> alguma coisa
##  7:    7    <NA> alguma coisa
##  8:    8    <NA> alguma coisa
##  9:    9    <NA> alguma coisa
## 10:   10    <NA> alguma coisa
## 11:   11    <NA> alguma coisa
## 12:   12    <NA> alguma coisa
## 13:   13    <NA> alguma coisa
## 14:   14    <NA> alguma coisa
## 15:   15    <NA> alguma coisa
## 16:   16   MATCH alguma coisa
## 17:   17   MATCH alguma coisa
## 18:   17   MATCH alguma coisa
## 19:   18   MATCH alguma coisa
## 20:   18   MATCH alguma coisa
## 21:   18   MATCH alguma coisa
## 22:   19   MATCH alguma coisa
## 23:   19   MATCH alguma coisa
## 24:   19   MATCH alguma coisa
## 25:   19   MATCH alguma coisa
## 26:   20   MATCH alguma coisa
## 27:   20   MATCH alguma coisa
## 28:   20   MATCH alguma coisa
## 29:   20   MATCH alguma coisa
## 30:   20   MATCH alguma coisa
##     id_B label_B      label_A
```

Esse é um JOIN bem confuso e é muito fácil fazer coisa errada. São raros os casos em que ele é necessário. Normalmente preenchimento de alguma coisa relacionada a uma linha temporal. Eu sugiro que tente usar ao máximo outras formas de fazer o que queira antes de tentar o NON EQUI JOINS.
