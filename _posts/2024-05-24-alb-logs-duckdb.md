---
title: Analisando logs de acesso de ALBs (Application Load Balancer) localmente com DuckDB
author: edson
date: 2024-0-12
categories: [Utils]
tags: [aws, logs, ptbr]
---

Só quem já precisou fazer alguma investigação que envolvia analisar os logs de acesso de ALBs (Application Load Balancer) da AWS sabe a dor e sofrimento que é.   
Mas pra quem nunca precisou? Não tem problema! Vou explicar um pouco sobre como funciona até chegar na solução com uso do duckDB.

---

## O que são e como funcionam os logs de ALBs?
Elastic Load Balancing provides access logs that capture detailed information about requests sent to your load balancer. Each log contains information such as the time the request was received, the client's IP address, latencies, request paths, and server responses. You can use these access logs to analyze traffic patterns and troubleshoot issues.

É possível ter logs com as informações detalhadas das requisições que chegam no ALB, informações como:
* hora da request
* endereço IP do cliente
* latência
* caminho da requisição
* resposta do servidor

E muitos outros, [aqui](https://docs.aws.amazon.com/elasticloadbalancing/latest/application/load-balancer-access-logs.html#access-log-entry-syntax) pode ver todos.

Como já comecei citando, esses logs são extremamente úteis para investigação de problemas e até mesmo análises de padrões no tráfego.

Esses logs não são habilitados por padrão, e depois que habilita são armazenados em um bucket s3 (aqui que começa o terror!).    
A estrutura do bucket com os logs é a seguinte:
```bash
- AWSLogs/
  - <IDdaConta>
     - elasticloadbalancing
        - <região>
          - <ano>
            - <mês>
              - <dia>
                - <nomeDoArquivodeLog>.log.gz
```

E é assim que se parece uma linha de arquivo de log desses:

```bash
https 2018-07-02T22:23:00.186641Z app/my-loadbalancer/50dc6c495c0c9188 
192.168.131.39:2817 10.0.0.1:80 0.086 0.048 0.037 200 200 0 57 
"GET https://www.example.com:443/ HTTP/1.1" "curl/7.46.0" ECDHE-RSA-AES128-GCM-SHA256 TLSv1.2 
arn:aws:elasticloadbalancing:us-east-2:123456789012:targetgroup/my-targets/73e2d6bc24d8a067
"Root=1-58337281-1d84f3d73c47ec4e58577259" "www.example.com" "arn:aws:acm:us-east-2:123456789012:certificate/12345678-1234-1234-1234-123456789012"
1 2018-07-02T22:22:48.364000Z "authenticate,forward" "-" "-" "10.0.0.1:80" "200" "-" "-"
```

Lembrando que é uma linha dessas por requisição, e que são gerados N arquivos de log desses em um certo intervalo de tempo, então as coisas começam a complicar quando se tem uma quantidade considerável de requisições.

Imaginando um caso onde precisa buscar uma requisição que retornou um código http 500, da forma artesenal o que teria que fazer:
1. Ir no bucket s3, no path do arquivo que precisa
2. Baixar o arquivo
3. Descompactar
4. Abrir no seu editor de texto predileto e fazer uma busca pelo campo que precisa

Isso não é nem de longe a melhor (e mais rápida) forma de fazer troubleshooting, então vamos as propostas de solução pra agilizar isso.

## Agilizando a consulta de informações nos logs

### Com AWS Athena
Uma forma de fazer busca e consulta nesses arquivos de logs, é usando o AWS Athena, que pra quem não conhece é um serviço da AWS que permite que você faça consulta usando SQL em arquivos do s3.

[Aqui](https://docs.aws.amazon.com/athena/latest/ug/application-load-balancer-logs.html) tem um guia detalhado de como fazer isso, mas em resumo o que vai precisar fazer:
1. criar uma tabela no Athena apontando pro bucket s3 dos logs
2. Fazer as consultas!

E dai a busca pelas requisições que retornaram código http 500 fica assim:
```bash
SELECT * FROM alb_logs
WHERE elb_status_code >= 500
```

Um ponto de atenção aqui é: dependendo da quantidade de logs que tiver as consultas podem ficar muito lentas.

### Com duckDB

A outra opção é usar o duckDB (que descobri recentemente graças a alguma thread aleatória no twitter)







