---
title: "Análise  - Inteligência Computacional -  COC 361"
author: "Lucas Rolim e Anderson Barbosa"
date: "21 de outubro de 2016"
output: html_document
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE, message = FALSE)
```

## Descrição do problema

**Problema: É possível prever a nota IMDB de um filme antes mesmo de ele ser lançado?**

Essa será a principal pergunta que irá guiar o desenvolvimento de uma análise de mais de 5000 filmes presentes no site do [IMDB](www.imdb.com). Utilizando técnicas de aprendizado de máquina e estatística nós iremos não só tentar criar um **modelo de aproximação de funções** que seja capaz de responder à pergunta anterior de maneira satistatória, como também iremos criar visualizações que nos permitam entender as relações entre essas variáveis.

Esse problema é importante não só para ajudar espectadores em geral a reduzir sua taxa de decepção ao ir ao cinema, mas também se caracteriza como de suma importância para a indústria cinematrográfica em si. Filmes, antes de tudo, são investimentos. Conseguir prever se um filme será um fiasco ou um sucesso de bilheteria pode poupar milhões as produtoras de filmes e investidores.

*Para os que desconhecem, o IMDB.com é a principal referência mundial na avaliação da qualidade de filmes. O site conta com dezenas de milhares de filmes e também com avaliações e críticas produzidas por centenas de milhares de usuários e críticos profissionais*

## Pesquisa Bibliográfica

Os dados utilizados nessa análise foram minerados por Chuan Sun no site do IMDB e depois disponibilizados na plataforma Kaggle. Tanto o tópico desse conjunto de dados quanto uma explicação em detalhes sobre o processo de mineração podem ser encontrados no seguinte link:

- [IMDB 5000 Movie Dataset](https://www.kaggle.com/deepmatrix/imdb-5000-movie-dataset)

## Descrição dos dados

O conjunto de dados possui informação de **5043 filmes**, lançados ao longo dos últimos 100 anos em 66 países. Ainda, conta com 2399 nomes de diretores e centenas de nomes de atores atrizes.

**As 28 variáveis disponíveis são:**

| Variável                     	| Significado                                                                     	|
|------------------------------	|---------------------------------------------------------------------------------	|
| color                        	| ?                                                                               	|
| num_voted_users              	| Número de usuários que deram uma classificação ao filme no site do IMDB         	|
| cast_total_facebook_likes    	| Número total de curtidas de toda página do elenco no Facebook                   	|
| facenumber_in_poster         	| Número de faces humanas que aparecem  no poster do filme                        	|
| plot_keywords                	| Palavras-chave da trama                                                         	|
| movie_imdb_link              	| Endereço da página do filme no IMDB                                             	|
| num_user_for_reviews         	| Número de críticas de usuários do IMDB                                          	|
| language                     	| Idioma do filme                                                                 	|
| country                      	| País em que o filme foi produzido                                               	|
| content_rating               	| Avaliação do conteúdo                                                           	|
| budget                       	| Orçamento do filme                                                              	|
| title_year                   	| Ano de lançamento do filme                                                      	|
| imdb_score                   	| Nota do filme no IMDB                                                           	|
| aspect_ratio                 	| ?                                                                               	|
| movie_facebook_likes         	| Número de curtidas da página do filme no Facebook                               	|
| director_name                	| Nome do diretor do filme                                                        	|
| num_critic_for_reviews       	| Número de críticas externas ao IMDB                                             	|
| actor_(1,2,3)_name           	| Nomes dos três principais atores do filme                                       	|
| movie_title                  	| Título do filme                                                                 	|
| actor_(1,2,3)_facebook_likes 	| Números de curtidas das páginas dos três principais atores do filme no Facebook 	|
| director_facebook_likes      	| Número de curtidas da página do diretor no Facebook                             	|
| genres                       	| Gênero do filme                                                                 	|
| gross                        	| lucro bruto de bilheteria obtido pelo filme                                     	|
| duration                     	| Duração do filme em minutos                                                     	|


## Ferramentas utilizadas

A principal ferramenta utilizada nessa análise foi a linguagem R e o ambiente RStudio. Ademais, as seguintes bibliotecas foram utilizadas.

- rMarkdown
- ggplot2
- caTools
- randomForest
- dplyr
- rpart
- rpart.plot
- corrplot
- neuralnet
- caTools


```{r bibliotecas}
library(dplyr)
library(corrplot)
library(ggplot2)
library(caTools)
library(rpart)
library(rpart.plot)
library(randomForest)
library(neuralnet)
library(caTools)
library(caret)
```

O formato do conjunto de cados que necessitamos ler é *.csv*, o qual já possui suporte nativo do próprio R. O formato desse relatório, por sua vez, é *.rmd*, um formato que possibilita uma explicação mais bem elaborada do processo executado.

Por fim, a linguagem R foi escolhida por ser a que possui bibliotecas no estado da arte no que tange a tratamentos estatísticos e criação de modelos de regressão, classificação e agrupamentos. Essa linguagem é uma "sucessora" da linguagem S e conta com a ampla adoção acadêmica e diversas bibliotecas otimizadas.



## Pré-processamento

O pré-processamento de dados é um passo importante no processo de mineração de dados. Os métodos de coleta de dados, em geral, não são controlados rigorosamente, o que resulta em um conjunto de dados com valores fora do intervalo, combinações impossíveis de dados, valores ausentes etc. Desta forma, o pré-processamento é necessário para evitar que a análise dos dados produza resultados enganosos. Ela envolve limpeza de dados, normalização, transformação de variáveis, entre outros. O resultado final do pré-processamento de dados é o conjunto de treinamento final, que será utilizado para o ajuste do modelo.

### Leitura dos Dados

Em primeiro lugar, vamos dar uma primeira olhada no formato do nosos conjunto de dados. Vamos verificar quais são as variáveis e seus tipos.

```{r leitura}
imdb_data = read.csv("movie_metadata.csv",stringsAsFactors = FALSE)
str(imdb_data)
```
### Normalização dos Dados e escolha de variáveis

Algumas das variáveis de texto do conjunto de dados serão convertidas em variáveis numéricas através de um critério de normalização definido pelas seguintes regras:

- Se o filme foi produzido no EUA a variável **country** será 1, caso contrário será 0;
- Se o filme foi produzido em inglês a variável **language** será 1, caso contrário será 0.
- Criação da variável **directorIMDB**, que terá o número de filmes já feito pelo diretor.
- Criação da variável **directorMovies**, que terá o número de filmes já feito pelo diretor do filme.

Essas regras de normalização foram definidas pensando no fato da indústria cinematográfica americana ser a mais bem desenvolvida e com maiores orçamentos do mundo, além de produzir os maiores sucessos das últimas décadas. Logo, espera-se que filmes americanos tenham uma probabilidade maior de pontuar melhor na escala IMDB de maneira geral.

Por fim, passaremos a considerar apenas os valores numéricos do modelo e também desconsideraremos as entradas com valores ausentes.

*A criação das variáveis directorIMDB e directorMovies será feita após o particionamento do conjunto de dados. Esse cuidado é necessário para que não sejam levados em conta no cálculo da média do IMDB do diretor os filmes que estão no conjunto de testes*

```{r normalization }

# Essa versão do directorMovies será usada na Análise Exploratória
directorMovies <- summarise(group_by(imdb_data,director_name),mean=mean(imdb_score),number = n())
directorMovies <- directorMovies[2:nrow(directorMovies),]

imdb_data$language[imdb_data$language !="English"] = 0
imdb_data$language[imdb_data$language =="English"] = 1
imdb_data$language <- as.numeric(imdb_data$language)
imdb_data$country[imdb_data$country != "USA"] = 0
imdb_data$country[imdb_data$country == "USA"] = 1
imdb_data$country <- as.numeric(imdb_data$country)

temp <- imdb_data$director_name

imdb_data <- imdb_data[sapply(imdb_data,is.numeric)]
NoTNAIndex = complete.cases(imdb_data)
imdb_data <- imdb_data[which(NoTNAIndex),]
temp= temp[which((NoTNAIndex))]

```

Nesse ponto é importante notar que, com a **exclusão de filmes que possuiam variáveis ausentes**, nosso conjunto de dados foi reduzido para um universo de 3801 amostras. Ou seja, **2059 registros possuiam algum valor ausente**, um total de 40% dos dados. Essa redução foi considerável, mas se faz necessária e o volume de dados restante aparenta ser grande o suficiente para não comprometer a análise. 

Por fim, após a limpezadas variáveis para apenas variáveis numéricas, restaram 18 variáveis. Lembrando que no momento de aplicação dos modelos serão adiconadas mais duas, *DirectorMovie e DirecotrIMDB*, somando um total de 20 variáveis a serem trabalhadas.


## Análise Exporatória


## Estatísticas básicas:

```{r estatisticaBasica}
summary(imdb_data)
```


**Legenda:**

| Métrica  | Significado                                                       |
|----------|-------------------------------------------------------------------|
| Min.:    | Valor mínimo registrado da variável                               |
| 1st Qu.  | Percentil 25, valor abaixo do qual se encontram 25% dos registros |
| Median:  | Percentil 50, ou mediana                                          |
| 3rd Qu.: | Percentil 75                                                      |
| Max.:    | Valor máximo registrado da variável                               |


### Matriz de Correlação

``` {r matriz de correlação}
corrplot(cor(imdb_data))
```



Analisando esta matriz de correlação, é possível ver que as seguintes variáveis apresentam forte correlação positiva: **cast_total_facebook_likes** e **actor_1_facebook_likes**, o que já era esperado, pois o conjunto de curtidas de um ator é um subconjunto do conjunto de curtidas totais do elenco do filme. **movie_facebook_likes** e **num_critic_for_reviews**, indicando que o número de críticas externas que o filme recebe contribui para o seu número de curtidas no facebook **num_voted_users e num_user_for_reviews**, indicando que quanto mais usuários dão uma nota ao filme, mais usuários escrevem um crítica para ele; **Cast_total_facebook_likes** e **actor_2_facebook_likes**, que, assim como a primeira correlação, já era esperada; e **gross** e **num_voted_users**, indicando que filmes que geram uma receita maior tendem a receber mais votos de usuários do IMDB. Outra coisa interessante a ser observada, é que o orçamento (**budget**) do filme não apresenta correlação com nenhuma variável, indicando que a quantidade de dinheiro investida no filme não influencia em como o filme será recebido pelo público. 

### Histogramas

**Notas IMDB dos filmes**

```{r HistogramaIMDB}
ggplot(imdb_data,aes(x=imdb_score)) + geom_histogram() + xlab("Nota no IMDB") + ylab("Número de Filmes")
```


A partir do histograma acima, é possível observar que a variável imdb_score segue uma distribuição normal com média aproximadamente igual 6,5. O valor que se esperava para essa média seria algo em torno de 5,0, uma vez que as notas variam de 0 a 10. Uma possível explicação para a média ter sido diferente da esperada é que o número de registros não é grande o suficiente ou que muito poucos filmes tendem a ser classificados com a nota menor que 2.


**Outros histogramas**

```{r Histogramas, echo=FALSE}

## num_critic_for_reviews
nomes=c('num_critic_for_reviews', 'duration', 'director_facebook_likes', 'actor_3_facebook_likes', 'actor_1_facebook_likes', 'gross', 'num_voted_users', 'cast_total_facebook_likes', 'facenumber_in_poster', 'num_user_for_reviews', 'language', 'country', 'budget', 'title_year', 'actor_2_facebook_likes', 'imdb_score', 'aspect_ratio', 'movie_facebook_likes')
par(mfrow=c(3, 3))
for (i in 1:9) {
    hist(imdb_data[,i],  col="blue", border="white", main="", ylab="frequência", xlab=nomes[i])
}
for (i in 10:18) {
  
    if (i!=16){
       hist(imdb_data[,i],  col="blue", border="white", main="", ylab="frequência", xlab=nomes[i])
    }
}
```

**Transformação de Variáveis**

Sabemos que nosos modelo de Regressão funciona de maneira mais precisa quando temos variáveis que seguem uma Distriguição Normal. Dessa forma, utilizaremos os histograma para analisar algumas variáveis que podem ser transformadas de modo a aproximar melhor esse tipo de distribuição:

```{r HistogramasTransformação, echo=FALSE}

par(mfrow=c(3, 3))

## num_critic_for_reviews
par(mfrow=c(3, 3))
for (i in 1:9) {
    hist(log(imdb_data[,i]),  col="blue", border="white", main="", ylab="frequência", xlab=nomes[i])
}
for (i in 10:18) {
    hist(log(imdb_data[,i]),  col="blue", border="white", main="", ylab="frequência", xlab=nomes[i])
}

```

Ao analisar essa variáveis percebemos que de fato a distribuição de muitas delas podem ser aproximadas por uma normal se aplicarmos o logaritmo. O problema que encontramos nesse caso é o fato de muitos valores *"infinito"* serem gerados. Logo, no momento, decidimos usar a análise com log apenas para os outliers, e não para os modelos.

### Outliers

Outliers, ou valores aberrantes, são registros que apresentam uma distância muito grande dos demais. A importância da detecção e remoção destes reside no fato de que a presença deles pode levar a resultados enganosos na análise de dados.

**Boxplots das variáveis**

```{r boxplot}
boxplot(imdb_data, names=c('1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18'))
```

Para observar melhor os outliers das variáveis, plotamos os boxplots dos logaritmos das variáveis. 

```{r boxplotlog, warning = FALSE}
boxplot(log(imdb_data), names=c('1', '2', '3', '4', '5', '6', '7', '8', '9', '10', '11', '12', '13', '14', '15', '16', '17', '18'))
```


### Relação entre a nota IMDB e a duração do filme


```{r IMDBvsDuração}

ggplot(imdb_data,aes(x=duration, y=imdb_score)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Nota no IMDB") + xlab("Duração do filme (minutos)")
```


Este scatterplot mostra que a duração do filme e sua nota no IMDB apresentam uma fraca correlação positiva.

### Relação entre a nota IMDB e o lucro

```{r IMDBvsLucro}
ggplot(imdb_data,aes(x=gross, y=imdb_score)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Nota no IMDB") + xlab("Lucro do Filme")
```

Este scatterplot, assim como o anterior, mostra que o lucro obtido pelo filme e sua nota no IMDB apresentam uma fraca correlação positiva.

### Relação entre a nota IMDB e orçamento

**Com outliers**

```{r IMDBvsOrçamento}
ggplot(imdb_data,aes(x=budget, y=imdb_score)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Nota no IMDB") + xlab("Orçamento do Filme")
```

No scatterplot acima, observamos que os registros não estão distribuidos de maneira comportada, o que se deve à presença de outliers. Por isso, removeremos os outliers relacionados a variável budget (orçamento) e plotar novamente o gráfico.

**Sem outliers**

```{r IMDBvsOrçamento2, warning = FALSE}
outlierKD <- function(dt, var) {
     var_name <- eval(substitute(var),eval(dt))
     na1 <- sum(is.na(var_name))
     m1 <- mean(var_name, na.rm = T)
     par(mfrow=c(2, 2), oma=c(0,0,3,0))
     boxplot(var_name, main="Com outliers")
     hist(var_name, main="Com outliers", xlab=NA, ylab=NA)
     outlier <- boxplot.stats(var_name)$out
     mo <- mean(outlier)
     var_name <- ifelse(var_name %in% outlier, NA, var_name)
     boxplot(var_name, main="Sem outliers")
     hist(var_name, main="Sem outliers", xlab=NA, ylab=NA)
     title("Verificação de outliers", outer=TRUE)
     na2 <- sum(is.na(var_name))
     cat("Outliers identificados:", na2 - na1, "\n")
     cat("Proporção (%) de outliers:", round((na2 - na1) / sum(!is.na(var_name))*100, 1), "\n")
     cat("Média dos outliers:", round(mo, 2), "\n")
     m2 <- mean(var_name, na.rm = T)
     cat("Média sem remover outliers:", round(m1, 2), "\n")
     cat("Média se removermos outliers:", round(m2, 2), "\n")
     dt[as.character(substitute(var))] <- invisible(var_name)
     assign(as.character(as.list(match.call())$dt), dt, envir = .GlobalEnv)
     return(invisible(dt))
}
outlierKD(imdb_data, budget)
ggplot(imdb_data,aes(x=budget, y=imdb_score)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Nota no IMDB") + xlab("Orçamento do Filme")
```

Este novo gráfico confirma o que foi observado na matriz de correlação acima, que a nota no IMDB de um filme e seu orçamento estão descorrelacionados.

### Relação entre a nota IMDB e o Diretor

```{r IMDBvsDiretor}
ggplot(directorMovies,aes(x=number, y=mean)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Nota no IMDB") + xlab("Número de filmes feitos")
```

Este scatterplot mostra que o diretor do filme e sua nota no IMDB apresentam forte correlação positiva.

### Relação entre o lucro e o orçamento

```{r lucroVsOrçamento}
ggplot(imdb_data,aes(x=budget, y=gross)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Lucro") + xlab("Orçamento")
```

Desse gráfico não podemos inferir nenhuma conclusão clara sobre a relação entre lucro e orçamento de um filme. Os pontos fora da reta são muitos para serem considerados outliers.



## Conclusões gerais da Análise Exploratória

Ao analisar nosso conjunto de dados um primeiro aspecto que podemos notar é a grande assimetria das variáveis. A maior parte delas possui uma distribuição assimétrica e só pode de alguma forma ser comparada com uma distribuição normal quando aplicamos a função logaritmo. Essa é uma característica bem interessante e que pode ser um indicador que um processo de *feature scaling* se faz necessário para a aplicação dos modelosde regressão.

Outra característica que deve ser levada em conta é a presença de um grande número de outliers em algumas variáveis, como o orçamento. Além de deixar os modelos de certa forma mais sensível, pode se fazer necessário retirar esses outliers para tornar futuras conclusões válidas.

Por fim, um outro aspecto interessante que merece ser comentado é o fato de não existir uma correlação forte entre quase nenhuma das variáveis. A falta de correlação entre orçamento e lucro, como tivemos a oportunidade de verificar tanto na matriz de correlação quanto em um scatterplot, foi a mais intrigante.



### Ajuste de Escala

Vamos converter todos os nossos dados para uma escala de variação aproximada entre -1 e 1, de modo a melhorar nossos resultados e muitas vezes a velocidade de conversão do método.

```{r scale}
maxs <- apply(imdb_data, 2, max) 
mins <- apply(imdb_data, 2, min)
imdb_data<- as.data.frame(scale(imdb_data, center = mins, scale = maxs - mins))
```


### Particionamento dos conjunto de dados

Vamos utilizar a técnica de validação cruzada de 10 ciclos. Para isso criaremos um vetor de índices folds, que nos auxiliará na separação dos grupos de validação durnate as diferentes iterações.

```{r splitFunctions}

imdb_data$director_name <- temp

set.seed(161095)

# Trecho de criação de novas variáveis para o diretor

directorMean <-function(data){
  data = directorHistory [which(directorHistory ==data),2]
  data
}

directorMNumber <-function(data){
  data = directorHistory [which(directorHistory ==data),3]
  data
}

DMeanNorm <- function(data){
  directorIMDB =  sapply(data$director_name,directorMean)
  directorMovies = sapply(data$director_name,directorMNumber)
  data$directorIMDB <- as.numeric(directorIMDB)
  data$directorMovies <- as.numeric(directorMovies)
  my_data <- subset(data, select=-c(director_name))
  my_data <- my_data [sapply(my_data ,is.numeric)]
  my_data$directorIMDB = (my_data$directorIMDB -mean(my_data$directorIMDB))/(max(my_data$directorIMDB)-min(my_data$directorIMDB))
  my_data$directorMovies = (my_data$directorMovies - mean(my_data$directorMovies))/(max(my_data$directorMovies)- min(my_data$directorMovies))
  my_data  <- na.exclude(my_data )
  my_data
}

maxs <- apply(imdb_data, 2, max) 
mins <- apply(imdb_data, 2, min)
imdb_data<- as.data.frame(scale(imdb_data, center = mins, scale = maxs - mins))

```

### Definição das Variáveis

Nesse trecho iremos definir de maneira genérica as variáveis que serão utilizadas daqui em diante. Esse processo poupa o trabalho de reescrever código depois e também facilita a manutenção do mesmo.

```{r variáveis}
n = colnames(imdb_data)[-c(which(colnames(imdb_data) == "director_name"))]
n = c(n,"directorMovies","directorIMDB")
goal_variable = "imdb_score"
dependent_variable = paste(goal_variable," ~ ")
independent_variables = paste(n[n != goal_variable],collapse = "+")
model_variables = paste(dependent_variable,independent_variables)
```

## Metodologia

A estratégia adotada será partir do modelo mais simples e rápido para o modelo mais complexo e robusto. Visto isso, em nossa análise testaremos os cinco seguintes modelos:

- Modelo "Burro", ou previsão pela média;
- Modelo de Regressão Linear;
- Modelo da Árvore de Regressão;
- Modelo de Florestas Aleatórias;
- Modelo de Redes Neurais.

Além disso, para facilitar a avaliação dos modelos, adotaremos um indicador numérico unitário de erro. O indicador escolhido será o **RMSE**. O RMSE *(root-mean-square error)* representa o desvio padrão das diferenças entre os valores estimados e os valores esperados e é calculado como a raiz quadrada do SSE *(sum of squared errors of prediction* dividido pelo número de registros. O interessante desse indicador é que ele dá uma estimativa de erro na mesma escala da variável.

Uma observação importante é que quanto normalizamos a escala dos dados o RMSE perde um pouco da sua vantagem sobre o MSE *(Mean-square-error)*, visto isso quanto optarmos por normalizar a escala do problema vamos gerar ambos os indicadores de erro para termos de comparação.

**Em primeiro lugar vamos criar um data.frame para armazenar nossos resultados e os nossos folds para validação cruzada**

```{r results}
results <- data.frame(Nome = character(0), MSE = integer(0), RMSE = integer(0))

folds <- cut(seq(1,nrow(imdb_data)),breaks=10,labels=FALSE)
folds_size = length(unique(folds))
```
---


### Previsão pela Média

O modelo de previsão pela média consiste em estimar os valores de uma variável através da média dos valores observados dessa mesma variável.

```{r ModeloMédia}
percent = 0
sum_RMSE = 0
sum_MSE = 0
SSE = 0
for(i in 1:10){
  testIndexes <- which(folds==i,arr.ind=TRUE)
  testData <- imdb_data[testIndexes, ]
  trainData <- imdb_data[-testIndexes, ]
  directorHistory = summarise(group_by(testData ,director_name),mean=mean(imdb_score),number = n())
  testData <- DMeanNorm(testData)
  directorHistory = summarise(group_by(trainData ,director_name),mean=mean(imdb_score),number = n())
  trainData <-DMeanNorm(trainData)
  DumbPredict = mean(trainData$imdb_score)
  SSE = sum((testData$imdb_score - DumbPredict)^2)
  sum_RMSE = sum_RMSE + sqrt(SSE / nrow(testData))
  sum_MSE = sum_MSE + SSE / nrow(testData)
  SSE = 0
  percent = percent+ 10
  print(paste(percent,"%",sep=""))
}


results <- rbind(results,data.frame(Nome = "Previsão pela Média",
                                      RMSE = sum_RMSE/folds_size ,
                                      MSE = sum_MSE/folds_size)
                   )
results
```


## Modelo de Regressão Linear

O modelo de Regressão Linear consiste em estimar uma variável *(chamada de variável dependente)* a partir de outra variável *(chamada de variável independente)* estimando os parâmetros da relação linear entre elas.

Uma relação linear é do tipo: y(t) = ax(t) + b, onde y(t) é a variável dependente e x(t) é a variável independente.


```{r ModeloLinear, echo=FALSE}
summary(linearModel)
percent = 0
sum_RMSE = 0
sum_MSE = 0
SSE = 0
for(i in 1:10){
  testIndexes <- which(folds==i,arr.ind=TRUE)
  testData <- imdb_data[testIndexes, ]
  trainData <- imdb_data[-testIndexes, ]
  directorHistory = summarise(group_by(testData ,director_name),mean=mean(imdb_score),number = n())
  testData <- DMeanNorm(testData)
  directorHistory = summarise(group_by(trainData ,director_name),mean=mean(imdb_score),number = n())
  trainData <-DMeanNorm(trainData)
  linearModel = lm(model_variables,data=trainData)
  linerPrediction = predict(linearModel,testData)
  SSE = sum((testData$imdb_score - linerPrediction)^2)
    sum_RMSE = sum_RMSE + sqrt(SSE / nrow(testData))
  sum_MSE = sum_MSE + SSE / nrow(testData)
  SSE = 0
  percent = percent+ 10
  print(paste(percent,"%",sep=""))
}
results <- rbind(results,data.frame(Nome = "Modelo Linear",
                                      RMSE = sum_RMSE/folds_size ,
                                      MSE = sum_MSE/folds_size)
                   )
results

ggplot(testData,aes(x=testData$imdb_score, y=linerPrediction )) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Modelo Linear") + xlab("Conjunto de Testes")
```

Como esperado, o modelo de Regressão Linear obteve um resultado melhor que o modelo de previsão pela média, porém ainda não apresenta um bom resultado. Muito disso porque, como vimos em diversas plotagens anteriormente, a maior parte das variáveis não segue uma relação linear.


## Modelo da Árvore de Regressão

O modelo da Árvore de Regressão consiste em criar uma árvore de decisão cujo resultado é um número que representa o valor estimado da variável. Os nós de decisão da árvore de regressão são formados por outras variáveis do conjunto de dados que são selecionadas pelo algoritmo.

``` {r modeloArvoreDeRegressao}
set.seed(3)
m.rpart <- rpart(imdb_score~.,data=trainData)
m.rpart
```

```{r plotArvoreR}
rpart.plot(m.rpart)
```


```{r}
p.rpart <- predict(m.rpart,testData)
tree_dataframe <- data.frame(p.rpart,testData$imdb_score)
ggplot(tree_dataframe, aes(x=p.rpart)) + geom_histogram(fill="white", colour="black")

```

```{r}
ggplot(tree_dataframe, aes(x=testData$imdb_score)) + geom_histogram(fill="white", colour="black")
```

```{r}
cor(p.rpart,testData$imdb_score)
```


```{r}
SSE = sum((testData$imdb_score - p.rpart)^2)
results <- rbind(results,data.frame(Nome = "Àrvore de Regressão",
                                      RMSE = sqrt(SSE / nrow(testData)),
                                      MSE = SSE / nrow(testData))
                   )
results
```

Diferentemente do que era esperado, o modelo de árvore de regressão obteve um resultado pior do que o modelo de regressão linear, que é mais simples. Uma possível causa deste resultado é a sensibilidade desse modelo a outliers.


## Modelo de Florestas Aleatórias

O modelo de florestas aleatórias cria um determinado número de árvores nas quais ele distribui os parâmetros aleatóriamente. Em geral, de todas essas árvores geradas aleatóriamente, aquela que obtem o melhor resultado é melhor do que a árvore criada pelo algoritmo no modelo de árvore de regressão simples. Nesta análise, foram criadas florestas aleatórias com diferentes números de árvores para obter aquele que gera o melhor resultado.


```{r ModeloFlorestaaleatória}
array_ntree<- c(100,200,300,400,500,600,700,800,900)
percent = 0
sum_RMSE = 0
sum_MSE = 0
SSE = 0
for(j in 1:10){
  testIndexes <- which(folds==j,arr.ind=TRUE)
  testData <- imdb_data[testIndexes, ]
  trainData <- imdb_data[-testIndexes, ]
  directorHistory = summarise(group_by(testData ,director_name),mean=mean(imdb_score),number = n())
  testData <- DMeanNorm(testData)
  directorHistory = summarise(group_by(trainData ,director_name),mean=mean(imdb_score),number = n())
  trainData <-DMeanNorm(trainData)
  RMSE_vector <- c()
  for(i in array_ntree){
    set.seed(1995)
    stevenForest = randomForest(imdb_score  ~.,data=trainData,ntree = i)
    predictForest = predict(stevenForest,newdata= testData)
    SSE = sum((testData$imdb_score - predictForest)^2)
    RMSE = sqrt(SSE / nrow(testData))
    RMSE_vector <- c(RMSE_vector,RMSE)
  }
  sum_RMSE = sum_RMSE + min(RMSE_vector)
  sum_MSE = sum_MSE + (min(RMSE_vector)*min(RMSE_vector))
  SSE = 0
  percent = percent+ 10
  print(paste(percent,"%",sep=""))
}
results <- rbind(results,data.frame(Nome = "Floresta Aleatória",
                                      RMSE = sum_RMSE/folds_size ,
                                      MSE = sum_MSE/folds_size)
                   )
results


```

**Desempenho do modelo de acordo com o número de árvores utilizadas**

```{r desempenhoVsArvores}
ggplot(data.frame(array_ntree,RMSE_vector), aes(x=(array_ntree), y=RMSE_vector)) + geom_line() + geom_point()
```

Como esperado, o modelo de florestas aleatórias, por ser o modelo mais complexo e robusto, obteve um excelente resultado, sendo melhor do que todos os testados anteriormente.


## Modelo de Rede Neural

```{r RedeNeural}
#Validação 5 ciclos
folds_n <- cut(seq(1,nrow(imdb_data)),breaks=5,labels=FALSE)
folds_size_n = length(unique(folds))

percent = 0
sum_RMSE = 0
sum_MSE = 0
SSE = 0
for(i in 1:folds_size_n){
  testIndexes <- which(folds_n==i,arr.ind=TRUE)
  testData <- imdb_data[testIndexes, ]
  trainData <- imdb_data[-testIndexes, ]
  directorHistory = summarise(group_by(testData ,director_name),mean=mean(imdb_score),number = n())
  testData <- DMeanNorm(testData)
  directorHistory = summarise(group_by(trainData ,director_name),mean=mean(imdb_score),number = n())
  trainData <-DMeanNorm(trainData)
  n <- names(trainData)
  nn <- neuralnet(model_variables,data= trainData,hidden=c(50,50),linear.output=,stepmax=1e6)
  pr.nn <- compute(nn,testData[,-c(which(colnames(testData)=="imdb_score"))])
  SSE = sum((testData$imdb_score - pr.nn$net.result)^2)
  sum_RMSE = sum_RMSE + sqrt(SSE / nrow(testData))
  sum_MSE = sum_MSE + SSE / nrow(testData)
  print(sum_RMSE/i)
  SSE = 0
  percent = percent+ 20
  print(paste(percent,"%",sep=""))
}


results <- rbind(results,data.frame(Nome = "Rede Neural",
                                      RMSE = sum_RMSE/folds_size ,
                                      MSE = sum_MSE/folds_size)
                   )
results


ggplot(testData,aes(x=testData$imdb_score, y=pr.nn$net.result)) + geom_point(colour="grey60") +
  stat_smooth(method=lm, se=FALSE, colour="black") + ylab("Rede Neural") + xlab("Conjunto de Testes")

```
  
