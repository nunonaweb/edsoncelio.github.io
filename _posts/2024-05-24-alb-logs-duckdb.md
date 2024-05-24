---
title: Analisando logs de acesso de ALBs (Application Load Balancer) localmente com DuckDB
author: edson
date: 2024-0-12
categories: [Utils]
tags: [aws, logs, ptbr]
---

Só quem já precisou fazer alguma investigação que envolvia analisar os logs de acesso de ALBs (Application Load Balancer) da AWS sabe a dor e sofrimento que é.   
Mas pra quem nunca precisou? Não tem problema! Vou explicar um pouco sobre como funciona até chegar na solução com uso do DuckDB.

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

### AWS Athena
Uma forma de fazer busca e consulta nesses arquivos de logs, é usando o AWS Athena, que pra quem não conhece é um serviço da AWS que permite que você faça consulta usando SQL em arquivos do s3.

[Aqui](https://docs.aws.amazon.com/athena/latest/ug/application-load-balancer-logs.html) tem um guia detalhado de como fazer isso, mas em resumo o que vai precisar fazer:
1. criar uma tabela no Athena apontando pro bucket s3 dos logs
2. Fazer as consultas!

E dai a busca pelas requisições que retornaram código http 500 fica assim:
```bash
SELECT * FROM alb_logs
WHERE elb_status_code >= 500
```

Pontos de atenção aqui:
* dependendo da quantidade de logs que tiver as consultas podem ficar muito lentas.
* custos atrelados as consultas do Athena.
* vai executar tudo na console do Athena na AWS.

## DuckDB

A outra opção é usar o [DuckDB](https://duckdb.org/why_duckdb) (que descobri recentemente graças a alguma thread aleatória no twitter).    
O DuckDB é um sistema de gerenciamento de banco de dados relacional que suporta SQL, mas o que torna diferente de outras soluções?
 
Pro meu caso o diferencial foi, além da simplicidade (apenas um binário sem dependências) foi a funcionalidade de fazer consultas em SQL a partir de fontes como json, csv, parquet localmente e também de fontes remotas (como o s3!).

**Obs.:** uma funcionalidade que quero experimentar em breve é usar as bibliotecas clientes (Python, Golang...).

Para o cenário dos logs do ALB, vamos ler diretamente do s3, convertendo pra csv pra fazer consultas em SQL localmente!

Finalmente vamos pro que interessa, a configuração para conseguir fazer isso.

### Instalação e Autenticação na AWS

Como já citei, a instalação é apenas fazer o download e usar um binário, [aqui](https://duckdb.org/docs/installation/index?version=stable&environment=cli&platform=linux&download_method=package_manager) na página de instalação você pode ver mais detalhes.

Depois de instalado pode usar a CLI `duckdb`.

Para autenticação, vamos configurar um [provider](https://duckdb.org/docs/extensions/httpfs/s3api#config-provider):

```
CREATE SECRET s3credentials (
    TYPE S3,
    PROVIDER CREDENTIAL_CHAIN
);
```
Também poderia ser feito passando diretamente as credenciais, mas não queremos fazer isso.

Criado esse bloco de provider, já podemos acessar os buckets s3 que essas credenciais tem acesso, no formato:
```
s3://<path-no-bucket-s3>
```

### Lendo os arquivos de logs e fazendo o parse para SQL

Já estando autenticado, podemos ler o arquivo ou diretório que queremos, converter pra CSV, salvar em uma tabela e dai sim começar a consultar as informações.

Essa é a estrutura da tabela que vamos criar:
```
CREATE TABLE alb_logs AS SELECT * FROM 
read_csv('s3://<path-no-bucket-s3>', delim=' ',
types = {
 'type': 'VARCHAR',
 'time': 'TIMESTAMP',
 'elb': 'VARCHAR',
 'client:port': 'VARCHAR',
 'target:port': 'VARCHAR',
 'request_processing_time': 'FLOAT',
 'target_processing_time': 'FLOAT',
 'response_processing_time': 'FLOAT',
 'elb_status_code': 'INTEGER',
 'target_status_code': 'VARCHAR',
 'received_bytes': 'BIGINT',
 'sent_bytes': 'BIGINT',
 'request': 'VARCHAR',
 'user_agent': 'VARCHAR',
 'ssl_cipher': 'VARCHAR',
 'ssl_protocol': 'VARCHAR',
 'target_group_arn': 'VARCHAR',
 'trace_id': 'VARCHAR',
 'domain_name': 'VARCHAR',
 'chosen_cert_arn': 'VARCHAR',
 'matched_rule_priority': 'VARCHAR',
 'request_creation_time': 'TIMESTAMP',
 'actions_executed': 'VARCHAR',
 'redirect_url': 'VARCHAR',
 'error_reason': 'VARCHAR',
 'target_status_code_list': 'VARCHAR',
 'classification': 'VARCHAR',
 'classification_reason': 'VARCHAR'
},
names=['type', 'time','elb','client:port','target:port','request_processing_time','target_processing_time','response_processing_time','elb_status_code','target_status_code','received_bytes','sent_bytes','request','user_agent','ssl_cipher','ssl_protocol','target_group_arn','trace_id','domain_name','chosen_cert_arn','matched_rule_priority','request_creation_time','actions_executed','redirect_url','error_reason','target_status_code_list','classification','classification_reason'],
auto_detect = true);
```
Onde usamos a função `read_csv` pra ler o arquivo .log.gz (isso mesmo, nem precisamos descompactar nem nada!) e fazer a conversão dos campos para uma tabela SQL.

Com isso já podemos executar comandos normalmente:

```
SELECT * FROM alb_logs WHERE elb_status_code == 500;
```

Algumas outras coisas que valem citar:
* O path do arquivo no bucket s3 suporta wildcards, então daria pra ter algo como `*.log.gz` e ler todos os arquivos de um diretório.
* Da forma que estou fazendo tudo fica em memória, então vale ficar atento a quantidade de logs que quer buscar.
* Se quiser gerar algo mais automatizado, pode criar um script `.sql` e executar com o *init*, assim: `duckdb -init read_alb_logs.sql`, dessa forma já vai abrir o shell com as informações prontas para serem consultadas.
* o DuckDB suporta alguns comandos com ponto (`.`): 
  * `.read`: ler um arquivo de script.
  * `.output`: salvar a saída em um arquivo ao invés de stdout. 
  * e muitos outros, pode ver [aqui](https://duckdb.org/docs/api/cli/overview#dot-commands).

Essas notas foram escritas no final de uma sexta-feira então desculpem se deixei passar algo, o script completo [está aqui no gist](https://gist.github.com/edsoncelio/93ce840c923677a7563f3462fdda77ed).


## Referências

Alguns dos links que me ajudaram:
* https://www.linkedin.com/pulse/analysing-aws-application-load-balancer-logs-duckdb-unleashing/
* https://duckdb.org/docs/data/csv/overview
* https://duckdb.org/docs/extensions/httpfs/s3api




