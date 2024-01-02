---
title: Terraform datasources - external
author: edson
date: 2023-12-20
categories: [IaC]
tags: [terraform, ptbr]
---

Continuando o assunto de datasource que falei no post anterior, dessa vez vou falar um pouco sobre outro tipo especial de datasource do Terraform: o `external`.

> Antes de continuar, é importante comentar que na documentação esse recurso é chamado de *escape hatch* (saída de emergência), e dever ser usado apenas em situações excepcionais onde um outro datasource de um provider não atenda.

## Provider: `external`

Esse datasource faz parte do provider *external*, que não requer nenhuma configuração, basicamente é usado para prover uma interface entre o Terraform e aplicações externas, para ser usado em casos onde outros providers não atendem.

## Datasource: `external`

O datasource external permite que uma aplicação externa (usando um protocolo específico) seja usada como entrada de dados, expondo uma saída para ser usada no código.

Ex: imagine que você por algum motivo (e em algum cenário muito específico) precisa criar algum recurso com base na saída de um script com uma determinada entrada.

Nesse caso, o datasource vai ficar assim:

```plaintext
data "external" "external_data" {
  # chamada do script/aplicação
  program = ["go", "run", "${path.module}/external_datasource.go"]

  # parâmetros de entrada, nesse caso é o 'sound'
  query = {
    sound = "honk"
  }
}

output "response" {
    value = data.external.external_data.result
}
```

Onde o script [`external_datasource.go`](https://github.com/edsoncelio/terraform-datasources/blob/main/external/external_datasource.go) é:

```plaintext
package main

import (
	"encoding/json"
	"log"
	"os"
)

type InputQuery struct {
	Query string `json:"sound"`
}

type OutputQuery struct {
	Code    string `json:"code"`
	Message string `json:"message"`
}

func main() {
	var inputQuery InputQuery
	var outputQuery OutputQuery

	err := json.NewDecoder(os.Stdin).Decode(&inputQuery)
	if err != nil {
		log.Fatal(err)
	}

	if inputQuery.Query == "honk" {
		outputQuery.Code = "200"
		outputQuery.Message = "successful honk"
	} else {
		outputQuery.Code = "400"
		outputQuery.Message = "failed honk"
	}

	if err := json.NewEncoder(os.Stdout).Encode(outputQuery); err != nil {
		log.Fatal(err)
	}

}
```

Nesse caso, a saída vai ser:

```plaintext
$ terraform output
response = tomap({
  "code" = "200"
  "message" = "successful honk"
})
```

Importante falar também sobre os requisitos do script para ser usado:

* A aplicação deve ler a entrada via `stdin` e parsear como um objeto JSON, esse objeto vai conter o valor de `query`, que vai ser sempre `string`;
    
* A saída deve ser também um objeto JSON válido via `stdout`, esse objeto vai popular o atributo `result` no output do Terraform, e os valores também precisam ser todos `string`;
    
* Qualquer erro na aplicação deve ser retornado no `stderr` com um código de saída diferente de zero.
    

Caso alguém queira brincar um pouco mais com isso, deixo os códigos usados nas referências.

## Referências

* [https://registry.terraform.io/providers/hashicorp/external/latest/docs](https://registry.terraform.io/providers/hashicorp/external/latest/docs)
    
* [https://registry.terraform.io/providers/hashicorp/external/latest/docs/data-sources/data\_source](https://registry.terraform.io/providers/hashicorp/external/latest/docs/data-sources/data_source)
    
* [https://github.com/edsoncelio/terraform-datasources/tree/main/external](https://github.com/edsoncelio/terraform-datasources/tree/main/external)