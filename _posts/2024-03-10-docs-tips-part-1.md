---
title: Dicas de Escrita Técnica - parte 01
author: edson
date: 2023-11-20
categories: [Documentation]
tags: [documentation, ptbr]
---

Normalmente quando me perguntam sobre documentação indico [esse curso](https://developers.google.com/tech-writing/overview) introdutório de escrita técnica da Google.
Baseado nisso, vou compilar alguns dos pontos que considero importantes.

> Como o curso é em inglês (e focado no inglês) tomei a liberdade de fazer uma tradução livre :)
{: .prompt-info }

## Palavras
Dicas relacionadas a palavras (que vou chamar de termos também) na documentação.

### Defina termos novos (ou que não são familiares pra audiência)
Sempre que tiver escrevendo uma documentação, fique atento a termos que podem não ser familiar (ou novos) pra sua audiência. Para esses casos, pode usar duas abordagens:
* Se o termo já existe, aponte para um link de uma boa explicação (não precisa reinventar a roda)
* Se o documento em questão tá introduzindo o termo, defina o termo. Se perceber que está definindo termos demais, separe todas as definições em um glossário

### Use termos de forma consistente
Se você alterar o nome de uma variável no meio de um método, seu código não vai compilar. Da mesma forma, se você renomear um termo no meio de um documento, suas ideias não serão compiladas (na cabeça dos usuários).

Moral da história: aplique a mesma palavra ou termo de forma consistente durante todo o seu documento.

### Use acrônimos de forma apropriada
No primeiro uso de um acrônimo desconhecido em um documento ou seção, soletrar o termo completo e em seguida colocar o acrônimo entre parênteses. Coloque a versão soletrada e a sigla em negrito. Por exemplo:   
"A **Cloud Native Computing Foundation** (**CNCF**) hospeda componentes críticos da infraestrutura tecnológica global."

Dai em diante você pode usar apenas o acrônimo sem problemas.


## Sentenças curtas
Na escrita de código, normalmente tentamos diminuir a quantidade de linha pelos seguintes motivos:
* Código mais curto normalmente é mais fácil de ser lido por outras pessoas.
* Código curto normalmente é mais fácil de manter.

Na escrita técnica temos razões similares pra escrever sentenças menores:
* Documentação curta permite uma leitura mais rápida.
* Documentação curta é mais fácil de manter.

Em resumo: sentenças mais curtas comunicam de forma melhor que sentenças longas além de normalmente serem mais fáceis de entender. 

### Foque em uma única ideia por sentença
Foque cada sentença em uma ideia, pensamento ou conceito. Assim como as instruções de um programa executam uma única tarefa, uma sentença deve representar uma única ideia.

### Converter sentenças longas em listas
Sentenças longas podem ser convertidas em listas para mais clareza. Por exemplo, quando você ver a conjunção **OU** em uma sentença longa, considere reescrever a sentença em uma lista com marcadores (conhecidas como *bullets*)

## Listas e Tabelas
Boas listas podem transformar informações caóticas em algo ordenado. 

### Escolha do tipo de lista
Normalmente existem os seguintes tipos de listas:
* Listas com marcadores
* Listas numeradas
* Listas embutidas (do inglês *embedded*)

Use uma lista com marcadores para itens não ordenados e lista numeradas para itens ordenados. Resumindo:
* Se você bagunçar os itens de uma lista com marcadores, o significado da lista não vai mudar.
* Se você bagunçar os itens de uma lista numerada, o significado da lista vai mudar

Uma lista embutida contém itens inseridos em uma frase. De modo geral, listas embutidas são uma forma inadequada de apresentar informações técnicas. Sempre que possível, tente transformar em uma lista com marcadores ou numerada.

### Comece itens de listas numeradas com verbos no imperativo
Considere começar itens de listas numeradas com um verbo no imperativo. Um verbo imperativo é um comando, como abra ou inicie.

### Pontue os itens adequadamente
A recomendação do [guia de estilo de desenvolvimento da google](https://developers.google.com/style/lists#capitalization-and-end-punctuation) é começar cada item da lista em caixa alta (com a primeira letra em maiúsculo).   
Se a lista é uma sentença use a pontuação adequada no final, por exemplo:
1. Abra o navegador.
2. Digite google.com na barra de pesquisa.


### Crie tabelas úteis
Tabelas são bem úteis, de uma página contendo muitos parágrafos e uma tabela, normalmente os olhos se voltam para a tabela.

Algumas recomendações:
* Nomeie as colunas com um nome significado. Não faça os leitores adivinharem o que cada coluna contém.
* Evite colocar muito texto em uma célula da tabela. Se uma célula da tabela contiver mais de duas sentenças, se pergunte se essa informação não pertence a outro formato.
* Embora as colunas possam conter diferentes tipos de dados, busque o paralelismo dentro de colunas individuais. Por exemplo, as células dentro de uma coluna específica da tabela não devem ser uma mistura de dados numéricos e artistas de circo famosos.

### Introduza listas e tabelas
É recomendado introduzir as listas com tabelas com uma sentença que explique para os leitores o que a lista ou a tabela apresenta. Termine a sentença introdutória com dois pontos ao invés de ponto final.

