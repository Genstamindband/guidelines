# Modifiy config.toml 
vi $BASE_DIR/$CHAIN_ID/config.toml
```
prometheus = true
prometheus_listen_addr = ":26660"
max_open_connections = 3
namespace = "namada_tm"
```
sudo service namadad restart  
curl -s localhost:26660  
# Setup the grafana key and install the grafana and prometheus packages
wget -q -O - https://packages.grafana.com/gpg.key | gpg --dearmor -o /usr/share/keyrings/grafana-archive-keyring.gpg  
sudo sh -c 'echo "deb [signed-by=/usr/share/keyrings/grafana-archive-keyring.gpg] https://packages.grafana.com/oss/deb stable main" > /etc/apt/sources.list.d/grafana.list'  
sudo apt update  
sudo apt-get install software-properties-common  
sudo apt-get update  
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"  
curl https://packages.grafana.com/gpg.key | sudo apt-key add -  
sudo apt-get update  
sudo apt-get install grafana  
sudo apt-get install -y prometheus prometheus-node-exporter prometheus-pushgateway prometheus-alertmanager  

# Modify Prometheus config
sudo vi /etc/prometheus/prometheus.yml
```
global:
  scrape_interval:     15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

  # Attach these labels to any time series or alerts when communicating with
  # external systems (federation, remote storage, Alertmanager).
  external_labels:
      monitor: 'example'

# Alertmanager configuration
alerting:
  alertmanagers:
  - static_configs:
    - targets: ['localhost:9093']

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: 'prometheus'

    # Override the global default and scrape targets from this job every 5 seconds.
    scrape_interval: 5s
    scrape_timeout: 5s

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ['localhost:9090']

  - job_name: node
    # If prometheus-node-exporter is installed, grab stats about the local
    # machine by default.
    static_configs:
      - targets: ['localhost:9100']

  - job_name: "namada"
    scrape_interval: 5s
    metrics_path: /
    static_configs:
      - targets: ['localhost:26660']
```
# Enable and start services
```
sudo chown -R prometheus:prometheus /usr/bin/prometheus
sudo chmod 777 /var/lib/prometheus/metrics2

sudo pkill -f prometheus
sudo pkill -f grafana

sudo systemctl daemon-reload

sudo systemctl enable grafana-server
sudo systemctl start grafana-server

sudo systemctl enable prometheus
sudo systemctl start prometheus

sudo systemctl enable prometheus-node-exporter
sudo systemctl start prometheus-node-exporter

sudo systemctl enable prometheus-pushgateway
sudo systemctl start prometheus-pushgateway

sudo systemctl enable prometheus-alertmanager
sudo systemctl start prometheus-alertmanager
ps -ef | grep grafana

sudo systemctl enable prometheus
sudo systemctl start prometheus
ps -ef | grep prometheus
```
# Setup grafana admin password
sudo grafana-cli admin reset-admin-password "<Password>"
# Access grafana instance
http://your_node_IP:3000/  
# 
