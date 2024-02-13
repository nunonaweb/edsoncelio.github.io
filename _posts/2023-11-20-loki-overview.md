---
title: Conhecendo a stack de observabilidade opensource da Grafana Labs - Loki
author: edson
date: 2023-11-20
categories: [Observability]
tags: [grafana, observability, ptbr]
---

Seguindo a última postagem sobre o Grafana, dessa vez vou falar um pouco sobre o Grafana Loki.

## Mas o que é o Grafana Loki?

O Grafana Loki é um sistema de agregação de logs multi-tenant, escalável horizontalmente e com HA inspirado no Prometheus.
Foi projetado para ser otimizado para custos e fácil de manter. Diferente de outros sistemas, não indexa o conteúdo dos logs, mas sim um conjunto de labels para cada log stream.

## Como o Grafana Loki funciona?
Estrutura do Loki:
<p align="center" width="100%">
    <img width="90%" src="https://grafana.com/static/img/logs/logs-loki-diagram.svg">
</p>
Onde,

* Os logs são coletados pelo Promtail.
* Os logs coletados são armazenados no Loki.
* Os logs coletados podem ser analisados com LogQL.
* Os logs armazenados podem gerar alertas (com AlertManager por exemplo) e/ou dashboards no Grafana


Estrutura de indexação dos logs:
<p align="center" width="100%">
    <img width="90%" src="https://grafana.com/static/img/logs/loki-tabs-with-console.svg">
</p>

Sendo que, apenas o timestamp e as labels são indexadas, garantindo menos consumo de armazenamento.


