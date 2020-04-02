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
- Loadbalancer http/https, 1 até 5 regras de forwarding custam aproximadamente `18 USD`.

Bancos de dados, buckets e outros serviços gerenciados do google não entram na conta pois não foram alterados e seus custos continuaram os mesmos.

## Preparando os containers

Como dito no primeiro tópico, as aplicações já rodavam containerizadas. Um container dedicado a aplicação web, um para as queues, um para a execução de schedules, um container redis e um nginx. O container certbot foi descartado pois foi adotada outra abordagem para gerenciamento dos certificados SSL.

Caso você ainda não tenha containerizado sua aplicação, prepare-a de modo que respeite o [Twelve-Factor App](https://12factor.net).

## Preparando os manifestos

Aqui vem o primeiro baque pra quem era acostumado a subir o ambiente de produção inteiro com um único arquivo
docker-compose.yaml 🙃

<p align="center">
  <img src="/k8s-manifests.png" width="400px">
</p>

Mostrarei o propósito de cada arquivo. Veja detalhes e conceitos do Kubernetes em sua [documentação](https://kubernetes.io/docs/concepts/).

A infraestrutura da aplicação é definida como código, os controllers do Kubernetes checam em loop `se o estado atual da aplicação é igual ao estado definido via código`, e caso não seja, aplica as modificações necessárias.

Os manifestos podem ser definidos em YAML ou JSON, as extensões `.yaml`, `.yml` e `.json` são aceitas. Coloco todos no mesmo diretório para facilitar o deploy com o comando `kubectl apply -f k8s/`.

###### 01-namespace.yaml

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: yourapp1
  labels:
    name: yourapp1

---
apiVersion: v1
kind: ResourceQuota
metadata:
  name: resource-quota
  namespace: yourapp1
spec:
  hard:
    requests.cpu: 100m
    requests.memory: 512Mi
    limits.cpu: 200m
    limits.memory: 1024Mi
```

Neste arquivo defino o [Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/) da aplicação. Com namespaces é possível definir o escopo das aplicações. Assim é possível executar várias aplicações diferentes no mesmo cluster sem que interfiram uma na outra (a comunicação entre namespaces ainda é possível através de serviços expostos como mostrarei). Outro exemplo da utilidade de namespaces é separar ambientes de `staging` e `production`. Por padrão, caso
namespaces não sejam definidos, os deploys são realizados no namespace `default`.

No mesmo arquivo defino um deploy do tipo [Resource Quota](https://kubernetes.io/docs/concepts/policy/resource-quotas/), nele é possível definir os recursos e limites de recursos solicitados pelo namespace. No exemplo, defino que:

- `requests.cpu: 100m` - todos os componentes do namespace podem requisitar (somados) no máximo 100 millicores de cpu (1vCPU = 1000m, valores de cpu podem ser definidos de 1m a 1000m).

- `requests.memory: 512Mi` - todos os componentes do namespace podem requisitar (somados) no máximo 512Mi de memória (1 Mebibyte (MiB) = (1024)^2 bytes = 1048576 bytes).

- `limits.cpu: 200m` - todos os componentes do namespace (apesar de requisitar 100m de cpu definidos anteriormente)
  podem utilizar o máximo 200 millicores de cpu.

- `limits.memory: 1024Mi` - todos os componentes do namespace (apesar de requisitar 512Mi de memória definidos anteriormente) podem utilizar no máximo 1024Mi de memória.

Quando os limites de cpu definidos são atingidos, a aplicação começa a sofrer `throttled` de cpu, ou seja, sua performance é afetada.

Quando os limites de memória são atingidos, não é possível "comprimir" a memória como é feito com cpu, e seu [Pod](https://kubernetes.io/docs/concepts/workloads/pods/pod-overview/) é terminado.

Quando um deploy de uma nova versão da sua aplicação é feita, caso o limite de cpu ou memória seja excedido, os pods não seram executados e ficarão com o estado [Evicted](https://kubernetes.io/docs/tasks/administer-cluster/out-of-resource/#eviction-policy).

A definição de ResourceQuota para um namespace é opcional, porém garante que uma aplicação não consuma recursos demasiadamente.

###### 02-nfs-server-deployment.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nfs-server
  namespace: yourapp1
spec:
  replicas: 1
  selector:
    matchLabels:
      name: nfs-server
  template:
    metadata:
      labels:
        name: nfs-server
    spec:
      containers:
        - name: nfs-server
          image: gcr.io/google_containers/volume-nfs:latest
          ports:
            - name: nfs
              containerPort: 2049
            - name: mountd
              containerPort: 20048
            - name: rpcbind
              containerPort: 111
          securityContext:
            privileged: true
          resources:
            requests:
              cpu: 1m
              memory: 168Mi
            limits:
              cpu: 2m
              memory: 192Mi
          volumeMounts:
            - name: data
              mountPath: /exports

      volumes:
        - name: data
          gcePersistentDisk:
            pdName: yourapp1-nfs-disk
            fsType: ext4
```

Neste arquivo defino o deployment de um volume NFS, já que o padrão [GCEPersistentDisk](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes) do GKE não suporta o tipo de acesso `ReadWriteMany` para ser lido e escrito por vários nodes ao mesmo tempo.

Este volume será usado para persistir os dados do redis e também as páginas estáticas que são geradas a partir do container app e compartilhadas com o container nginx.

Alguns detalhes do arquivo:

- O namespace deve ser o mesmo definido anteriormente para que o escopo do deploy seja o mesmo namespace.
- Ele possui apenas uma réplica.
- As requests e limits de cpu desse deployment podem ser bem pequenas (mostro mais a frente como analisar).
- O volume é montado no path /exports do disco.
- O disco definido em `pdName` nos volumes deve ser criado anteriormente com:

```bash
gcloud compute disks create --size=1GB --zone=us-central1-a --type=pd-ssd yourapp1-nfs-disk
```

###### 03-nfs-server-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nfs-server
  namespace: yourapp1
spec:
  ports:
    - name: nfs
      port: 2049
    - name: mountd
      port: 20048
    - name: rpcbind
      port: 111
  selector:
    name: nfs-server
```

O service acima é o responsável por [expor o deployment criado anteriormente para ser acessado pelos outros pods](https://kubernetes.io/docs/concepts/services-networking/service/). O serviço é exposto com ClusterIp, apenas para outros pods, e não para a internet.

Lembre-se de definir o `selector` do service com o mesmo valor definido nas `labels` do selector no deployment.

###### 04-redis-statefulset.yaml

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
  namespace: yourapp1
spec:
  serviceName: redis
  selector:
    matchLabels:
      name: redis
  template:
    metadata:
      name: redis
      labels:
        name: redis
    spec:
      containers:
        - name: redis
          image: redis:5.0.5-alpine
          ports:
            - containerPort: 6379
          resources:
            requests:
              cpu: 5m
              memory: 14Mi
            limits:
              cpu: 5m
              memory: 16Mi
          volumeMounts:
            - name: data
              mountPath: /data
      volumes:
        - name: data
          nfs:
            server: nfs-server.yourapp1.svc.cluster.local
            path: "/redis/data"
```

O deploy do redis é do tipo [StatefulSet](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/), o volume pode acessar diretamente o serviço deployado anteriormente via `<service>`.`<namespace>`.svc.cluster.local.

###### 05-redis-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
  namespace: yourapp1
spec:
  ports:
    - port: 6379
      protocol: TCP
  selector:
    name: redis
```

Expõe o redis com ClusterIp para ser acessado pelos outros pods.

###### 06-app-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
  namespace: yourapp1
  labels:
    name: app
  annotations:
    secret.reloader.stakater.com/reload: "env"
spec:
  replicas: 2
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      name: app
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 50%
  template:
    metadata:
      labels:
        name: app
    spec:
      containers:
        - name: app
          image: gcr.io/yourproject/yourapp1:TAG_NAME
          command: ["/bin/bash"]
          args:
            - -c
            - |
              sleep 12
              php artisan migrate --force
              php artisan optimize
              php artisan view:cache
              ln -s public html
              ln -s /var/www /usr/share/nginx
              /usr/local/sbin/php-fpm
          envFrom:
            - secretRef:
                name: env
          readinessProbe:
            initialDelaySeconds: 20
            periodSeconds: 10
            timeoutSeconds: 5
            failureThreshold: 3
            successThreshold: 1
            tcpSocket:
              port: 9000
          ports:
            - containerPort: 9000
          resources:
            requests:
              cpu: 20m
              memory: 320Mi
            limits:
              cpu: 50m
              memory: 512Mi
          volumeMounts:
            - name: static
              mountPath: /static
            - name: cache-html
              mountPath: /var/www/public/cache-html
          lifecycle:
            postStart:
              exec:
                command: ["/bin/bash", "-c", "cp -r /var/www/public/. /static"]

        - name: cloudsql-proxy
          image: gcr.io/cloudsql-docker/gce-proxy:latest
          command:
            [
              "/cloud_sql_proxy",
              "-instances=yourproject:cloudsql-region:yourproject=tcp:5432",
              "-credential_file=/secrets/cloudsql/cloudsqlproxy.json",
            ]
          resources:
            requests:
              cpu: 1m
              memory: 8Mi
            limits:
              cpu: 10m
              memory: 16Mi
          volumeMounts:
            - name: cloudsql-instance-credentials
              mountPath: /secrets/cloudsql
              readOnly: true

      volumes:
        - name: static
          nfs:
            server: nfs-server.yourapp1.svc.cluster.local
            path: "/static"
        - name: cache-html
          nfs:
            server: nfs-server.yourapp1.svc.cluster.local
            path: "/cache-html"
        - name: cloudsql-instance-credentials
          secret:
            secretName: cloudsql-instance-credentials
```

Responsável pelo deploy da aplicação laravel com fpm, em `annotations` temos uma anotação `secret.reloader.stakater.com/reload: "env"` que irá realizar o deploy de um novo pod `sempre que a secret com nome env` for atualizada. Esse comportamento não é padrão do kubernetes e para habilitá-lo, instalamos um controller chamado [Reloader](https://github.com/stakater/Reloader) com o comando:

```bash
kubectl apply -f https://raw.githubusercontent.com/stakater/Reloader/master/deployments/kubernetes/reloader.yaml
```

- `replicas: 2` - definimos que o deploy irá criar 2 pods
- `revisionHistoryLimit: 1` - só teremos acesso a uma versão anterior a atual para rollback

Em `strategy`:

- `type: RollingUpdate` - Nosso deploy é do tipo [Rolling Update](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/#rolling-update-deployment)
- `maxSurge: 1` - Só pode surgir um novo pod por vez
- `maxUnavailable: 50%` - No mínimo metade dos pods devem estar disponíveis durante o deploy, ou seja, no exemplo com `replicas: 2` e `maxSurge: 1`, um novo pod surgirá, então um pod antigo será terminado (respeitando o `maxUnavailable: 50%`), então um novo pod surgirá, e o último pod antigo será terminado.

Em `containers`:

Esse pod possui 2 containers, o app principal e um sidecar (um proxy para conexão com o banco de dados no Google Cloud SQL).

No primeiro container `app` temos:

- `commands:` e `args:` - são os comandos que serão executados pelo nosso container, o `sleep` inicial serve para aguardar até que o container sidecar esteja disponível. Na versão `1.18` em diante [esse sleep não será mais necessário](https://banzaicloud.com/blog/k8s-sidecars/).
- `envFrom:` - injetamos nossas variáveis de ambiente (mostrarei como criá-las no tópico [Automatizando o processo de teste e deploy com um pipeline CI/CD](#automatizando-o-processo-de-testes-e-deploy-com-um-pipeline-de-cicd)).
- `readinessProbe:` - O container só receberá tráfego após uma conexão TCP bem sucedida com o container na porta 9000.
- `volumeMounts:` - O volume static compartilha assets entre o app e o nginx. O volume cache-html armazena algumas páginas de forma estática geradas a partir do conteúdo dinâmico, evitando a renderização a todo momento.

No sidecar container `cloudsql-proxy` fazemos a autenticação com o uso de um secret que também mostro como criá-la no tópico [Automatizando o processo de teste e deploy com um pipeline CI/CD](#automatizando-o-processo-de-testes-e-deploy-com-um-pipeline-de-cicd).

###### 07-app-hpa.yaml

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: app
  namespace: yourapp1
spec:
  maxReplicas: 3
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  metrics:
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: 90
    - type: Resource
      resource:
        name: memory
        targetAverageUtilization: 90
```

O [Horizontal Pod Autoscaler (HPA)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/) escala automaticamente o número de pods do nosso deployment. No exemplo em 06-app-deployment.yaml, nosso deployment foi feito com duas réplicas, o HPA acima consegue irá escalar esse deployment para 1 ou 3 réplicas baseado na média de utilização de cpu e memória do deployment.

###### 08-app-service.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
  name: app
  namespace: yourapp1
  labels:
    name: app
spec:
  ports:
    - protocol: TCP
      port: 9000
  selector:
    name: app
```

Expõe o app com ClusterIp para ser acessado pelos outros pods.

###### 09-queue-deployment.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: queue
  namespace: yourapp1
  labels:
    name: queue
  annotations:
    secret.reloader.stakater.com/reload: "env"
spec:
  replicas: 1
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      name: queue
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 50%
  template:
    metadata:
      labels:
        name: queue
    spec:
      containers:
        - name: queue
          image: gcr.io/yourproject/yourapp1:TAG_NAME
          command: ["/bin/bash"]
          args:
            - -c
            - |
              php artisan config:cache
              php artisan horizon --quiet
          envFrom:
            - secretRef:
                name: env
          resources:
            requests:
              cpu: 100m
              memory: 128Mi
            limits:
              cpu: 150m
              memory: 512Mi

        - name: cloudsql-proxy
          image: gcr.io/cloudsql-docker/gce-proxy:latest
          command:
            [
              "/cloud_sql_proxy",
              "-instances=yourproject:cloudsql-region:yourproject=tcp:5432",
              "-credential_file=/secrets/cloudsql/cloudsqlproxy.json",
            ]
          resources:
            requests:
              cpu: 1m
              memory: 8Mi
            limits:
              cpu: 10m
              memory: 16Mi
          volumeMounts:
            - name: cloudsql-instance-credentials
              mountPath: /secrets/cloudsql
              readOnly: true

      volumes:
        - name: cloudsql-instance-credentials
          secret:
            secretName: cloudsql-instance-credentials
```

Os conceitos são os mesmos do [06-app-deployment.yaml](#06-app-deploymentyaml), porém iniciamos com apenas uma réplica e o HPA a seguir controla a necessidade de outras réplicas.

###### 10-queue-hpa.yaml

```yaml
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: queue
  namespace: yourapp1
spec:
  maxReplicas: 2
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: queue
  metrics:
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: 100
    - type: Resource
      resource:
        name: memory
        targetAverageUtilization: 90
```

Os conceitos são os mesmos do [07-app-hpa.yaml](#07-app-hpayaml)

## Criando o cluster Kubernetes

## Realizando o deploy dos manifestos

## Adicionando certificados SSL auto gerenciados

## Automatizando o processo de testes e deploy com um pipeline de CI/CD

## Monitorando o cluster com Kontena Lens e métricas Prometheus

## Problemas identificados

## Próximos passos
