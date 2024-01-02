---
title: O que muda no Karpenter a partir das versões 0.32.x?
author: edson
date: 2023-11-12
categories: [Cloud Native]
tags: [cloudnative, karpenter, ptbr]
---


Com o objetivo de se preparar para uma versão v1, o Karpenter trouxe mudanças significativas nas versões 0.32.x que devem ser levadas em conta caso queira fazer *upgrade*.

Nesse blog post vou fazer um resumo das principais mudanças das APIs `v1alpha5` para `v1beta1` , que foi introduzida a partir das versões 0.32.x .

### O que era `Provisioner` agora é `NodePool`

Antes:

```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
...
```

Depois:

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
...
```

Além das seguintes mudanças no manifesto:

* Os campos `Limits` e `Weight` ficam fora da seção `spec.template`
    
* Os campos `Labels` e `Annotations` ficam dentro da seção `spec.template.metadata`
    
* Todos os outros campos como *requirements*, *taints*, *kubelet* e outros ficam dentro da seção `spec.template.spec`
    
* Não existe mais suporte para `spec.template.spec.kubelet.containerRuntime` . Se você usa EKS 1.23 deve fazer upgrade pra usar containerd antes de atualizar
    

#### Importante:

O campo `spec.ttlSecondsAfterEmpty` foi removido em favor dos campos `consolidationPolicy` e `consolidateAfter` .

Antes:

```yaml
apiVersion: karpenter.sh/v1alpha5
kind: Provisioner
...
spec:
  ttlSecondsAfterEmpty: 120
```

Depois:

```yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
...
spec:
  disruption:
    consolidationPolicy: WhenEmpty
    consolidateAfter: 2m
```

### `AWSNodeTemplate` agora é `EC2NodeClass`

Antes:

```yaml
apiVersion: karpenter.k8s.aws/v1alpha1
kind: AWSNodeTemplate
...
```

Depois:

```yaml
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
...
```

Além das seguintes mudanças no manifesto:

* O campo `spec.instanceProfile` foi removido em favor do `spec.role` . Além disso foi removido o suporte para o campo `defaultInstanceProfile`, o que torna o campo `spec.role` obrigatório
    
* Os campos `spec.subnetSelector`, `spec.securityGroupSelector` e `spec.amiSelector` foram modificados para suportar múltiplos termos e chaves como *id* e *name*
    

### Outras informações relevantes

* `Machine` agora é `NodeClaim`
    
* [Mudanças em algumas *labels* e *annotations* e *status conditions*](https://karpenter.sh/docs/upgrading/v1beta1-migration/#annotations-labels-and-status-conditions)
    
* [Mudanças na nomeclatura das métricas](https://karpenter.sh/docs/upgrading/v1beta1-migration/#metrics)
    
* [Mudanças nos valores do *helm chart*](https://karpenter.sh/docs/upgrading/v1beta1-migration/#helm-values)
    

---

Para ajudar no processo de atualização, foi disponibilizada uma ferramenta para conversão dos manifestos.

Para instalar:

```bash
go install github.com/aws/karpenter/tools/karpenter-convert/cmd/karpenter-convert@latest
```

O repositório da ferramenta pode ser acessado [aqui](https://github.com/aws/karpenter/tree/main/tools/karpenter-convert).

### Referências

Todas as informações aqui citadas, foram tiradas da documentação de *upgrade* que pode ser consultada [aqui](https://karpenter.sh/docs/upgrading/v1beta1-migration/).