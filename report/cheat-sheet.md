# Cheat sheets and checklists

- Student name: Gilles De Meerleer
- GitHub repo: <https://github.com/HoGentTIN/infra-2526-DeMeerleerGilles>

## Basic commands

| Task                    | Command        |
| :---------------------- | :------------- |
| Query IP address(es)    | `ip a`         |
| Vagrant VM starten      | `vagrant up`   |
| Vagrant VM stoppen      | `vagrant halt` |
| SSH naar Vagrant VM     | `vagrant ssh`  |
| Docker containers tonen | `docker ps -a` |

## Labo 2: VM lab

### Ansible commands

| Task                         | Command                                                  |
| :--------------------------- | :------------------------------------------------------- |
| Test connectivity naar hosts | `ansible all -m ping -i inventory.yml`                   |
| Voer playbook uit            | `ansible-playbook -i inventory.yml site.yml`             |
| Requirements installeren     | `ansible-galaxy install -r requirements.yml`             |
| Router instellen             | `ansible-playbook -i inventory.yml router-config.yml`    |
| DNS instellen                | `ansible-playbook -i inventory.yml set-dns.yml`          |
| Specifieke host targetten    | `ansible-playbook -i inventory.yml -l HOSTNAME site.yml` |
| Key toevoegen                | `ssh-keyscan -H 172.16.128.1 >> ~/.ssh/known_hosts`      |
| test ping naar alle hosts    | `ansible all -m ping -i inventory.yml`                   |

## Labo 4: Kubernetes

### Minikube commands

| Task                                   | Command                                 |
| :------------------------------------- | :-------------------------------------- |
| Start Minikube                         | `minikube start`                        |
| Stop Minikube                          | `minikube stop`                         |
| List Minikube addons                   | `minikube addons list`                  |
| Enable metrics-server addon            | `minikube addons enable metrics-server` |
| Enable dashboard addon                 | `minikube addons enable dashboard`      |
| Dashboard openen                       | `minikube dashboard`                    |
| Kijken op welke url een service draait | `minikube service wordpress --url`      |

### Kubectl commands

| Task                           | Command                                                      |
| :----------------------------- | :----------------------------------------------------------- |
| Pods tonen                     | `kubectl get pods`                                           |
| Services tonen                 | `kubectl get services`                                       |
| Deployments tonen              | `kubectl get deployments`                                    |
| Nodes tonen                    | `kubectl get nodes`                                          |
| Labels tonen                   | `kubectl get pods --show-labels`                             |
| Filteren op een label          | `kubectl get pods -l app=wordpress`                          |
| Label toevoegen                | `kubectl label pod POD_NAME app=label1`                      |
| Pod logs bekijken              | `kubectl logs POD_NAME`                                      |
| Pods verwijderen               | `kubectl delete pod POD_NAME`                                |
| Pods verwijderen met een label | `kubectl delete pod -l env=label`                            |
| Service tonen in de browser    | `kubectl port-forward svc/frontend 8080:80`                  |
| Deployment schalen             | `kubectl scale deployment DEPLOYMENT_NAME --replicas=NUMBER` |
| Manifest file toepassen        | `kubectl apply -f bootcamp-deployment.yml`                   |
| versie bekijken                | `kubectl describe pods`                                      |

## Git workflow

Simple workflow for a personal project without other contributors:

| Task                                         | Command                   |
| :------------------------------------------- | :------------------------ |
| Current project status                       | `git status`              |
| Select files to be committed                 | `git add FILE...`         |
| Commit changes to local repository           | `git commit -m 'MESSAGE'` |
| Push local changes to remote repository      | `git push`                |
| Pull changes from remote repository to local | `git pull`                |

## Checklist network configuration

1. Is the IP address correct? `ip a`
2. Is the router/default gateway correct? `ip r -n`
3. Is a DNS-server available? `cat /etc/resolv.conf`
