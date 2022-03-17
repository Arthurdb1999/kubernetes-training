# Instalação

1. Baixar e Instalar Docker Desktop, kubectl e Minikube

2. Adicionar o .exe do kubectl e do minikube em uma mesma pasta. Adicionar esta pasta na variável PATH das variáveis de ambiente.

3. Ativar o Microsoft Hyper-V no powershell: 
```powershell
Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Hyper-V -All
```

4. Dentro do Hyper-V Manager: Virtual Switch Manager -> Create Virtual Switch -> Dar um nome para a conexão (minikube) -> Connection Type: Internal network -> Apply, Ok.

5. Dentro do Painel de Controle: Network and Internet -> Network and Sharing Center -> Botão esquerdo na conexão de internet -> Propriedades -> Sharing -> Marcar a primeira opção e selecionar a conexão criada (minikube) -> Ok

6. Certificar que o Docker esteja rodando

7. 
```powershell
minikube start --driver="hyperv" --hyperv-virtual-switch="minikube"
```