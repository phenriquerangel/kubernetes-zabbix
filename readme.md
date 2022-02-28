#### 💻 Pré-requisitos
Cluster Kubernetes with 2 servers
- Master - 2 Cpu , 4 Gb , 50Gb de disco e SO Ubuntu 20.04
- Worker - 2 Cpu , 4 Gb , 50Gb de disco e SO Ubuntu 20.04

Kubernetes version is v1.21.4 (In this moment, this is environment to CKA certified course)

First cluster configurations :

- execute file namespace.yaml for create namespace zabbix on cluster
    ```
    kubectl apply -f namespace.yaml
    ```

- execute file rbac_zabbix.yaml for grant permissions on cluster for agent zabbix (Role, Rolebinding e ServiceAccount)
    ```
    kubectl apply -f rbac_zabbix.yaml
    ```

- execute file secret-data.yaml for create a secret with certifieds and passwords default 
    ```
    kubectl apply -f secret-data.yaml
    ```
OBS: default keys from zabbix.com , just for learning enviroment

#### Zabbix - Database - MySQL :

- Execute file mysql-db-volumes.yaml for create a persistent volume, wich alocated on /mnt/dados on machine cluster( worker or node)
	```
    kubectl apply -f database-mysql/mysql-db-volumes.yaml
	kubectl get pv -n zabbix
	kubectl get pvc -n zabbix
    ```

- execute file mysql-db-svc.yaml for create service communication from db-mysql
	```
    kubectl apply -f database-mysql/mysql-db-svc.yaml
    ```

- execute file mysql-db-deployment.yaml, for create the database. Will needed check logs for that's creation it's ok. 
	```
    kubectl apply -f database-mysql/mysql-db-deployment.yaml
    ```

- Check logs
	```
    kubectl logs -f container_name -n zabbix
    ```

#### Zabbix - Server 

- execute file zbx-srv-svc.yaml for create zabbix server service.
	```
    kubectl apply -f server/zbx-srv-svc.yaml
    ```

- execute file zbx-srv-deployment.yaml for create a zabbix server container. this step, create automaticly a zabbix trapper container too
	```
    kubectl apply -f server/zbx-srv-deployment.yaml
    ```

#### Zabbix - Web

- executer file zbx-web-svc.yaml for create zabbix web services from HTTP and HTTPS protocols and port 10053 for internal zabbix server 
	```
    kubectl apply -f web/zbx-web-svc.yaml
    ```
	
- execute file zbx-web-deployment.yaml for create container template from Zabbix Web
	```
    kubectl apply -f web/zbx-web-deployment.yaml
    ```
	
- execute file zbx-web-rep-contr.yaml for create Replication Controller 
	```
    kubectl apply -f web/zbx-web-rep-contr.yaml
    ```
	
- execute file  zbx-web-hpa.yaml for create a Horizontal Pod AutoScale based on Replication Controller, with start a pods from container Zabbix Web
	```
    kubectl apply -f web/zbx-web-hpa.yaml
    ```

PS: 
-   For this step, svc type original is ClusterIP, does changed from NodePort for realized to facilitate access.

#### Zabbix - Agent 

- execute file zbx-agent-svc.yaml for create service to expose web port
	```
    kubectl apply -f agent/zbx-web-hpa.yaml
    ```

- execute file zbx-agent-deamonset.yaml for create a DeamonSet from Zabbix Agent 

	```
    kubectl apply -f agent/zbx-agent-deamonset.yaml
    ```

After this, the basic environment for learning its online. 

For acess this environment, do you need check port exposed from kubernets.
    ```
    kubectl get svc -n zabbix
    ```

PS:
- This environment should not be put into production, it's just for LAB LEARNING.

- All file described on this repos, can be found on Zabbix Official documentation