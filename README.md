# loki-promtail-with-existing-grafana

## To monitor kubernetes cluster:
~~~
helm upgrade --install loki --namespace=monitoring loki/loki

helm upgrade --install promtail loki/promtail --namespace=monitoring --set "loki.serviceName=loki"


From grafana--> add loki datasource --> url--> http://loki:3100

go to explore and search for the log
~~~
## To monitor bare metal or centos:

~~~
cd /usr/local/bin
sudo curl -fSL -o promtail.gz "https://github.com/grafana/loki/releases/download/v1.4.1/promtail-linux-amd64.zip"
gunzip promtail.gz
chmod a+x promtail
~~~
## Create a file to monitor journal log:

vim config-promtail.yml
~~~
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://118.67.215.43:30283/loki/api/v1/push

scrape_configs:
  - job_name: journal
    journal:
      max_age: 12h
      path: /var/log/journal
      labels:
        job: systemd-journal
        host: test-bed
    relabel_configs:
      - source_labels: ['__journal__systemd_unit']
        target_label: 'unit'
~~~

## Create a file to monitor All log:   
vim config-promtail-all-log.yml
~~~
server:
  http_listen_port: 9082
  grpc_listen_port: 0

positions:
  filename: /var/log/positions3.yaml # This location needs to be writeable by promtail.

client:
  url: http://118.67.215.43:30283/loki/api/v1/push

scrape_configs:
 - job_name: system
   pipeline_stages:
   static_configs:
   - targets:
      - localhost
     labels:
      job: varlogs  # A `job` label is fairly standard in prometheus and useful for linking metrics and logs.
      host: test-bed # A `host` label will help identify logs from this machine vs others
      __path__: /var/log/*.log  # The path matching uses a third party library: https://github.com/bmatcuk/doublestar
~~~

