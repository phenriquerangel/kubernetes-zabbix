#### 💻 Pré-requisitos
Cluster Kubernetes com 2 servidores
- Master - 2 Cpu , 4 Gb , 50Gb de disco e SO Ubuntu 20.04
- Worker - 2 Cpu , 4 Gb , 50Gb de disco e SO Ubuntu 20.04

Kubernetes utilizado para o estudo é o v1.21.4 (Neste momento, o ambiente é o ambiente utilizado para a prova do CKA)

Primerias configurações no cluster :

- executar namespace.yaml para criar o namespace zabbix
    ```
    kubectl apply -f namespace.yaml
    ```

- executar rbac_zabbix.yaml  para dar o permissionamento necessário ,para o Agent do zabbix. ( Role, Rolebinding e ServiceAccount)
    ```
    kubectl apply -f rbac_zabbix.yaml
    ```

- executar secret-data.yaml para criar arquivos de certificado e senhas
    ```
    kubectl apply -f secret-data.yaml
    ```
OBS: chaves padrões da zabbix.com para estudos
#### Zabbix - Database - MySQL :

- Primeiro vamos executar o arquivo mysql-db-volumes.yaml para criação do volume persistente. que irá se alocar no /mnt/dados
	```
    kubectl apply -f database-mysql/mysql-db-volumes.yaml
	kubectl get pv -n zabbix
	kubectl get pvc -n zabbix
    ```

- criação do serviço de comunicação mysql-db-svc.yaml 
	```
    kubectl apply -f database-mysql/mysql-db-svc.yaml
    ```

- Criação do banco em si, validando nos logs a criação do banco de dados, arquivo mysql-db-deployment.yaml 
	```
    kubectl apply -f database-mysql/mysql-db-deployment.yaml
    ```

- Check logs
	```
    kubectl logs -f container_name -n zabbix
    ```

#### Zabbix - Server 

- executar arquivo zbx-srv-svc.yaml para criação do serviço
	```
    kubectl apply -f server/zbx-srv-svc.yaml
    ```

- executar arquivo zbx-srv-deployment.yaml para criação do container do server, o mesmo também irá subir o container do Zabbix Trapper
	```
    kubectl apply -f server/zbx-srv-deployment.yaml
    ```

#### Zabbix - Web

- executar arquivo zbx-web-svc.yaml para criação do serviços (HTTP e HTTPS) e também serviço web da porta 10053 para uso interno do zabbix 
	```
    kubectl apply -f web/zbx-web-svc.yaml
    ```
	
- executar arquivo zbx-web-deployment.yaml para criação do template do container Zabbix Web
	```
    kubectl apply -f web/zbx-web-deployment.yaml
    ```
	
- executar arquivo zbx-web-rep-contr.yaml para criação do Replication Controller 
	```
    kubectl apply -f web/zbx-web-rep-contr.yaml
    ```
	
- executar arquivo zbx-web-hpa.yaml ára criação do Horizontal Pod Autoscale, baseado no Replication Controller , o que fará os Pods do Zabbix Web iniciar
	```
    kubectl apply -f web/zbx-web-hpa.yaml
    ```

OBS: Neste passo, o svc foi alterado para Type NodePort para testes locais. a informação original seria ClusterIP.

#### Zabbix - Agent 

- Executar arquivo zbx-agent-svc.yaml para criação do serviço e expor a porta 10050
	```
    kubectl apply -f agent/zbx-web-hpa.yaml
    ```

- Executar arquivo zbx-agent-deamonset.yaml para criação do DeamonSet do zabbix Agent e subida dos Pods do serviço

	```
    kubectl apply -f agent/zbx-agent-deamonset.yaml
    ```

Após isso, o ambiente básico e PARA ESTUDOS estara no ar.

Para acesso ao ambiente, é necesário validar com o comando abaixo, qual porta será disposta no node para acesso
    ```
    kubectl get svc -n zabbix
    ```

OBS: 
-   O ambiente não deve ser colocado em produção, é apenas para aprendizado em laboratório.
-   Todos arquivos setados nesse repositório foram baseados na documentação oficial do Zabbix.