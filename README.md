### Lokales K8S Cluster mit Prometeus, Grafana und Loki aufsetzen

## Repo
- Klone das Repo https://github.com/TS-BudgetBook/budgetbook/blob/feature/monitoring-statistics/k8s/LOGGING.md

## Hosteintrag hinzufügen
- Starte den Editor als Admin und füge folgendes der hosts Datei (c:\Windows\System32\Drivers\etc\hosts) hinzu: 127.0.0.1 budgetbook.me

## Docker Images bauen
- In den jeweiligen Microservices (auth, expense, frontend und statistics) das Image wie folgt bauen:
- docker build -t <SERVICENAME>:latest .
- Dabei <SERVICENAME> mit dem jeweiligen Service ersetzen

## NGINX-INGRESS im Cluster installieren
- helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace

## ## MYSQL Datenbank im Cluster installieren
- helm upgrade --install mysql --set auth.database=budgetbook,auth.username=budgetbook,auth.password=BuDg3tB00k oci://registry-1.docker.io/bitnamicharts/mysql

- Parameter Beispiele: --set auth.database=budgetbook,auth.user=app_database,auth.password=BuDg3tB00k

## MySQL Root Password anzeigen lassen
- kubectl get secret --namespace default mysql -o jsonpath="{.data.mysql-root-password}" | base64 -d

## TLS Zertifikate generieren
- Folgenden Befehl zum Installieren in Powershell als Admin ausführen: choco install mkcert
- Erstellung einer lokalen CA: mkcert -install
- Folgenden Befehl innerhalb des Repos ausführen: mkcert budgetbook.me localhost 127.0.0.1 ::1

## K8S Secret erstellen
- Folgender Befehl zum Anzeigen mit --dry-run: kubectl create secret tls budgetbook.me --cert=./budgetbook.me+3.pem --key=./budgetbook.me+3-key.pem --dry-run=client --output=yaml
- Im Cluster ausführen: kubectl create secret tls budgetbook.me --cert=./budgetbook.me+3.pem --key=./budgetbook.me+3-key.pem

## Die Andwendung im Cluster deployen
- Verzeichnis wechseln: cd k8s/
- Deployment ausführen: kubectl apply -f .
- Pods anzeigen lassen: kubectl get pods

## Anwendung im Browser starten
- https://budgetbook.me 



## Prometheus Repo hinzufügen
- Helm Repo hinzufügen: helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
- Helm Repo updaten: helm repo update

## Prometheus installieren
- helm install prometheus prometheus-community/kube-prometheus-stack --namespace=metrics --create-namespace

## Port Forward zu Prometheus via kubectl
- ID anzeigen: kubectl get pods -n metrics
- Die Prometheus-Server ID raussuchen und mit folgendem Befehl einfügen: kubectl port-forward <prometheus-server-POD-ID> 9090:9090 -n metrics
- Dabei <prometheus-server-POD-ID> mit der tatsächlichen Server-ID ersetzen

## Hostnamen zur hosts Datei hinzufügen
- 127.0.0.1 grafana.budgetbook.me

## TLS Secret erstellen aus dem Cert für grafana.budgetbook.me
- mkcert grafana.budgetbook.me localhost 127.0.0.1 ::1
- kubectl create secret tls grafana.budgetbook.me --cert=./grafana.budgetbook.me+3.pem --key=./grafana.budgetbook.me+3-key.pem -n metrics



## Grafana Loki zum Repo hinzufügen
- Hinzufpgen: helm repo add grafana https://grafana.github.io/helm-charts
- Updaten: helm repo update

## Helm upgraden
- Folgenden Befehl innerhalb des k8s/logging/ Ordners ausführen: helm upgrade --install loki grafana/loki -n metrics --values logging.yaml
- helm upgrade --install promtail -n metrics grafana/promtail
- folgenden Befehl innerhalb des k8s/metrics/ Ordners ausführen: helm upgrade prometheus prometheus-community/kube-prometheus-stack --namespace=metrics --create-namespace --values metrics.yaml 



## Grafana öffnen
- grafana.budgetbook.me in den Browser eingeben
- Login aus der mectrics.yaml raussuchen (Name und Passsword)

## Metrics hinzufügen
- In k8s/dashboards/budgetbook_dashboard.json unter neue Metrics bei Grafana hinzufügen und json einfügen

## Loki hinzufügen
- Neue Metrics unter Grafana öffnen und Loki im gleichen Namespace hinzufügen, dabei die folgende URL verwenden http://loki:3100