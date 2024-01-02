---
title: Terraform datasources - http
author: edson
date: 2023-05-20
categories: [IaC]
tags: [terraform, ptbr]
---

*Datasources* são recursos do Terraform que dão a possibilidade de buscar e usar informações definidas fora do Terraform (em outra configuração por exemplo) dentro do seu código.

Nesse post vou mostrar um pouco de um tipo de datasource que pode salvar bastante no dia a dia.

## Datasource `http`

Esse datasource basicamente faz uma requisição `GET` em uma dada URL e exporta o *body* e os *headers* da resposta como saída.

Exemplo: Suponha que você precise criar um *security group* para uma EC2 na AWS e já adicionar seu endereço IP atual na lista de endereços permitidos, como poderíamos fazer?

Usando o datasource `http`:

```plaintext
data "http" "myip" {
  url = "https://api.ipify.org"

  # Opcional: header da requisição
  request_headers = {
    Accept = "application/text"
  }
}

output "myip" {
    description = "Endereço IP"
    value = data.http.myip.body
}
```

Nesse caso, foi feito um `GET` na URL [https://api.ipify.org](https://api.ipify.org) para buscar o IP atual com o *header* do tipo *text*, e a saída foi exposta via *output*.

É importante ressaltar que a URL consultada precisa responder `200 OK` e com um `Content-Type` do tipo `text/*` ou `application/json`.

## Referências

* [https://registry.terraform.io/providers/hashicorp/http/latest/docs/data-sources/http](https://registry.terraform.io/providers/hashicorp/http/latest/docs/data-sources/http)
    
* [https://www.terraform.io/docs/language/data-sources/index.html](https://www.terraform.io/docs/language/data-sources/index.html)