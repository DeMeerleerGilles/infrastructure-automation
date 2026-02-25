# Lab Report: Kubernetes

## Learning goals

- Understanding the basic architecture of Kubernetes
- Being able to operate a Kubernetes cluster
    - Applying changes using manifest files
- Being able to manipulate Kubernetes resources
    - Pods
    - Controllers: ReplicaSets, Deployments, Services
    - Organising applications: Labels, Selectors
- Deploying a multi-tier application on a Kubernetes cluster

## Acceptance criteria

- Demonstrate that your Kubernetes cluster is running and that you are able to manage it:
    - Open the dashboard to show what's running on the cluster: nodes, pods, services, deployments, etc.
    - Also show these from the command line (using `kubectl`)
- Demonstrate the applications that are running on the cluster:
    - Show them in the web browser
    - From the command line, show which resources are used by each application (Pods, Deployments, Services, etc.)
- Show an example of filtering by label and an operation on the filtered resources (e.g. delete all resources with a certain label)
- Scale the echo-all deployment to 5 replica's using the manifest file
- Show your lab notes and cheat sheet with useful commands

## Prerequisites installeren

Ik begon met het dowloaden van kubectl:

```bash
curl.exe -LO "https://dl.k8s.io/release/v1.30.0/bin/windows/amd64/kubectl.exe"
move .\kubectl.exe "C:\Windows\System32\kubectl.exe"
kubectl version --client
```

Vervolgens installeerde ik minikube:

```bash
curl.exe -LO https://storage.googleapis.com/minikube/releases/latest/minikube-installer.exe
Start-Process .\minikube-installer.exe
minikube version
```

Nu moeten we minikube nog configureren om de juiste driver te gebruiken. Ik koos voor de virtualbox driver:

```bash
minikube config set driver virtualbox
minikube start
```

Om te controleren of alles correct is geïnstalleerd, gebruikte ik de volgende commando's:

```bash
kubectl get nodes
minikube status
```

## Kubernetes Dashboard installeren
Vervolgens installeerde ik het Kubernetes Dashboard met de volgende commando's:

```bash
minikube addons enable metrics-server
minikube addons enable dashboard
minikube dashboard
```

Het dashboard opende automatisch in mijn webbrowser:

![alt text](<img/Schermafbeelding 2025-11-23 112443.png>)

## Hello world applicatie deployen

Nu werkte ik de hello world applicatie van de officiële Kubernetes documentatie uit:
Ik begon met het aanmaken van een deployment die de pod met de hello world container zal beheren.

```bash
kubectl create deployment hello-node --image=registry.k8s.io/e2e-test-images/agnhost:2.53 -- /agnhost netexec --http-port=8080
```

Hierna bekeek ik de status van de deployment:

```bash
PS C:\WINDOWS\system32> kubectl get deployments
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
hello-node   0/1     1            0           7s
```

Ik vroeg vervolgens de pods op:

```bash
kubectl get pods
NAME                          READY   STATUS    RESTARTS   AGE
hello-node-6c9b5f4b59-rrfdr   1/1     Running   0          114s
```

Daarna bekeek ik de cluster events:

```bash
kubectl get events
```

Alles stond op normaal dus ik ging verder met het bekijken van de logs van de pod:

```bash
kubectl logs hello-node-6c9b5f4b59-rrfdr
I1123 10:38:08.053138       1 log.go:245] Started UDP server on port  8081
I1123 10:38:08.053540       1 log.go:245] Started HTTP server on port 8080
```

### Service aanmaken
Vervolgens maakte ik een service aan om de applicatie toegankelijk te maken:

```bash
kubectl expose deployment hello-node --type=LoadBalancer --port=8080
```

Als ik vervolgens het volgende commando uitvoerde:

```bash
minikube service hello-node
```

Opende er automatisch een webpagina in mijn browser met de volgende inhoud:

![alt text](<img/Schermafbeelding 2025-11-23 115516.png>)

### Add ons bekijken
Tot slot bekeek ik de beschikbare add ons in minikube:

```bash
minikube addons list
```

Ik enablede de metrics-server en dashboard add ons:

```bash
minikube addons enable metrics-server
```

Om de output van de metrics-server te bekijken, gebruikte ik het volgende commando:

```bash
kubectl top pods
```

### Clean up
Tot slot verwijderde ik de deployment en service die ik had aangemaakt:

```bash
kubectl delete service hello-node
kubectl delete deployment hello-node
```

## Learn Kubernetes Basics

Na het voltooien van de hello world applicatie, volgde ik de stappen uit de learn kubernetes basics tutorial op.

Wat zijn kubernetes clusters?

Kubernetes clusters zijn een verzameling van nodes (fysieke of virtuele machines) die samenwerken om containerized applicaties te draaien. Elk cluster heeft een master node die de controle en coördinatie van de andere nodes beheert, en meerdere worker nodes die de daadwerkelijke applicaties uitvoeren.

 Kubernetes automates the distribution and scheduling of application containers across a cluster in a more efficient way. 

### Using kubectl to Create a Deployment

Kubernetes gebruikt deployments om de gewenste staat van een applicatie te definiëren. Een deployment specificeert welke container image moet worden gebruikt, hoeveel replica's er moeten draaien, en hoe updates moeten worden beheerd.

Deployen van een applicatie met kubectl kan gedaan worden met het volgende commando:

```bash
kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1
```

We starten vervolgens een proxy server om toegang te krijgen tot de applicatie:

```bash
kubectl proxy
```

We kunnen vervolgens testen of de applicatie werkt door een curl commando uit te voeren:

```bash
curl http://localhost:8001/version
```

### Viewing Pods and Nodes

Een pod is een groep van één of meer containers die op dezelfde host draaien en dezelfde netwerk- en opslagbronnen delen.

#### Check application configuration

We kunnen de pods bekijken met het volgende commando:

```bash
kubectl get pods
```

Dit geeft ons een lijst van alle pods die momenteel draaien in het cluster, inclusief hun status en leeftijd.

```bash
kubectl describe pods
```

Dit geeft gedetailleerde informatie over een specifieke pod, inclusief de container specificaties, events.

Troubleshooting met kubectl
Belangrijke commando’s:

- kubectl get: lijst resources
- kubectl describe: gedetailleerde info
- kubectl logs: logs van containers
- kubectl exec: commando uitvoeren in een container

#### Using a Service to Expose Your App

Een service in Kubernetes is een abstractie die een consistente manier biedt om toegang te krijgen tot een set van pods, ongeacht waar ze draaien in het cluster.

```bash
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```

Om te kijken welke poort de service gebruikt, kunnen we het volgende commando uitvoeren:

```bash
kubectl describe services/kubernetes-bootcamp
```

De deployment heeft automatisch een label toegevoegd aan de pods, die we kunnen zien met het volgende commando:

```bash
kubectl describe deployment
```

#### Scaling Your App

We kunnen de applicatie schalen door met replica's te werken. 

We beginnen met het opbrengen van een nieuwe applicatie:

```bash
kubectl expose deployment/kubernetes-bootcamp --type="LoadBalancer" --port 8080
```

Wanneer we een deployment opschalen betekent dit dat nieuwe pods worden aangemaakt om aan de gewenste staat te voldoen. 

Om de replicaset te bekijken, gebruiken we het volgende commando:

```bash
kubectl get rs
``` 

Nu gaan we opschalen naar 4 replica's:

```bash
kubectl scale deployments/kubernetes-bootcamp --replicas=4
```

We zien dat er nu 4 pods draaien:

```bash
kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   4/4     4            4           142m
kubectl get pods
NAME                                   READY   STATUS    RESTARTS   AGE
kubernetes-bootcamp-658f6cbd58-5t8m6   1/1     Running   0          36s
kubernetes-bootcamp-658f6cbd58-9zr44   1/1     Running   0          36s
kubernetes-bootcamp-658f6cbd58-bgpdg   1/1     Running   0          36s
kubernetes-bootcamp-658f6cbd58-rp2sd   1/1     Running   0          142m
```

Scalen naar 2 replica's:

```bash
kubectl scale deployments/kubernetes-bootcamp --replicas=2
```

We zien dat er nu 2 pods draaien:

```bash
kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
kubernetes-bootcamp   2/2     2            2           143m
```

#### Updating Your App

Een rolling update is een manier om een applicatie bij te werken zonder downtime. Kubernetes zorgt ervoor dat de nieuwe versie van de applicatie wordt uitgerold terwijl de oude versie nog steeds beschikbaar is. Dit wordt gedaan door geleidelijk nieuwe pods met de nieuwe versie te starten en oude pods te verwijderen.

We kunnen de huidige versie van onze pods bekijken met het volgende commando:

```bash
kubectl describe pods
```

Om de image van onze applicatie bij te werken naar versie v2, gebruiken we het volgende commando:

```bash
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=docker.io/jocatalin/kubernetes-bootcamp:v2
```

We zien nu dat de pods worden bijgewerkt:

```bash
kubectl get pods
NAME                                   READY   STATUS        RESTARTS   AGE
kubernetes-bootcamp-57cc954bb9-2kz7b   1/1     Running       0          11s
kubernetes-bootcamp-57cc954bb9-45rhx   1/1     Running       0          17s
kubernetes-bootcamp-658f6cbd58-9zr44   1/1     Terminating   0          8m58s
kubernetes-bootcamp-658f6cbd58-rp2sd   1/1     Terminating   0          151m
```

We kunnen nu de update confirmeren met het volgende commando:

```bash
kubectl rollout status deployments/kubernetes-bootcamp
```

Roll back an update:

We voeren nu een update uit naar versie v10:

```bash
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
```

Deze versie bestaat niet, dus de pods zullen in een crashloop terechtkomen. We kunnen dit controleren met het volgende commando:

```bash
PS C:\WINDOWS\system32> kubectl get pods
NAME                                   READY   STATUS             RESTARTS   AGE
kubernetes-bootcamp-57cc954bb9-2kz7b   1/1     Running            0          11m
kubernetes-bootcamp-57cc954bb9-45rhx   1/1     Running            0          11m
kubernetes-bootcamp-677ff875c4-2nbbv   0/1     ImagePullBackOff   0          45s
```

We kunnen nu terugrollen naar de vorige versie met het volgende commando:

```bash
kubectl rollout undo deployments/kubernetes-bootcamp
```

Om onze locale cluster op te ruimen, gebruiken we het volgende commando:

```bash
kubectl delete deployments/kubernetes-bootcamp services/kubernetes-bootcamp
```

## Working with manifest files

Gekomen tot 4.2.2

Hierna voerde ik de volgende commando's uit om de manifest files te gebruiken:

```bash
kubectl apply -f bootcamp-deployment.yml
kubectl apply -f bootcamp-service.yml
```

Het eerste commando maakt de deployment aan, het tweede commando zorgt ervoor dat de app beschikbaar is voo de users. Met het volgende commando kunnen we de app bekijken in onze browser:

```bash
minikube service bootcamp-service
```

We kunnen ook opnieuw de status van de deployment bekijken:

```bash
kubectl get deployments
NAME                  READY   UP-TO-DATE   AVAILABLE   AGE
bootcamp-deployment   1/1     1            1           27m
kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
bootcamp-deployment-78f8fbbf-zctk4   1/1     Running   0          27m
```

We kunnen nu hetzelfde doen maar met één enkele file:

```bash
kubectl apply -f bootcamp-all.yml
```

Ook hier kunnen we via minikube de app bekijken:

```bash
minikube service bootcamp-service
```

We veranderen in de bootcamp-all.yml file het aantal replicas van 1 naar 3 en voeren het volgende commando uit om de deployment bij te werken:

```bash
kubectl apply -f bootcamp-all.yml
kubectl get deployments
NAME                      READY   UP-TO-DATE   AVAILABLE   AGE
bootcamp-all-deployment   3/3     3            3           43h
bootcamp-deployment       1/1     1            1           44h
```

We zien nu dat er 3 pods draaien, waarvan 1 die recent is aangemaakt:

```bash
NAME                                     READY   STATUS    RESTARTS      AGE
bootcamp-all-deployment-78f8fbbf-cwpf2   1/1     Running   1 (43h ago)   43h
bootcamp-all-deployment-78f8fbbf-jhqfn   1/1     Running   1 (43h ago)   43h
bootcamp-all-deployment-78f8fbbf-lqpv9   1/1     Running   0             15m
bootcamp-deployment-78f8fbbf-zctk4       1/1     Running   1 (43h ago)   44h
```

## 4.3 Selectors and Labels

Om een overzicht te bewaren van alles wat er in een kubernetes cluster gebeurt is het belangrijk om labels en selectors te gebruiken. Dit zijn simpele key-value paren die we kunnen toewijzen aan kubernetes objecten zoals pods, services en deployments.

Met het volgende commando kunnen we de labels van onze pods bekijken:

```bash
kubectl get pods --show-labels
```

We gaan nu het label application_type=demo toevoegen aan onze pods. We halen eerst de naam van onze pods op:

```bash
kubectl get pods
NAME                                     READY   STATUS    RESTARTS      AGE
bootcamp-all-deployment-78f8fbbf-cwpf2   1/1     Running   1 (43h ago)   43h
bootcamp-all-deployment-78f8fbbf-jhqfn   1/1     Running   1 (43h ago)   43h
bootcamp-all-deployment-78f8fbbf-lqpv9   1/1     Running   0             37m
bootcamp-deployment-78f8fbbf-zctk4       1/1     Running   1 (43h ago)   44h
```

Nu voegen we het label toe aan elke pod:

```bash
kubectl label pods bootcamp-all-deployment-78f8fbbf-cwpf2 application_type=demo
kubectl label pods bootcamp-all-deployment-78f8fbbf-jhqfn application_type=demo
kubectl label pods bootcamp-all-deployment-78f8fbbf-lqpv9 application_type=demo
kubectl label pods bootcamp-deployment-78f8fbbf-zctk4 application_type=demo
```

Nu gaan we proberen om een pod van label te veranderen.

```bash
kubectl label pod bootcamp-all-deployment-78f8fbbf-jhqfn application_type=test
error: 'application_type' already has a value (demo), and --overwrite is false
```

Zoals verwacht krijgen we een foutmelding omdat er al reen waarde bestaat voor dat label. We gebruiken nu de --overwrite flag om het label te veranderen:

```bash
kubectl label pod bootcamp-all-deployment-78f8fbbf-jhqfn application_type=test --overwrite
```

Nu is het label succesvol veranderd.

```bash
kubectl get pods --show-labels
NAME                                     READY   STATUS    RESTARTS      AGE   LABELS
bootcamp-all-deployment-78f8fbbf-cwpf2   1/1     Running   1 (43h ago)   44h   app=bootcamp,application_type=demo,pod-template-hash=78f8fbbf
bootcamp-all-deployment-78f8fbbf-jhqfn   1/1     Running   1 (43h ago)   44h   app=bootcamp,application_type=test,pod-template-hash=78f8fbbf
bootcamp-all-deployment-78f8fbbf-lqpv9   1/1     Running   0             40m   app=bootcamp,application_type=demo,pod-template-hash=78f8fbbf
bootcamp-deployment-78f8fbbf-zctk4       1/1     Running   1 (43h ago)   44h   app=bootcamp,application_type=demo,pod-template-hash=78f8fbbf
```

Hierna probeerde ik om alle pods met het label application_type=demo te verwijderen:

```bash
kubectl delete pods -l application_type=demo
```

We merken dat de pods automatisch worden hermaakt door de deployment, als we de pods opnieuw opvragen:

```bash
kubectl get pods --show-labels    
NAME                                     READY   STATUS    RESTARTS      AGE     LABELS
bootcamp-all-deployment-78f8fbbf-csjzq   1/1     Running   0             4m49s   app=bootcamp,pod-template-hash=78f8fbbf  
bootcamp-all-deployment-78f8fbbf-jhqfn   1/1     Running   1 (43h ago)   44h     app=bootcamp,application_type=test,pod-template-hash=78f8fbbf
bootcamp-all-deployment-78f8fbbf-nqbpg   1/1     Running   0             4m49s   app=bootcamp,pod-template-hash=78f8fbbf  
bootcamp-deployment-78f8fbbf-29nx4       1/1     Running   0             4m49s   app=bootcamp,pod-template-hash=78f8fbbf 
```

Zien we dat de labels application_type=demo verdwenen zijn bij de hergemaakte pods.

We halen het label application_type=test weg bij de pod:

```bash
kubectl label pod bootcamp-all-deployment-78f8fbbf-jhqfn application_type-
```
Nu is het label succesvol verwijderd:

```bash
kubectl get pods --show-labels    
NAME                                     READY   STATUS    RESTARTS      AGE     LABELS
bootcamp-all-deployment-78f8fbbf-csjzq   1/1     Running   0             9m15s   app=bootcamp,pod-template-hash=78f8fbbf  
bootcamp-all-deployment-78f8fbbf-jhqfn   1/1     Running   1 (43h ago)   44h     app=bootcamp,pod-template-hash=78f8fbbf  
bootcamp-all-deployment-78f8fbbf-nqbpg   1/1     Running   0             9m15s   app=bootcamp,pod-template-hash=78f8fbbf  
bootcamp-deployment-78f8fbbf-29nx4       1/1     Running   0             9m15s   app=bootcamp,pod-template-hash=78f8fbbf 
```

Ten slotte verwijderde ik alle kubernetes resources die momenteel op het cluster draaiden met het volgende commando:

```bash
kubectl delete all --all
```

### Setting labels in manifest files

Nu voerde ik het volgende commando uit om de resources aan te maken vanuit de manifest files:

```bash
kubectl apply -f 4.3/example-pods-with-labels.yml
```

We bekijken de pods in de productie omgeving met het volgende commando:

```bash
kubectl get pods -l env=production
NAME       READY   STATUS    RESTARTS   AGE
api-prod   1/1     Running   0          25s
db-prod    1/1     Running   0          25s
fe-prod    1/1     Running   0          25s
```

De pods die niet in de productie omgeving draaien bekijken we met het volgende commando:

```bash
kubectl get pods -l 'env!=production'
NAME       READY   STATUS    RESTARTS   AGE
api-prod   1/1     Running   0          25s
db-prod    1/1     Running   0          25s
fe-prod    1/1     Running   0          25s
```

Pods in development en acceptance omgeving bekijken we met het volgende commando:

```bash
kubectl get pods -l 'env in (development,acceptance)'
NAME             READY   STATUS    RESTARTS   AGE
api-acceptance   1/1     Running   0          11m
api-dev          1/1     Running   0          11m
db-acceptance    1/1     Running   0          11m
db-dev           1/1     Running   0          11m
fe-acceptance    1/1     Running   0          11m
fe-dev           1/1     Running   0          11m
```

Pods met versie v2 bekijken we met het volgende commando:

```bash
kubectl get pods -l release_version=2.0
NAME             READY   STATUS    RESTARTS   AGE
api-acceptance   1/1     Running   0          12m
api-dev          1/1     Running   0          12m
db-acceptance    1/1     Running   0          12m
db-dev           1/1     Running   0          12m
fe-acceptance    1/1     Running   0          12m
fe-dev           1/1     Running   0          12m
```

Pods van het API-team met v2.0 bekijken we met het volgende commando:

```bash
kubectl get pods -l team=api,release_version=2.0
NAME             READY   STATUS    RESTARTS   AGE
api-acceptance   1/1     Running   0          14m
api-dev          1/1     Running   0          14m
```

Om alle pods te verwijderen in de development omgeving, gebruikte ik het volgende commando:

```bash
kubectl delete pods -l env=development
```

De snelste manier om deze pods terug aan te maken is door de manifest file opnieuw toe te passen:

```bash
kubectl apply -f 4.3/example-pods-with-labels.yml
```

## 4.4 Deploy a multi-tier web application

We volgen de stappen uit de tutorial op de kubernetes documentatie om een multi-tier web applicatie te deployen.

### Stap 1: Een redis leader deployen

We beginnen met het aanmaken van een redis leader deployment:

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-deployment.yaml
```

We zien dat de pod draait met het volgende commando:

```bash
kubectl get pods
redis-leader-665d87459f-pkwcf   1/1     Running   0          8s
```

Nu maken we een service aan voor de redis leader:

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-leader-service.yaml
kubectl get services
NAME           TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
kubernetes     ClusterIP   10.96.0.1        <none>        443/TCP    25m
redis-leader   ClusterIP   10.102.154.215   <none>        6379/TCP   0s
```

### Stap 2: Redis followers deployen

Om de followers te deployen, gebruiken we het volgende commando:

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-deployment.yaml
```

Nu doen we hetzelfde voor de redis follower service:

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/redis-follower-service.yaml
```

### Stap 3: De frontend deployen

We deployen de frontend met het volgende commando:

```bash
 kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-deployment.yaml
```

De frontend service maken we aan met het volgende commando:

```bash
kubectl apply -f https://k8s.io/examples/application/guestbook/frontend-service.yaml
kubectl get services
NAME             TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)    AGE
frontend         ClusterIP   10.103.69.7      <none>        80/TCP     4s
kubernetes       ClusterIP   10.96.0.1        <none>        443/TCP    38m
redis-follower   ClusterIP   10.110.187.228   <none>        6379/TCP   5m27s
redis-leader     ClusterIP   10.102.154.215   <none>        6379/TCP   13m
```

Nu kunnen we de frontend service openen in onze browser met het volgende commando:

```bash
kubectl port-forward svc/frontend 8080:80
```

We openen vervolgens http://localhost:8080 in onze browser:
![alt text](<img/Schermafbeelding 2025-11-28 161149.png>)

### Uitbreiding wordpress

We starten met het aanmaken van een map voor onze manifest files:

```bash
mkdir wordpress-k8s
cd wordpress-k8s
```

Vervolgens maken we een kustomization.yaml met een Secret.

### Download de MySQL + WordPress manifests

Hierna downloaden we de MySQL en WordPress manifest files:

```bash
Invoke-WebRequest -Uri "https://k8s.io/examples/application/wordpress/mysql-deployment.yaml" -OutFile "mysql-deployment.yaml"
Invoke-WebRequest -Uri "https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml" -OutFile "wordpress-deployment.yaml"
```

Nu voegen we de bestanden toe aan de kustomization.yaml:
resources:
  - mysql-deployment.yaml
  - wordpress-deployment.yaml


Hierna konden we alles uitrollen met het volgende commando:

```bash
kubectl apply -k ./
```

We controleren of alle pods draaien met het volgende commando:

```bash
kubectl get secrets
NAME                    TYPE     DATA   AGE
mysql-pass-m5tftmk94d   Opaque   1      29s
```

Met het volgende commando kunnen we vragen op welke url wordpress draait:

```bash
minikube service wordpress --url
```

![alt text](<img/Schermafbeelding 2025-11-28 163019.png>)
