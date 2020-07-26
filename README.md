# loki-promtail-with-existing-grafana


helm upgrade --install loki --namespace=monitoring loki/loki

helm upgrade --install promtail loki/promtail --namespace=monitoring --set "loki.serviceName=loki"


From grafana--> add loki datasource --> url--> http://loki:3100

go to explore and search for the log
