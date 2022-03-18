# Advanced topis

## Install Kubernetes on Production

1. Ver [Kubernetes the hard way](https://github.com/kelseyhightower/kubernetes-the-hard-way)

2. kubeadm (mais utilizados)

    - Instalar uma rede de pods: ver Flannel e Weave Net

4. kops - Para infra AWS 

5. Managed Kubernetes - terceirizar a configuração

## Namespaces

Comandos úteis

- `kubectl get namespaces`

- `kubectl create namespace <namespace name>`

- `kubectl delete namespace <namespace name>`

- Para o deploy de um recurso em um namespace específico, adicionar a flag `-n <namespace name>`

## Logging e Monitoramento

- Em produção, há a possibilidade de utilizar uma plataforma de logs, como o Kibana com Elasticsearch (Fluentd ou Filebeat(Logstash) para enviar os logs para a plataforma)

- Para monitoramento da saúde do nodo e do Kubernetes: **cAdvisor**, **Prometheus** ou **Datadog**

- cAdvisor, Prometheus ou Datadog podem enviar os dados para o **Grafana**, responsável pela visualização dos dados

## Autenticação e Autorização

Métodos populares de autenticação:

- Client certs, Static token files, OpenId Connect ou Webhook Tokens

Métodos populares de autorização:

- ABAC (Attribute-based access control), RBAC (Role-based access control, método mais comum) ou Webhook

