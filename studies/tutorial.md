# Usage

## HelloWorld

1. Listar nodos: `kubectl get nodes`

2. Listar todos os recursos: `kubectl get all`

3. Navegar até a pasta `exercices`

4. Criar um recurso que é um arquivo (-f): `kubectl create -f .\03_05\helloworld.yaml`

Agora, se rodarmos o passo 2 novamente, veremos outros recursos, com um pod já rodando. 

5. Para permitir o acesso a esse pod, rodamos: `kubectl expose deployment helloworld --type=NodePort`

Dessa forma, criamos 2 services para conexão com o app

6. Listagem de services: `minikube service list`

7. Copiar a URL e colar no navegador

## Breaking Down the HelloWorld

1. Mostra a estrtura yaml do deployment: `kubectl get deployment/helloworld -o yaml`

2. Mostra a estrutura yaml do service: `kubectl get service/helloworld -o yaml`

Nos dois passos acima, atentar-se a seção `spec`, que contém a maioria das informações úteis

4. `kubectl create -f .\03_05\helloworld-all.yml`

Através de `---` no arquivo `helloworld-all.yaml` é possível setar múltiplas configurações no mesmo arquivo, como deployments e services

## Labels

### Adicionar Labels ao fazer deploy

1. `kubectl create -f .\03_05\helloworld-pod-with-labels.yml`

2. Mostra uma nova coluna chamada Labels, com as labels configuradas no arquivo yaml: `kubectl get pods --show-labels`

### Adicionar Labels enquanto o pod está executando

1. `kubectl label pod/helloworld app=helloworldapp --overwrite`

### Deletar uma Label

1. `kubectl label pod/helloworld app-`

### Working with Labels

1. `kubectl create -f .\04_01\sample-infrastructure-with-labels.yaml`

Procurar Labels:

2. `kubectl get pods --selector env=production --show-labels`

3. `kubectl get pods --selector dev-lead=carisa --show-labels`

4. `kubectl get pods --selector dev-lead=karthik,env=staging --show-labels`

5. `kubectl get pods --selector dev-lead!=karthik,env=staging --show-labels`

`--selector` e `-l` fazem a mesma coisa.

Operadores IN e NOTIN:

6. `kubectl get pods -l 'release-version in (1.0,2.0)'`

7. `kubectl get pods -l 'release-version notin (1.0,2.0)'`

8. Deletar todos os pods associados com um selector: `kubectl delete pods -l dev-lead=karthik`

## Application Health checks

1. `kubectl create -f .\04_03\helloworld-with-bad-readiness-probe.yaml`

Esse pod nunca vai ficar "Ready".

2. `kubectl describe pod/<nome do pod>`

Na seção Events, podemos ver uma mensagem "warning, unhealthy" e a razão.

3. `kubectl create -f .\04_03\helloworld-with-bad-liveness-probe.yaml`

4. Ao rodar `kubectl get pods`, podemos ver que o pod continua restartando, até que uma hora gera o erro `CrashLoopBackOff`

5. `kubectl describe pod/<nome do pod>`

Na seção Events, podemos ver uma mensagem "warning, unhealthy, killing" e a razão.

## Handling Application Upgrade (Not Working very well)

1. `kubectl create -f .\04_04\helloworld-black.yaml --record`

2. `minikube service list --url` e acessar a url do output

A atualização será (seria, por que não funciona) mudar a cor da toptab da página.

3. `kubectl set image deployment/navbar-deployment helloworld=karthequian/helloworld:blue --record=true`

A cor não muda, mas podemos verificar que foi feito uma mudança: `kubectl get rs`.
O comando busca as ReplicaSets, e podemos ver 3 pods rodando para uma ReplicaSet de nome `navbar-deployment-<hash1>` e 0 pods para `navbar-deployment-<hash2>`.

4. A partir deste comando podemos ver o histórico da deploy: `kubectl rollout history deployments/navbar-deployment`

5. Fazemos um Rollback: `kubectl rollout undo deployments/navbar-deployment`

6. Ao digitar `kubectl get rs`, podemos verificar que a RS `navbar-deployment-<hash1>` agora não tem pods rodando, mas que a `navbar-deployment-<hash2>` agora tem 3 rodando.

## Troubleshooting

Utilizar `kubectl describe` e `kubectl logs` nas deployments e nos pods traz bastante informação

Podemos entrar dentro do pod e executar algum comando através de:

```powershell
kubectl exec -it helloworld-deployment-<hash> -c helloworld -- ps -ef
```

O que vier depois de `--` é o comando desejado a executar. A flag `-c` é pra especificar o pod que desejamos ter acesso, caso haja mais de um pod rodando.

## Exemplo Maior

Vamos usar um banco de dados MongoDB. Esse walkthrough está disponível na documentação oficial do Kubernetes.

1. `kubectl create -f .\05_01\guestbook.yaml`

2. Esperar os pods ficarem up, acessar a URL e fazer um teste: `minikube service frontend --url`

## Dashboard

Tudo que se faz pelo kubectl pode ser executado no dashboard.

1. Abrir o dashboard: `minikube enable dashboard` & `minikube dashboard`

## Configuration Data

1. `kubectl create -f .\05_03\reader-deployment.yaml`

Ao rodar um `kubectl logs` no pod, vemos que o `log_level` está com valor de erro, e queremos arrumar isso durante o deployment. Para isso precisamos criar uma configmap:

2. `kubectl create configmap logger --from-literal=log_level=debug`

O yaml do passo 3 utiliza o nome da configmap criada no passo 2 (logger) na variável `log_level`. Ou seja, adicionamos uma configmap em tempo de deployment.

3. `kubectl create -f .\05_03\reader-configmap-deployment.yaml`

4. Listar as configmaps: `kubectl get configmaps`

5. Ver a especificação da configmap: `kubectl get configmap/logger -o yaml`

## Application Secrets

Utilizado para connections strings de banco de dados, por exemplo, ou informações sensíveis.

Criamos uma secret do tipo `generic` de chave api_key e valor 123456789.

1. `kubectl create secret generic apikey --from-literal=api_key=123456789`

2. `kubectl get secrets/apikey -o yaml`

No output do passo 2, pode ser observado que o valor de `apikey` está criptografado.

3. `kubectl create -f .\05_04\secretreader-deployment.yaml`

4. `kubectl logs secretreader-<hash>`

## Jobs

1. `kubectl create -f .\05_05\simplejob.yaml`

2. `kubectl get jobs`

O yaml deste recurso realiza um for e um print de números. Após fazer o deployment, basta visualizar os logs do pod para ver o output com a contagem de 9 até 1.

### Cron Jobs

A cronjob a seguir roda a cada minuto.

1. `kubectl create -f .\05_05\cronjob.yaml`

2. `kubectl get cronjobs`

Se ficarmos rodarmos um `kubectl get jobs` a cada minuto, vemos que a cada minuto um novo `hellocron` é executado.

3. Editar um cronjob: `kubectl edit cronjobs/hellocron` (se tiver um editor configurado)

    - Se não tiver, pode editar pelo dashboard do minikube ou então assim:

    - Setar o suspend para true, assim a cronjob não roda mais: `kubectl patch cronjobs hellocron -p '{\"spec\" : {\"suspend\" : true }}'`

## DaemonSet

Documentação oficial: A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.

Não entendi muito bem, mas vamo lá.

1. `kubectl create -f .\05_06\daemonset.yaml`

2. `kubectl get daemonsets`

3. Adiciona uma label ao nodo do minikube: `kubectl label nodes/minikube infra=development --overwrite`

4. Visualizar labels do nodo: `kubectl get nodes --show-labels`

5. `kubectl create -f .\05_06\daemonset-infra-development.yaml`

Agora, ao rodar um `get daemonsets`, podemos ver que ela está ativa, e pelo selector `infra=development`, ela foi atribuída ao nodo `minikube`

6. `kubectl create -f .\05_06\daemonset-infra-prod.yaml`

Agora, ao rodar um `get daemonsets`, podemos ver que o daemonset não está ativo, pois não há nenhum nodo com uma label `infra=production`.

### StateFulSets

Manage the deployment and scaling for a set of pods and provide guarantees about the ordering and the uniqueness of these pods

O exemplo será feito com zookeeper, que é um armazenamento de chaves-valor.

1. `kubectl create -f .\05_06\statefulset.yaml`

2. `kubectl get statefulsets`

