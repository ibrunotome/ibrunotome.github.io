---
title: "Múltiplas aplicações em um cluster Kubernetes"
url: multiplas-aplicacoes-em-um-kluster-kubernetes
date: "2020-03-29T13:29:17-03:00"
lastmod: "2020-03-29T13:29:17-03:00"
tags: ["gcp", "laravel", "prometheus"]
categories: ["devops", "kubernetes"]
imgs: ["/../multiple-applications-in-one-kubernetes-cluster.svg"]
cover: "/multiple-applications-in-one-kubernetes-cluster.svg"
readingTime: true
toc: true
comments: true
justify: false
single: false
license: ""
draft: true
translationKey: "multiple-applications-in-one-kubernetes-cluster"
---

Neste post mostro como preparei múltiplas aplicações para deploy em um único cluster kubernetes e também: o motivo da
escolha do kubernetes, os benefícios, as dificuldades enfrentadas e os próximos passos.

<!--more-->

---

Tópicos:

- [O problema inicial](#o-problema-inicial)
- [Por que Kubernetes?](#por-que-kubernetes)
- [Preparando os containers](#preparando-os-containers)
- [Preparando os manifestos](#preparando-os-manifestos)
- [Criando o cluster kubernetes](#criando-o-cluster-kubernetes)
- [Realizando o deploy dos manifestos](#realizando-o-deploy-dos-manifestos)
- [Adicionando certificados SSL auto gerenciados](#adicionando-certificados-ssl-auto-gerenciados)
- [Automatizando o processo de teste e deploy com um pipeline CI/CD](#automatizando-o-processo-de-testes-e-deploy-com-um-pipeline-de-cicd)
- [Monitorando o cluster com Kontena Lens e métricas Prometheus](#monitorando-o-cluster-com-kontena-lens-e-metricas-prometheus)
- [Problemas identificados](#problemas-identificados)
- [Próximos passos](#proximos-passos)

---

## O problema inicial

Temos pelo menos 5 aplicações principais rodando no _[Google Cloud Platform (GCP)](https://cloud.google.com)_ e também
algumas funções que foram desacopladas de uma api e hoje são executadas a partir de _[google functions](https://cloud.google.com/functions)_, acessadas pelas aplicações principais.

Três dessas aplicações principais rodavam individualmente (uma aplicação por VM) em VMs do tipo n1-standard-1 (1vCPU e 3,75GB de RAM). Todas as aplicações containerizadas e o deploy em produção realizado por um simples _docker-compose up -d_

Mesmo com todo o processo de deploy automatizado, sem gerar dores de cabeça, `a má utilização dos recursos` me incomodava:

- Uma das aplicações (painel de administração, pouquíssimas pessoas com acesso) consumia no máximo 10% da CPU somando
  todos os containers (nginx, app, queue, redis, certbot) e no máximo 2 dos 3,75GB de RAM.
- Uma segunda aplicação com métricas bem parecidas com as de cima.
- Uma terceira aplicação, com situação oposta as de cima, com métricas recomendando o upgrade da VM para pelo menos 6GB
  de RAM. Essa aplicação executa jobs em queues, pode ficar alguns minutos sem receber nenhum novo job e não exigir
  recursos de hardware, porém, quando recebe um novo job ela deve executar todos rapidamente, além de poder receber novos
  jobs enquanto executa o antigo e já ter que iniciar a execução desse novo job sem espera. No framework utilizado nessa aplicação (Laravel), cada worker utiliza pelo menos 32MB de RAM, se configurarmos um valor máximo de 120 workers, já
  são necessários pelo menos 3840MB de memória RAM, excedendo os 3,75GB de RAM dessa VM. Além do fato de muitas vezes
  os 120 workers não serem suficientes para uma entrega rápida, ocasionando em um wait time longo para executar os jobs
  nas queues:

  ![Longo tempo de espera para execução dos jobs no horizon](/horizon-queue-long-wait-time.png)

  Essa aplicação definivamente precisava de mais recursos enquanto as outras duas citadas anteriormente não utilizavam
  todos os recursos disponíveis.

- Uma quarta aplicação, um MVP (_Minimum viable product_) rodando em um único container no _[cloud.run](https://cloud.run)_, já estava validada e precisava evoluir com implementação de queues e cache por exemplo. Como o cloud.run é feito para conteúdo _stateless_ e não possui acesso a redis (pelo menos não de forma fácil, sem ter que expor o redis de alguma VM por exemplo), era necessário tirá-lo dali.

- Uma quinta aplicação, também em container único, rodava bem no cloud.run e, diferentemente da anterior não precisa de conexão com queues. Porém, como possui muitos acessos no cloud.run e o tempo de execução de CPU de cada request dessa aplicação é alto, os custos no cloud.run começaram a incomodar (abaixo os preços do cloud.run com e sem free tier):

  ![Cloud Run Pricing](/cloud-run-pricing.png)

Uma solução viável para otimização da utilização de recursos seria executar as aplicações num cluster, possuindo assim o
controle de quanto hardware dedicar a cada aplicação e abrindo possibilidade para escalabilidade da terceira aplicação
mencionada anteriormente. Para orquestrar os contâiners no cluster, dentre as opções disponíveis eu teria que escolher
bem entre duas: Swarm ou Kubernetes, pois possuía um pouco de conhecimento prévio de ambas as ferramentas.

## Por que Kubernetes?

Gerenciar um cluster é difícil, aplicar patches de segurança, auto reparo, auto upgrade, auto scaling e garantir
disponibilidade são só alguns dos exemplos do que teríamos que manter caso optássemos por gerenciá-lo.

Dentre as opções de cluster citadas anteriormente (Swarm e Kubernetes), nosso cloud provider disponibiliza apenas o serviço
de gerenciamento de Kubernetes (na minha opinião, um dos melhores e mais robustos, talvez porque eles projetaram o Kubernetes e a maioria dos seus serviços rodam no mesmo). O serviço é o Google Kubernetes Engine (GKE) e [oferece um plano gratuito para um cluster zonal](https://cloud.google.com/kubernetes-engine#pricing), atendendo nossas necessidades.

Os custos previstos:

- 2 VMs [e2-standard-2](https://cloud.google.com/blog/products/compute/google-compute-engine-gets-new-e2-vm-machine-types) com 2vCPU e 8GB de ram cada, totalizando um cluster com 4vCPU e 16GB de RAM. Com um contrato de [desconto por uso contínuo](https://cloud.google.com/compute/docs/instances/signing-up-committed-use-discounts) de 3 anos, a previsão mensal de custo é de aproximadamente `45 USD`.
- Loadbalancer http/htpts, 1 até 5 regras de forwarding custam aproximadamente `18 USD`.

Bancos de dados, buckets e outros serviços gerenciados do google não entram na conta pois não foram alterados e seus custos continuaram os mesmos.

## Preparando os containers

Como dito no primeiro tópico, as aplicações já rodavam containerizadas. Um container dedicado a aplicação web, um para as queues, um para a execução de schedules, um container redis e um nginx. O container certbot foi descartado pois foi adotada outra abordagem para gerenciamento dos certificados SSL.

Caso você ainda não tenha containerizado sua aplicação, prepare-a de modo que respeite o [Twelve-Factor App](https://12factor.net).

## Preparando os manifestos

## Criando o cluster Kubernetes

## Realizando o deploy dos manifestos

## Adicionando certificados SSL auto gerenciados

## Automatizando o processo de testes e deploy com um pipeline de CI/CD

## Monitorando o cluster com Kontena Lens e métricas Prometheus

## Problemas identificados

## Próximos passos
