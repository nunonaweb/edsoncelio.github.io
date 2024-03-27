---
title: Grafana as a Code 101 
author: edson
date: 2024-03-18
categories: [Iac, Observability]
tags: [grafana, iac, ptbr]
---

Nesse post vou falar um pouco sobre as alternativas para quem quer gerenciar todos (ou quase todos) os recursos de uma instância do Grafana usando código.   
A ideia aqui é mostrar as opções que considero as melhores atualmente, espero que sirva pra ajudar a escolher uma delas no seu cenário de trabalho.

## Grafana Provider (Terraform)

A primeira opção é o provider do Terraform para o Grafana. O provider tem suporte tanto para Grafana OSS quanto para Grafana Cloud.

### Autenticação

Via string, que pode ser um API token, usuário e senha (no formato user:pass) ou de forma anônima (com a string *anonymous*):

```
provider "grafana" {
  url  = "http://grafana.example.com/"
  auth = var.grafana_auth
}
```

### Recursos que podem ser gerenciados

Praticamente todos os recursos, tanto os recursos "core" quanto outras funcionalidades do Grafana Cloud, como Alerting, OnCall e Synthetic Monitoring.
Uma lista, não exaustiva de recursos que podem ser gerenciados no Grafana OSS:
* grafana_service_account
* grafana_service_account_token
* grafana_user
* grafana_team
* grafana_organization
* grafana_folder
* grafana_datasource
* grafana_dashboard

Para ver todos os recursos você pode consultar as docs do provider [aqui](https://registry.terraform.io/providers/grafana/grafana/latest/docs).


## Grafana Operator (Kubernetes)
Essa é uma das opções mais interessantes pra quem quer manter a gestão dos recursos via manifestos do Kubernetes (o nosso querido yaml).   
Com o Operator do Grafana você vai conseguir fazer deploy e gerenciar instâncias do Grafana de dentro de um cluster Kubernetes e também gerenciar instâncias externas (como o Grafana Cloud).

## Grizzly

TODO








## Pontos de atenção

Manter dashboards como código pode ser um desafio, principalmente quando a ferramenta que você está usando simplesmente recebe um arquivo .json da dashboard como entrada.    
E sendo sincero, não é nada agradável criar e alterar dashboards do Grafana diretamente via .json, dependendo da complexidade da dashboard pode se tornar um trabalho bem cansativo.


---

Aqui eu trouxe apenas as opções que considero as melhores, mas se quiser ver uma lista completa pode dar uma olhada nesse blog post da Grafana que tem várias opções e faz comparações entre elas: [https://grafana.com/blog/2022/12/06/a-complete-guide-to-managing-grafana-as-code-tools-tips-and-tricks/](https://grafana.com/blog/2022/12/06/a-complete-guide-to-managing-grafana-as-code-tools-tips-and-tricks/).



