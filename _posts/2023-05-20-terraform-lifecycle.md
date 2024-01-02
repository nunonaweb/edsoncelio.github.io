---
title: Modificando o comportamento padrão do ciclo de vida de recursos do Terraform
author: edson
date: 2023-05-20
categories: [IaC]
tags: [terraform, ptbr]
---


Nesse post, vou tentar abordar um pouco sobre o ciclo de vida de recursos do Terraform e falar um pouco sobre um bloco de meta-argumento bem especial.

## O que acontece quando eu executo um `terraform apply?`

De forma resumida, quando você executa um `apply`, o comportamento padrão do Terraform é tentar realizar uma (ou mais de uma) das seguintes tarefas:

* Criar recursos que existem no seu código mas não estão associados a um objeto da infraestrutura real no *state*.
    
* Atualizar em execução recursos que tiveram alguma mudança na sua configuração em código.
    
* Destruir recursos que existem no *state* mas não existem mais no seu código.
    
* Destruir e recriar recursos que tiveram alguma mudança na sua configuração mas não podem ser atualizados em execução por limitações da API do provider.
    

Como deu pra ver, o `apply` não vai só criar ou atualizar recursos em execução, em certos casos vai ser preciso destruir o recurso.

## O bloco de meta-argumento `lifecycle`

Pensando em casos onde por algum motivo você não quer que o Terraform tenha esse comportamento padrão em um `apply`, existe um bloco de meta-argumento bem especial chamado `lifecycle`:

```plaintext
  lifecycle {
   
  }
```

Esse bloco pode ser adicionado no corpo de um `resource` e tem os seguintes argumentos:

* `create_before_destroy (booleano)`: Caso seja `true` vai alterar o comportamento para casos onde a atualização precise destruir e recriar o recurso, nesse caso vai primeiro criar o novo recurso e só depois destruir.
    
* `prevent_destroy (booleano)`: Caso seja `true` o Terraform vai retornar um erro em **qualquer** `plan` que tenha a destruição de algum recurso.
    
* `ignore_changes (lista de nomes de atributos)`: Para a lista de atributos recebida, o Terraform vai ignorar mudanças nesses atributos. Um caso especial desse argumento é que caso seja uma lista com a *keyword* `all` o Terraform nunca vai verificar por mudanças.
    

Abaixo um exemplo com todos os argumentos:

```plaintext
resource "aws_instance" "my_instance" {

....
  lifecycle {
    prevent_destroy = true 
    create_before_destroy = true # esse caso vai ser desnecessário porque já existe o prevent_destroy habilitado.
    ignore_changes = [
      tags,
    ]
  }
}
```

## Referências

* [https://www.terraform.io/language/resources/behavior](https://www.terraform.io/language/resources/behavior)
    
* [https://www.terraform.io/language/meta-arguments/lifecycle](https://www.terraform.io/language/meta-arguments/lifecycle)