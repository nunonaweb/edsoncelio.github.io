---
title: Recurso experimental de testes do Terraform
author: edson
date: 2023-12-20
categories: [IaC]
tags: [terraform, ptbr]
---

> Importante: esse recurso não é mais experimental, já está disponível nas versões mais recentes.
{: .prompt-tip }

## Onde se quer chegar

Quando falamos sobre escrever testes para IaC com Terraform temos algumas opções que podem ser usadas atualmente, vou falar de forma rápida sobre cada uma delas até chegarmos na solução proposta.

### Terratest

Biblioteca em Go feita com a ideia de facilitar a escrita de testes automatizados para infraestrutura como código. Apesar do nome, não é de uso exclusivo com Terraform, algumas das funcionalidades incluem:

* Teste para templates packer;
    
* Teste para imagens docker;
    
* Funções para executar comandos via ssh, executar requisições HTTP;
    
* Funciona com APIs da AWS, GCP e Azure.
    

Exemplo de um teste pode ser visto em [https://github.com/gruntwork-io/terratest/blob/master/test/terraform\_aws\_s3\_example\_test.go](https://github.com/gruntwork-io/terratest/blob/master/test/terraform_aws_s3_example_test.go)

### Kitchen-Terraform

Biblioteca em Ruby que permite que sejam usados os benefícios do TDD (*test-driven development*) com projetos Terraform por meio de plugins do [Kitchen](https://kitchen.ci/index.html).

Um exemplo de uso pode ser visto em [https://newcontext-oss.github.io/kitchen-terraform/tutorials/amazon\_provider\_ec2.html](https://newcontext-oss.github.io/kitchen-terraform/tutorials/amazon_provider_ec2.html)

---

O que essas duas bibliotecas tem em comum? usam a abordagem de escrever os testes em uma outra linguagem que não é a 'linguagem do Terraform'.

Isso pode trazer vantagens e desvantagens (um exemplo seria a necessidade de aprender uma outra linguagem para escrever os testes), então foi pensando nisso que foi liberado de forma experimental um recurso de testes nativo do Terraform (dando a capacidade de escrever os testes em HCL)

## Testes nativos no Terraform: `terraform test`

Esse recurso foi liberado na versão `0.15.0` e consiste nos seguintes componentes:

* Um provider experimental usado para fazer os *asserts* dos testes (agindo como as bibliotecas que citei antes): [`terraform.io/builtin/test`](http://terraform.io/builtin/test);
    
* O comando `terraform test` para facilitar a execução de múltiplos testes de uma única vez;
    
* Uma convenção experimental de organizar os testes em subdiretórios dentro do diretório `tests/`, de onde o comando `terraform test` vai buscar os testes a serem executados.
    

### Como funciona na prática

Imaginando que temos um módulo de exemplo para criar buckets s3 na aws com a seguinte estrutura:

```plaintext
.
├── main.tf
├── outputs.tf
├── providers.tf
├── tests # aqui é o diretório onde vão os testes criados
├── variables.tf
└── versions.tf
```

Dentro do diretório de testes vamos escrever os cenários de testes para testar o módulo com os valores padrões(*defaults*):

```plaintext
.
├── main.tf
├── outputs.tf
├── providers.tf
├── tests
│   └── defaults
│       └── test_defaults.tf
├── variables.tf
└── versions.tf
```

Agora efetivamente escrevendo o teste, teríamos o seguinte:

```plaintext
terraform {
  required_providers {
    test = {
      source = "terraform.io/builtin/test"
    }
  }
}

# locals podem ser usados normalmente
locals {
  region = "us-east-1"
  access = "private"
}

module "main" {
  source = "../.."
}

resource "test_assertions" "example_checks" {
  component = "unique_name_to_this_assertion"

  equal "region" {
    description = "resource region is us-east-1"
    got         = module.main.resource_region
    want        = local.region
  }

  check "access" {
    description = "default port number is 80"
    condition   = can(local.access, module.main.resource_access)
  }
}
```

Onde:

* O *resource* `test_assertions` é o responsável por receber as asserções dos testes;
    
* A variável `component` é usada como identificador único do *resource* de asserção;
    
* os blocos `equal` e `check` servem para fazer as asserções dos testes.
    

Executando o `terraform test`, temos:

```plaintext
❯ tf test
╷
│ Warning: The "terraform test" command is experimental
│ 
│ We'd like to invite adventurous module authors to write integration tests for their modules using this command, but all of the behaviors of this command are currently experimental and may
│ change based on feedback.
│ 
│ For more information on the testing experiment, including ongoing research goals and avenues for feedback, see:
│     https://www.terraform.io/docs/language/modules/testing-experiment.html
╵
Success! All of the test assertions passed.
```

Uma outra possibilidade é usar programas externos (em uma linguagem de sua preferência) com o data `testing_tap` que usa o [Test Anything Protocol](https://testanything.org/) pra interpretar.

Mais exemplo de uso e todas as possibilidades podem ser vistas no repositório do provider: [https://github.com/apparentlymart/terraform-provider-testing/blob/master/docs/index.md](https://github.com/apparentlymart/terraform-provider-testing/blob/master/docs/index.md)

## Referências

* [https://www.terraform.io/docs/language/modules/testing-experiment.html](https://www.terraform.io/docs/language/modules/testing-experiment.html)
    
* [https://github.com/gruntwork-io/terratest](https://github.com/gruntwork-io/terratest)
    
* [https://newcontext-oss.github.io/kitchen-terraform/](https://newcontext-oss.github.io/kitchen-terraform/)
    
* [https://github.com/apparentlymart/terraform-provider-testing/blob/master/docs/index.md#external-test-programs](https://github.com/apparentlymart/terraform-provider-testing/blob/master/docs/index.md#external-test-programs)