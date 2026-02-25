# Labo 3: Monitoring

DEMO: https://hogent.cloud.panopto.eu/Panopto/Pages/Viewer.aspx?id=b395f18d-8b2c-41f5-a2ea-b3b500d7646f

In deze opdracht heb ik een monitoring systeem opgezet met Prometheus en Grafana. Prometheus verzamelt de data van verschillende servers en services, en Grafana toont deze overzichtelijk op mooie dashboards.

## Prerequisites

In de requirements.yml heb ik de volgende roles toegevoegd:

```bash
grafana.grafana
prometheus.prometheus
```

## Node Exporter installeren

Op alle virtuele machines heb ik de node_exporter role geïnstalleerd. Node Exporter verzamelt basis systeemstatistieken zoals CPU-gebruik, geheugen, schijf en netwerkgebruik. Dit is nodig zodat Prometheus deze metrics kan ophalen.

Ik heb gecontroleerd of Node Exporter draait door op elke VM te curlen naar poort 9100, bijvoorbeeld:

```bash
curl http://srv100.infra.lan:9100/metrics
```

Dit gaf een lijst met metrics terug, wat bevestigde dat node exporter correct werkte. De poort 9100 was nu open op alle VMs en bereikbaar vanaf de monitoring server.

## Monitoring server opzetten

Ik maakte een nieuwe VM `srv004` aan met vagrant, ik gaf deze het IP-adres 172.16.128.4. Op deze server heb ik de prometheus role geïnstalleerd. 

In de site.yml heb ik de volgende configuratie toegevoegd om Prometheus te vertellen welke targets hij moet monitoren:

```yaml
 prometheus_scrape_configs:
      - job_name: node_exporter
        static_configs:
          - targets:
              - srv100.infra.lan:9100
              - srv001.infra.lan:9100
              - srv002.infra.lan:9100
              - srv003.infra.lan:9100
              - localhost:9100
```

Hiermee monitort Prometheus de node exporters op alle servers in het netwerk, inclusief zichzelf.

Na het uitvoeren van de playbook kon ik de Prometheus webinterface bereiken op `http://srv004.infra.lan:9090`. Hier kon ik de targets zien en controleren of ze "UP" waren. 

## Gebruik van DNS via Ansible

Om hostnamen in plaats van IP-adressen te gebruiken, heb ik een aparte playbook set-dns.yml geschreven die /etc/resolv.conf aanpaste op alle servers:

```yaml
---
- name: Configure DNS on all servers
  hosts: all
  become: true
  tasks:
    - name: Set DNS server in resolv.conf
      lineinfile:
        path: /etc/resolv.conf
        regexp: '^nameserver'
        line: 'nameserver 172.16.128.1'
        state: present
```

Hierna kon ik de servers pingen via hun hostnamen, bijvoorbeeld:

```bash
ping srv001.infra.lan
ping srv002.infra.lan
```

## Grafana dashboard maken

Het dashboard in Grafana heb ik gehaald van de grafana workshop site. https://grafana.com/grafana/dashboards/1860-node-exporter-full/ Dit is een kant-en-klaar dashboard voor het monitoren van systemen met Node Exporter.

![alt text](<img/Schermafbeelding 2025-12-16 143829.png>)