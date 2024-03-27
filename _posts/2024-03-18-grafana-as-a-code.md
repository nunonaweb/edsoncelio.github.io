---
title: Alternativas para Gerenciar Recursos do Grafana como Código
author: edson
date: 2024-03-27
categories: [Iac, Observability]
tags: [grafana, iac, ptbr]
---

Nesse post vou falar um pouco sobre as alternativas para quem quer gerenciar todos (ou quase todos) os recursos de uma instância do Grafana usando código.   
A ideia aqui é mostrar as opções que considero as mais interessantes atualmente, espero que sirva pra ajudar a escolher uma delas no seu cenário de trabalho.

## Grafana Provider (Terraform)

A primeira opção é o provider do Terraform para o Grafana, que talvez seja a opção mais conhecida.   
O provider tem suporte tanto para Grafana OSS quanto para Grafana Cloud.

Link para acesso a documentação: https://registry.terraform.io/providers/grafana/grafana/latest/docs

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

A instalação do Operator tanto via Helm quanto via Kustomize.   

Instalação via Helm:
```bash
helm upgrade -i grafana-operator oci://ghcr.io/grafana/helm-charts/grafana-operator --version v5.7.0
```

Link para acesso a documentação: https://grafana.github.io/grafana-operator/docs/quick-start/

### Autenticação
A autenticação pode ser feita tanto via Secrets quanto via config do CRD.   
Autenticação via Secret:

```yaml
---
kind: Secret
apiVersion: v1
metadata:
  name: credentials
  namespace: grafana
stringData:
  GF_SECURITY_ADMIN_PASSWORD: secret
  GF_SECURITY_ADMIN_USER: root
type: Opaque
```

Autenticação via config:
```yaml
---
apiVersion: grafana.integreatly.org/v1beta1
kind: Grafana
metadata:
  name: grafana
  labels:
    dashboards: "grafana"
spec:
  config:
    log:
      mode: "console"
    auth:
      disable_login_form: "false"
    security:
      admin_user: root
      admin_password: secret
```
Mais detalhes você pode ver [aqui](https://grafana.github.io/grafana-operator/docs/examples/credential_secret/readme/) e [aqui](https://grafana.github.io/grafana-operator/docs/examples/credential_config/readme/).


### Recursos que podem ser gerenciados

Os recursos que podem ser gerenciados pelo Operator:
* instâncias do Grafana
* Datasources
* Alerts
* Dashboards
* Folders


Um ponto interessante aqui é relacionado as dashboards, que podem ser gerenciadas não só em json, mas também nas formas abaixo:
* json
* gzipJson
* URL
* Jsonnet

## Grizzly

Grizzly é uma CLI que permite que você gerencie alguns recursos do Grafana usando yaml ou Json (com Jsonnet).

Como é uma CLI escrita em Golang, o Grizzly é apenas um binário, então a instalação é simplificada:

```bash
# download the binary (adapt os and arch as needed)
$ curl -fSL -o "/usr/local/bin/grr" "https://github.com/grafana/grizzly/releases/download/v0.4.0/grr-linux-amd64"

# make it executable
$ chmod a+x "/usr/local/bin/grr"

# have fun :)
$ grr --help
```
O download das versões é feito diretamente da página de releases [aqui](https://github.com/grafana/grizzly/releases).

Link para acesso a documentação: https://grafana.github.io/grizzly/

### Autenticação

Para autenticar em uma instância do Grafana:
```bash
grr config set grafana.url http://localhost:3000 
grr config set grafana.user admin # Opcional: Usuário se tiver usando autenticação básica
grr config set grafana.token abcd12345 # Token da service account (ou senha se usar autenticação básica)
```

O Grizzly também gerencia recursos do Grafana Cloud Prometheus (Mimir) e do Synthetic Monitoring, sendo que cada um tem sua forma de autenticação.     
Mais detalhes [aqui](https://grafana.github.io/grizzly/configuration/).


### Recursos que podem ser gerenciados

O Grizzly gerencia os seguintes recursos de instâncias Grafana:
* Dashboards
* Folders
* Datasources
* Panels e Variables
* Contact Points (alertas)
* Notification Policy (alertas)

Já relacionado ao Mimir, os seguintes recursos podem ser relacionados:
* Prometheus Alerts
* Prometheus Recording Groups

E por último, também gerencia checks de Synthetic Monitoring.

Comparado as outras opções aqui é possível definir dashboards como yaml, como por exemplo:
```yaml
apiVersion: grizzly.grafana.com/v1alpha1
kind: Dashboard
metadata:
    name: prod-overview
    folder: general
spec:
    schemaVersion: 17
    tags:
        - templated
    timezone: browser
    title: Production Overview
    uid: prod-overview
```


## Pontos de atenção

Manter dashboards como código pode ser um desafio, principalmente quando a ferramenta que você está usando simplesmente recebe um arquivo .json da dashboard como entrada.    

E sendo sincero, não é nada agradável criar e alterar dashboards do Grafana diretamente via .json, dependendo da complexidade da dashboard pode se tornar um trabalho bem cansativo então é importante sempre colocar na balança se vale a pena ou não manter 100% do fluxo via código.


---
---

Uma lista completa de ferramentas (incluindo collection do Ansible e provider do Crossplane) pode ser vista nesse blog post da Grafana, inclusive com comparação entre elas: [https://grafana.com/blog/2022/12/06/a-complete-guide-to-managing-grafana-as-code-tools-tips-and-tricks/](https://grafana.com/blog/2022/12/06/a-complete-guide-to-managing-grafana-as-code-tools-tips-and-tricks/).



