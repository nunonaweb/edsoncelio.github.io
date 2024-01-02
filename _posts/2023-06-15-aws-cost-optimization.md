---
title: Dicas para otimização de custos na AWS
author: edson
date: 2023-06-15
categories: [AWS]
tags: [aws, ptbr]
---


Antes de tudo, é importante dizer que a AWS tem algumas soluções (além das dashboards padrões) que ajudam a entender em detalhes os custos e servem como guia para as melhorias e para a criação de uma cultura de FinOps.

Por exemplo, com uma combinação de [CUDOS com Amazon QuickSight](https://aws.amazon.com/pt/blogs/mt/visualize-and-gain-insights-into-your-aws-cost-and-usage-with-cloud-intelligence-dashboards-using-amazon-quicksight/) é possível ter uma visão do tipo:

<p align="center" width="100%">
    <img width="100%" src="https://cdn.hashnode.com/res/hashnode/image/upload/v1688262763466/6cbe4a82-3d31-4bc2-aae0-01212c2fdc95.png">
</p>

Link com informações para configuração: [https://aws.amazon.com/pt/blogs/mt/visualize-and-gain-insights-into-your-aws-cost-and-usage-with-cloud-intelligence-dashboards-using-amazon-quicksight/](https://aws.amazon.com/pt/blogs/mt/visualize-and-gain-insights-into-your-aws-cost-and-usage-with-cloud-intelligence-dashboards-using-amazon-quicksight/)

---

E Agora indo para o dicas - separei por categorias para melhor entendimento, com o tempo vou atualizando com mais dicas e/ou novas categorias.

### Storage

* Verifique sempre por volumes EBS que não estão atachados a instâncias e se realmente são necessários, se for o caso, pode usar a *flag* **DeleteOnTermination** para remover os volumes sempre que a instância for terminada ;
    
* Verifique se para o seu cenário, é possível mudar o tipo dos volumes EBS de uso geral de gp2 para gp3 (pode ter até 20% em redução do custo);
    
* Faça gestão do ciclo de vida dos objetos no S3, com [*lifecycle rules*](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html) consegue transicionar os objetos para um *storage class* mais barato, com o [*Storage Lens*](https://aws.amazon.com/pt/blogs/aws/s3-storage-lens/) consegue ter uma visão geral dos objetos;
    

### Compute

* Para ambientes não produtivos (desenvolvimento/testes), configure políticas/automações para desligamento em horário não comercial;
    
* Entenda seu *workload* (parece fácil falando, mas não é) - use métricas para saber qual família de instâncias atende melhor cada ambiente, uma ferramenta que pode ajudar: [https://github.com/aws/amazon-ec2-instance-selector](https://github.com/aws/amazon-ec2-instance-selector)
    
* Use autoscalling para fornecer recursos sob demanda, se usa EKS - use [Karpenter](https://karpenter.sh/) para cluster autoscaling (tem [esse artigo](https://medium.com/@fidelissauro/provisionando-um-cluster-de-eks-sem-node-groups-com-karpenter-4d302b32b620) muito bom sobre o assunto escrito pelo Fidelis!)
    
* Para ambientes/aplicações que conseguem lidar com interrupções, instâncias Spot são uma ótima escolha (se usar Karpenter consegue lidar com essas interrupções usando a funcionalidade de [*Spot Termination Handler*](https://aws.github.io/aws-eks-best-practices/karpenter/#enable-interruption-handling-when-using-spot));
    
* Se aplicável, usar instâncias EC2 baseadas em processadores Graviton/Graviton2 (arm64), onde vai ter um melhor mix entre performance e preço;
    
* Para instâncias *on-demand*, verifique as recomendações de *Saving Plans* e *Reservations;*
    

### Outras dicas e recomendações de leitura

* [https://aws.amazon.com/pt/blogs/mt/visualize-and-gain-insights-into-your-aws-cost-and-usage-with-cloud-intelligence-dashboards-using-amazon-quicksight/](https://aws.amazon.com/pt/blogs/mt/visualize-and-gain-insights-into-your-aws-cost-and-usage-with-cloud-intelligence-dashboards-using-amazon-quicksight/)
    
* Algumas ferramentas de CLI que podem ajudar:
    
    * [https://github.com/awslabs/eks-node-viewer](https://github.com/awslabs/eks-node-viewer) (visualização de custos em tempo real de clusters EKS)
        
    * [https://instances.vantage.sh/](https://instances.vantage.sh/) (comparativo entre tipos de instâncias EC2)
        
* Use e abuse de automações e/ou ferramentas para fazer *enforcement* de boas práticas de custos
    
* [https://wellarchitectedlabs.com/cost/200\_labs/200\_cloud\_intelligence/](https://wellarchitectedlabs.com/cost/200_labs/200_cloud_intelligence/)
    
* O material do pilar de *Cost Optimization* do AWS *Well-Architected Framework*: [https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html](https://docs.aws.amazon.com/wellarchitected/latest/cost-optimization-pillar/welcome.html)
    
* Leia/conheça sobre FinOps, vai ajudar bastante a desenhar estratégias de otimização de custos: [https://www.finops.org/introduction/what-is-finops/](https://www.finops.org/introduction/what-is-finops/)