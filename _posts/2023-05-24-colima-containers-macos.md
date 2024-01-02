---
title: Executando (e orquestrando) containers no MacOS com Lima e Colima
author: edson
date: 2023-05-24
categories: [Cloud Native]
tags: [containers, colima, lima, ptbr]
---

Usar docker no macOS pode ser um problema, dado que o Docker Desktop é um pouco agressivo com o consumo de recursos da máquina, daí que surgem algumas alternativas, usar o Lima é uma delas, e é disso que vou falar um pouco.

## O projeto Lima

O projeto [lima-vm](https://github.com/lima-vm/lima) (Linux Virtual Machines) usa máquinas virtuais Linux para fazer compartilhamento automático de arquivo e encaminhamento de portas (parecido com o WSL2) com containerd.

Algumas características que valem a pena destacar:

* Além do suporte nativo ao containerd suporta outras engine de containers, como docker, podman, apptainer
    
* Suporte a orquestradores de container como kubernetes (com k3s e via kubeadm), faasd e nomad
    
* Suporte a uma boa quantidade de distribuições linux, como Alpine, Debian, Fedora, openSUSE, Arch Linux e por aí vai
    

Mais detalhes vocês podem conferir no repositório do Github (que super recomendo a dar uma olhada, tem bastante coisa).

### Hora de usar

A instalação no macOS pode ser feita com homebrew:

```plaintext
$ brew install lima
```

Depois disso é só usar a CLI (`limactl`) para iniciar uma instância:

```plaintext
$ limactl start
```

As configurações da instância padrão (`default`) são:

* Sistema operacional: Ubuntu 22.10 (Kinetic Kudu)
    
* CPU: 4 cores
    
* Memória: 4 GiB
    
* Disco: 100 GiB
    
* Montagens: ~ (somente leitura), /tmp/lima (escrita)
    
* SSH: 127.0.0.1:60022
    

Você pode iniciar uma instância a partir de um template, para ver os templates disponíveis:

```plaintext
$ limactl start --list-templates
```

A saída vai ser algo assim:

```plaintext
$ almalinux-8
almalinux-9
almalinux
alpine
apptainer-rootful
apptainer
archlinux
buildkit
centos-stream-8
....
```

Iniciando a instância a partir de um template:

```plaintext
$ limactl start --name=alpine-instance template://alpine
```

Você pode listar as instâncias criadas/iniciadas com:

```plaintext
$ limactl list
```

Para acessar a instância, você pode usar o comando `limactl shell`, de duas formas:

* abrindo um shell: `limactl shell <nome-da-instancia>`
    
* executando um comando: `limactl shell <nome-da-instancia> <comando>`
    

Se quiser para a instância, só usar:

```plaintext
$ limactl stop <nome-da-instancia>
```

E caso queira deletar:

```plaintext
$ limactl delete <nome-da-instancia>
```

Existem alguns outros comandos, podem consultar a lista [aqui](https://github.com/lima-vm/lima#command-reference).

## O projeto Colima

A ideia do projeto [colima](https://github.com/abiosoft/colima) é criar um ambiente de de container no macOS com uma configuração mínima usando o Lima.

### Hora de usar

A instalação também pode ser feita via homebrew:

```plaintext
$ brew install colima
```

Para iniciar com as configurações padrões:

```plaintext
$ colima start
```

Por padrão, o container runtime é Docker, então se quiser usar outro:

```plaintext
$ colima start --runtime containerd
```

As configurações padrões da VM criada pelo colima são:

* 2 CPUs
    
* 2Gib de memória
    
* goGiB de disco
    

Isso é totalmente customizável com as flags `--cpu`, `--memory`, `--disk` ou alterando no arquivo de configuração, com `--edit`.

Agora a parte MAIS LEGAL, se quiser usar kubernetes, é só executar:

```plaintext
$ colima start --runtime containerd --kubernetes
```

E pronto, seu cluster já está criado:

```plaintext
$ k get nodes
$ NAME     STATUS   ROLES                  AGE     VERSION
colima   Ready    control-plane,master   7d12h   v1.25.4+k3s1
```

Mais detalhes dos comandos vocês podem ver no [repositório do projeto no Github](https://github.com/abiosoft/colima).

Por agora é isso, espero que seja útil pra mais alguém (assim como foi pra mim).