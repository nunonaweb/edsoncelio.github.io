---
title: Contribuindo pro Glossário Cloud Native da CNCF em Português
author: edson
date: 2024-07-10
categories: [OpenSource]
tags: [cncf, glossary, docs]
---

Nesse guia rápido vou resumir o passo a passo para você começar a contribuir com o glossário da CNCF em português!

## Antes de tudo, o que é o glossário Cloud Native?
O [glossário da CNCF](https://glossary.cncf.io/pt-br/) é um projeto que visa ser uma fonte de consulta de termos relacionados ao universo nativo de nuvem (*cloud native*) de forma simples e clara, inclusive tentando tornar o entendimento fácil sem precisar de conhecimento prévio.

Inicialmente o projeto foi lançado apenas em inglês, mas pouco tempo depois começamos a versão em português (do Brasil), e atualmente temos quase 100% dos termos localizados.

A ideia desse blog post é centralizar informações para quem tem interesse em contribuir com o projeto.

## Como Funciona o Projeto

O website do glossário é construído com [Hugo](https://gohugo.io/) sendo que as páginas são escritas em [Markdown](https://www.markdownguide.org/basic-syntax/).

### Repositório
O repositório no GitHub é o [cncf/glossary](https://github.com/cncf/glossary), sendo que as documentações ficam no diretório `content/`, com a seguinte estrutura:
```bash
├── content
│   ├── bn
        ...
│   ├── pt-br
```

As strings relacionadas a localização, estão no arquivo `i18n/pt-br.toml`.

### Estrutura dos Termos do Glossário
Os termos do glossário seguem o seguinte padrão:

```md
---
title: 
status: 
category: 
---


Sumário rápido da tecnologia ou conceito.

## Problema Relacionado

Algumas linhas sobre o que isso resolve.

## Como Isso Ajuda

Algumas linhas sobre como isso resolve o problema relacionado.
```

Sendo que,
* `status`: pode ser Completed, Feedback Appreciated ou Deprecated. Apenas termos com o status Completed são exibidos no glossário.
* `category:` é uma label que serve para categorizar os termos e facilitar as buscas, são valores pré-definidos pelo time que mantém o projeto.


## Fazendo a Primeira Contribuição

## Escolhendo o termo para começar
Atualmente, temos issues no repositório para os termos que ainda estão precisando ser localizados.   

Você pode fazer uma busca do tipo: `is:issue is:open label:lang/pt` (ou ir direto nessse [link](https://github.com/cncf/glossary/issues?q=is%3Aissue+is%3Aopen+label%3Alang%2Fpt)), que vai retornar as issues em aberto.

Tendo escolhido a issue, pode comentar que vai trabalhar nela, que atribuimos a você.

## Fluxo de contribuição
De forma geral, esse é o fluxo de contribuição:

![](https://supabase.mermaidchart.com/storage/v1/object/public/chatgpt-diagrams/2024-07-12/f6b7cc73-f201-4b43-950c-9af8be51f8ce.png)

Um passo além desses que é recomendado é enviar uma mensagem no canal oficial de comunicação no [slack da CNCF](https://communityinviter.com/apps/cloud-native/cncf) `##glossary-localization-portuguese` com o link do seu PR, assim o time de revisão é notificado.

---

Com isso, você deve ser capaz de fazer a sua contribuição, mas se ainda tiver dúvidas, pode me chamar ou mandar uma mensagem no canal do slack!
