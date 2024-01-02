---
title: Conhecendo a stack de observabilidade opensource da Grafana Labs - Grafana
author: edson
date: 2023-11-20
categories: [Observability]
tags: [grafana, observability, ptbr]
---

Quando se fala de começar uma stack de observabilidade usando componentes opensource (OSS) o Grafana é uma das primeiras opções a ser citadas, e é sobre essa ferramenta que vamos falar um pouco nesse primeiro post, que (espero) vai ser o primeiro de uma série.

## Mas o que é o Grafana?

O Grafana é uma plataforma de visualização que permite que você faça consultas (*queries*), visualize, configure alertas e entenda seus dados independente de onde estejam armazenados.

## Instalação

A instalação pode ser feita de três formas:

* Via pacote específico do sistema operacional (Linux, macOS, Windows)
    
* Via imagem Docker
    
* Via helm chart em um cluster Kubernetes
    

Em todas as opções você vai encontrar informações detalhadas na documentação oficial [aqui](https://grafana.com/docs/grafana/latest/setup-grafana/installation/).

**Importante:** Se quiser testar as funcionalidades que vou descrever aqui sem precisar instalar, pode usar o [https://play.grafana.org/](https://play.grafana.org/).

## Componentes Principais

### Data sources

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1700438539044/bca73d10-e623-45df-ac13-e56c70fa911c.png">
</p>

*Data sources* em uma tradução literal seriam as fontes de dados de onde o Grafana vai receber os dados para gerar as informações.

Esses dados sources são configurados via plugins, que podem ser os plugins nativos, como:

* Prometheus
    
* ElasticSearch
    
* MySQL
    
* AWS Cloudwatch
    
    ....
    

A lista completa de plugins nativos você consegue ver [aqui](https://grafana.com/docs/grafana/latest/datasources/#built-in-core-data-sources).

Além disso, é possível [customizar plugins](https://grafana.com/developers/plugin-tools), ou usar os plugins da comunidade, que consegue ver [aqui](https://grafana.com/grafana/plugins/data-source-plugins/).

A partir de um *data source*, você pode consultar os dados usando o Explore, visualizar usando painéis e configurar alertas.

É importante citar que cada plugin possui suas particularidades e configurações específicas, então é importante ver na documentação a configuração que atende melhor ao seu cenário.

### Painéis e visualizações

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1700438713623/900556fa-93da-438e-a2d1-1dd8d9935e3d.png">
</p>

No Grafana, um painel é o bloco mais básico de visualização, cada painel traz consigo um editor de consultas específico do data source selecionado naquele painel.

Existe uma variedade de opções para estilo e formatação de cada painel.

### Dashboards

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1700438586621/7707a5f6-6abc-49b1-993f-9629eb23adcf.png">
</p>

Uma dashboard é um conjunto de um ou mais painéis, organizados em uma ou mais linhas. Você não precisa configurar uma dashboard do zero se for um cenário comum, é possível consultar uma série de dashboards da comunidade mantidas pela Grafana [nessa biblioteca](https://grafana.com/grafana/dashboards/?pg=docs-grafana-latest-dashboards).

### Explore

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1700438731862/5d1f3e4a-e313-4db6-8d41-0ea24a974be4.png">
</p>


Se você quiser só explorar os dados sem precisar criar uma dashboard, você pode fazer isso facilmente com o Explore. Se o *data* *source* que usar tiver suporte a exibição dos dados em tabela e gráfico o resultado vai ser mostrado nos dois tipos.

### Alertas

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1700439250181/fb1b8cc0-a2f2-413a-8cd4-4b97b0e4add5.png">
</p>

Essa funcionalidade é uma das mais importantes: a capacidade de gerar alertas sempre que algo tiver fora do esperado. Com esse recurso, você gerar consultas e expressões a partir de múltiplos data sources, ainda tendo a possibilidade de combinar métricas e logs para gerar os alertas.

Algumas funcionalidades que valem a pena ser citadas:

* Cada alerta pode ser roteado para um contact point específico (slack, email e outros) baseado em labels definidas nos alertas,
    
* É possível silenciar notificações de alertas que precisam ser investigados por uma janela de tempo específica.
    

[Aqui](https://grafana.com/docs/grafana/latest/alerting/#design-your-alerting-system) você tem algumas dicas de boas práticas para a criação e manutenção dos alertas.

## Gestão de Acessos a Recursos

A estrutura de administração de recursos no Grafana de forma resumida se parece com isso:

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1700483570264/caf73155-a32b-4930-9e0f-43e0975adc82.png">
</p>


Onde,

* Grafana instance é a instância do Grafana,
    
* Organization, é uma estrutura que permite que você isole recursos como dashboards, alerts uns dos outros,
    
* Role, é a função que tem atrelada um conjunto de permissões que determinado usuário vai ter,
    
* Usuário, indivíduo que tem acesso ao Grafana
    

Existem muitos outros detalhes relacionados a configuração de acessos, recomendo ler a documentação [aqui](https://grafana.com/docs/grafana/latest/administration/) pra ter uma visão mais completa do que é possível fazer.

---

A partir do próximo blog post dessa série, vamos começar um projeto prático usando os outros componentes da stack, começando pelo Loki.