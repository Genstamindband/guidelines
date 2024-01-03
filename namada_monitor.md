**Assuming you‘ve installed namada validator node. Be familiar with namada config and prometheus/grafana setup.**   
_Thanks for King Sirouk, refer to https://hackmd.io/@x8TnZOIJTLCGlasbXxkcMQ/HyvKv-3hn_

![image](https://github.com/aquariusluo/images/blob/master/namada_grafana.png)  
# Modifiy Namada config.toml 
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
# Add prometheus datasource
add "http://localhost:9090" to datasource in grafana  
# Import Dashboard via json
Replace CHAIN_ID with current chain-id  
Replace Validator_Address with $validator_addr  
validator_addr=$(curl -s localhost:26657/status | jq -r .result.validator_info.address)
```
{
    "annotations": {
      "list": [
        {
          "builtIn": 1,
          "datasource": {
            "type": "datasource",
            "uid": "grafana"
          },
          "enable": true,
          "hide": true,
          "iconColor": "rgba(0, 211, 255, 1)",
          "name": "Annotations & Alerts",
          "target": {
            "limit": 100,
            "matchAny": false,
            "tags": [],
            "type": "dashboard"
          },
          "type": "dashboard"
        }
      ]
    },
    "description": "",
    "editable": true,
    "fiscalYearStartMonth": 0,
    "gnetId": 18401,
    "graphTooltip": 0,
    "id": 4,
    "links": [],
    "liveNow": false,
    "panels": [
      {
        "collapsed": false,
        "datasource": {
          "type": "loki",
          "uid": "l2N9Iwb4z"
        },
        "gridPos": {
          "h": 1,
          "w": 24,
          "x": 0,
          "y": 0
        },
        "id": 64,
        "panels": [],
        "targets": [
          {
            "datasource": {
              "type": "loki",
              "uid": "l2N9Iwb4z"
            },
            "refId": "A"
          }
        ],
        "title": "$chain_id overview",
        "type": "row"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            },
            "unit": "locale"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 0,
          "y": 1
        },
        "hideTimeOverride": false,
        "id": 4,
        "links": [],
        "maxDataPoints": 100,
        "options": {
          "colorMode": "value",
          "graphMode": "none",
          "justifyMode": "auto",
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "lastNotNull"
            ],
            "fields": "",
            "values": false
          },
          "textMode": "auto"
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_latest_block_height{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "hide": false,
            "instant": true,
            "interval": "30s",
            "intervalFactor": 1,
            "refId": "A"
          }
        ],
        "title": "Block Height",
        "type": "stat"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            },
            "unit": "locale"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 6,
          "y": 1
        },
        "hideTimeOverride": false,
        "id": 40,
        "links": [],
        "maxDataPoints": 100,
        "options": {
          "colorMode": "value",
          "graphMode": "none",
          "justifyMode": "auto",
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "lastNotNull"
            ],
            "fields": "",
            "values": false
          },
          "textMode": "auto"
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_total_txs{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "interval": "30s",
            "intervalFactor": 1,
            "range": true,
            "refId": "A"
          }
        ],
        "title": "Total Transactions",
        "type": "stat"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 2,
            "mappings": [
              {
                "options": {
                  "match": "null",
                  "result": {
                    "text": "N/A"
                  }
                },
                "type": "special"
              }
            ],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            },
            "unit": "s"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 12,
          "y": 1
        },
        "hideTimeOverride": true,
        "id": 39,
        "links": [],
        "maxDataPoints": 100,
        "options": {
          "colorMode": "value",
          "graphMode": "area",
          "justifyMode": "auto",
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "mean"
            ],
            "fields": "",
            "values": false
          },
          "textMode": "auto"
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_block_interval_seconds_count{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "interval": "30s",
            "intervalFactor": 1,
            "range": true,
            "refId": "A"
          }
        ],
        "timeFrom": "1h",
        "title": "Avg Block Time",
        "type": "stat"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            },
            "unit": "short"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 18,
          "y": 1
        },
        "hideTimeOverride": false,
        "id": 47,
        "links": [],
        "maxDataPoints": 100,
        "options": {
          "colorMode": "value",
          "graphMode": "area",
          "justifyMode": "auto",
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "lastNotNull"
            ],
            "fields": "",
            "values": false
          },
          "textMode": "auto"
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_validator_power{chain_id=\"$chain_id\", validator_address=\"$validator\"}",
            "format": "time_series",
            "instant": false,
            "interval": "30s",
            "intervalFactor": 1,
            "refId": "A"
          }
        ],
        "title": "Bonded Tokens",
        "type": "stat"
      },
      {
        "aliasColors": {
          "Height for last 3 hours": "#447ebc",
          "Total Transactions for last 3 hours": "#ef843c"
        },
        "bars": false,
        "dashLength": 10,
        "dashes": false,
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "decimals": 0,
        "fieldConfig": {
          "defaults": {
            "links": []
          },
          "overrides": []
        },
        "fill": 3,
        "fillGradient": 0,
        "gridPos": {
          "h": 9,
          "w": 12,
          "x": 0,
          "y": 5
        },
        "hiddenSeries": false,
        "hideTimeOverride": false,
        "id": 15,
        "legend": {
          "alignAsTable": true,
          "avg": false,
          "current": true,
          "max": true,
          "min": true,
          "rightSide": false,
          "show": true,
          "sideWidth": 350,
          "total": false,
          "values": true
        },
        "lines": true,
        "linewidth": 1,
        "links": [],
        "nullPointMode": "null",
        "options": {
          "alertThreshold": true
        },
        "percentage": false,
        "pluginVersion": "10.0.3",
        "pointradius": 5,
        "points": false,
        "renderer": "flot",
        "seriesOverrides": [],
        "spaceLength": 10,
        "stack": false,
        "steppedLine": false,
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_validators{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": false,
            "interval": "",
            "intervalFactor": 1,
            "legendFormat": "Active",
            "refId": "A"
          },
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_missing_validators{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "interval": "",
            "intervalFactor": 1,
            "legendFormat": "Missing",
            "range": true,
            "refId": "B"
          },
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_byzantine_validators{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "intervalFactor": 1,
            "legendFormat": "Byzantine",
            "range": true,
            "refId": "C"
          }
        ],
        "thresholds": [],
        "timeRegions": [],
        "title": "Validators",
        "tooltip": {
          "shared": true,
          "sort": 0,
          "value_type": "individual"
        },
        "type": "graph",
        "xaxis": {
          "mode": "time",
          "show": true,
          "values": []
        },
        "yaxes": [
          {
            "decimals": 0,
            "format": "locale",
            "label": "",
            "logBase": 1,
            "min": "0",
            "show": true
          },
          {
            "format": "none",
            "logBase": 1,
            "show": false
          }
        ],
        "yaxis": {
          "align": false
        }
      },
      {
        "aliasColors": {
          "Height for last 3 hours": "#447ebc",
          "Total Transactions for last 3 hours": "#ef843c"
        },
        "bars": false,
        "dashLength": 10,
        "dashes": false,
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "fieldConfig": {
          "defaults": {
            "links": []
          },
          "overrides": []
        },
        "fill": 3,
        "fillGradient": 0,
        "gridPos": {
          "h": 9,
          "w": 12,
          "x": 12,
          "y": 5
        },
        "hiddenSeries": false,
        "hideTimeOverride": false,
        "id": 48,
        "legend": {
          "alignAsTable": true,
          "avg": false,
          "current": true,
          "max": true,
          "min": true,
          "rightSide": false,
          "show": true,
          "sideWidth": 350,
          "total": false,
          "values": true
        },
        "lines": true,
        "linewidth": 1,
        "links": [],
        "nullPointMode": "null",
        "options": {
          "alertThreshold": true
        },
        "percentage": false,
        "pluginVersion": "10.0.3",
        "pointradius": 5,
        "points": false,
        "renderer": "flot",
        "seriesOverrides": [],
        "spaceLength": 10,
        "stack": false,
        "steppedLine": false,
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_validators_power{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": false,
            "interval": "",
            "intervalFactor": 1,
            "legendFormat": "Online",
            "refId": "A"
          },
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_missing_validators_power{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "interval": "",
            "intervalFactor": 1,
            "legendFormat": "Missing",
            "range": true,
            "refId": "B"
          },
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_byzantine_validators_power{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "intervalFactor": 1,
            "legendFormat": "Byzantine",
            "range": true,
            "refId": "C"
          }
        ],
        "thresholds": [],
        "timeRegions": [],
        "title": "Voting Power",
        "tooltip": {
          "shared": true,
          "sort": 0,
          "value_type": "individual"
        },
        "type": "graph",
        "xaxis": {
          "mode": "time",
          "show": true,
          "values": []
        },
        "yaxes": [
          {
            "decimals": 0,
            "format": "short",
            "label": "",
            "logBase": 1,
            "min": "0",
            "show": true
          },
          {
            "format": "none",
            "logBase": 1,
            "show": false
          }
        ],
        "yaxis": {
          "align": false
        }
      },
      {
        "aliasColors": {
          "Height for last 3 hours": "#447ebc",
          "Total Transactions for last 3 hours": "#ef843c"
        },
        "bars": false,
        "dashLength": 10,
        "dashes": false,
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "fieldConfig": {
          "defaults": {
            "links": []
          },
          "overrides": []
        },
        "fill": 3,
        "fillGradient": 0,
        "gridPos": {
          "h": 5,
          "w": 12,
          "x": 0,
          "y": 14
        },
        "hiddenSeries": false,
        "hideTimeOverride": false,
        "id": 49,
        "legend": {
          "alignAsTable": false,
          "avg": true,
          "current": false,
          "max": true,
          "min": false,
          "rightSide": false,
          "show": true,
          "total": false,
          "values": true
        },
        "lines": true,
        "linewidth": 1,
        "links": [],
        "nullPointMode": "null",
        "options": {
          "alertThreshold": true
        },
        "percentage": false,
        "pluginVersion": "10.0.3",
        "pointradius": 5,
        "points": false,
        "renderer": "flot",
        "seriesOverrides": [],
        "spaceLength": 10,
        "stack": false,
        "steppedLine": false,
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_block_size_bytes{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": false,
            "interval": "",
            "intervalFactor": 1,
            "legendFormat": "Block Size",
            "refId": "A"
          }
        ],
        "thresholds": [],
        "timeRegions": [],
        "title": "Block Size",
        "tooltip": {
          "shared": true,
          "sort": 0,
          "value_type": "individual"
        },
        "type": "graph",
        "xaxis": {
          "mode": "time",
          "show": true,
          "values": []
        },
        "yaxes": [
          {
            "decimals": 2,
            "format": "bytes",
            "label": "",
            "logBase": 1,
            "min": "0",
            "show": true
          },
          {
            "format": "none",
            "logBase": 1,
            "show": false
          }
        ],
        "yaxis": {
          "align": false
        }
      },
      {
        "aliasColors": {
          "Height for last 3 hours": "#447ebc",
          "Total Transactions for last 3 hours": "#ef843c"
        },
        "bars": false,
        "dashLength": 10,
        "dashes": false,
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "decimals": 0,
        "fieldConfig": {
          "defaults": {
            "links": []
          },
          "overrides": []
        },
        "fill": 3,
        "fillGradient": 0,
        "gridPos": {
          "h": 5,
          "w": 12,
          "x": 12,
          "y": 14
        },
        "hiddenSeries": false,
        "hideTimeOverride": false,
        "id": 50,
        "legend": {
          "alignAsTable": false,
          "avg": true,
          "current": false,
          "max": true,
          "min": false,
          "rightSide": false,
          "show": true,
          "total": true,
          "values": true
        },
        "lines": true,
        "linewidth": 1,
        "links": [],
        "nullPointMode": "null",
        "options": {
          "alertThreshold": true
        },
        "percentage": false,
        "pluginVersion": "10.0.3",
        "pointradius": 5,
        "points": false,
        "renderer": "flot",
        "seriesOverrides": [],
        "spaceLength": 10,
        "stack": false,
        "steppedLine": false,
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_consensus_num_txs{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": false,
            "interval": "",
            "intervalFactor": 1,
            "legendFormat": "Transactions",
            "refId": "A"
          }
        ],
        "thresholds": [],
        "timeRegions": [],
        "title": "Transactions",
        "tooltip": {
          "shared": true,
          "sort": 0,
          "value_type": "individual"
        },
        "type": "graph",
        "xaxis": {
          "mode": "time",
          "show": true,
          "values": []
        },
        "yaxes": [
          {
            "decimals": 0,
            "format": "short",
            "label": "",
            "logBase": 1,
            "min": "0",
            "show": true
          },
          {
            "format": "none",
            "logBase": 1,
            "show": false
          }
        ],
        "yaxis": {
          "align": false
        }
      },
      {
        "collapsed": false,
        "datasource": {
          "type": "loki",
          "uid": "l2N9Iwb4z"
        },
        "gridPos": {
          "h": 1,
          "w": 24,
          "x": 0,
          "y": 19
        },
        "id": 55,
        "panels": [],
        "repeat": "instance",
        "targets": [
          {
            "datasource": {
              "type": "loki",
              "uid": "l2N9Iwb4z"
            },
            "refId": "A"
          }
        ],
        "title": "instance overview: $instance",
        "type": "row"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "max": 20,
            "min": 0,
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "#e24d42",
                  "value": null
                },
                {
                  "color": "#ef843c",
                  "value": 2
                },
                {
                  "color": "#7eb26d",
                  "value": 5
                }
              ]
            },
            "unit": "short"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 0,
          "y": 20
        },
        "hideTimeOverride": true,
        "id": 53,
        "links": [],
        "options": {
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "last"
            ],
            "fields": "",
            "values": false
          },
          "showThresholdLabels": false,
          "showThresholdMarkers": true
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_p2p_peers{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": true,
            "interval": "30s",
            "intervalFactor": 1,
            "refId": "A"
          }
        ],
        "title": "Connected Peers",
        "type": "gauge"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "max": 50,
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "#7eb26d",
                  "value": null
                },
                {
                  "color": "#ef843c",
                  "value": 10
                },
                {
                  "color": "#e24d42",
                  "value": 20
                }
              ]
            },
            "unit": "short"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 6,
          "y": 20
        },
        "hideTimeOverride": true,
        "id": 56,
        "links": [],
        "options": {
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "last"
            ],
            "fields": "",
            "values": false
          },
          "showThresholdLabels": false,
          "showThresholdMarkers": true
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_mempool_size{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": true,
            "interval": "30s",
            "intervalFactor": 1,
            "refId": "A"
          }
        ],
        "title": "Unconfirmed Transactions",
        "type": "gauge"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            },
            "unit": "short"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 12,
          "y": 20
        },
        "hideTimeOverride": true,
        "id": 60,
        "links": [],
        "maxDataPoints": 100,
        "options": {
          "colorMode": "value",
          "graphMode": "none",
          "justifyMode": "auto",
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "lastNotNull"
            ],
            "fields": "",
            "values": false
          },
          "textMode": "auto"
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_mempool_failed_txs{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": true,
            "interval": "30s",
            "intervalFactor": 1,
            "refId": "A"
          }
        ],
        "title": "Failed Transactions",
        "type": "stat"
      },
      {
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "description": "",
        "fieldConfig": {
          "defaults": {
            "color": {
              "mode": "thresholds"
            },
            "decimals": 0,
            "mappings": [],
            "thresholds": {
              "mode": "absolute",
              "steps": [
                {
                  "color": "green",
                  "value": null
                },
                {
                  "color": "red",
                  "value": 80
                }
              ]
            },
            "unit": "locale"
          },
          "overrides": []
        },
        "gridPos": {
          "h": 4,
          "w": 6,
          "x": 18,
          "y": 20
        },
        "hideTimeOverride": true,
        "id": 61,
        "links": [],
        "maxDataPoints": 100,
        "options": {
          "colorMode": "value",
          "graphMode": "none",
          "justifyMode": "auto",
          "orientation": "horizontal",
          "reduceOptions": {
            "calcs": [
              "lastNotNull"
            ],
            "fields": "",
            "values": false
          },
          "textMode": "auto"
        },
        "pluginVersion": "10.0.3",
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_mempool_recheck_times{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "instant": true,
            "interval": "30s",
            "intervalFactor": 1,
            "refId": "A"
          }
        ],
        "title": "Recheck Times",
        "type": "stat"
      },
      {
        "aliasColors": {},
        "bars": true,
        "dashLength": 10,
        "dashes": false,
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "fieldConfig": {
          "defaults": {
            "links": []
          },
          "overrides": []
        },
        "fill": 1,
        "fillGradient": 0,
        "gridPos": {
          "h": 9,
          "w": 12,
          "x": 0,
          "y": 24
        },
        "hiddenSeries": false,
        "id": 59,
        "legend": {
          "avg": false,
          "current": false,
          "max": false,
          "min": false,
          "rightSide": false,
          "show": false,
          "total": false,
          "values": false
        },
        "lines": false,
        "linewidth": 1,
        "links": [],
        "nullPointMode": "null",
        "options": {
          "alertThreshold": true
        },
        "percentage": false,
        "pluginVersion": "10.0.3",
        "pointradius": 5,
        "points": false,
        "renderer": "flot",
        "seriesOverrides": [],
        "spaceLength": 10,
        "stack": false,
        "steppedLine": false,
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_p2p_peer_receive_bytes_total{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "intervalFactor": 1,
            "legendFormat": "{{peer_id}}",
            "range": true,
            "refId": "A"
          }
        ],
        "thresholds": [],
        "timeRegions": [],
        "title": "Total Network Input",
        "tooltip": {
          "shared": false,
          "sort": 0,
          "value_type": "individual"
        },
        "type": "graph",
        "xaxis": {
          "mode": "series",
          "show": false,
          "values": [
            "current"
          ]
        },
        "yaxes": [
          {
            "format": "bytes",
            "logBase": 1,
            "show": true
          },
          {
            "format": "short",
            "logBase": 1,
            "show": false
          }
        ],
        "yaxis": {
          "align": false
        }
      },
      {
        "aliasColors": {},
        "bars": true,
        "dashLength": 10,
        "dashes": false,
        "datasource": {
          "type": "prometheus",
          "uid": "$DS"
        },
        "fieldConfig": {
          "defaults": {
            "links": []
          },
          "overrides": []
        },
        "fill": 1,
        "fillGradient": 0,
        "gridPos": {
          "h": 9,
          "w": 12,
          "x": 12,
          "y": 24
        },
        "hiddenSeries": false,
        "id": 58,
        "legend": {
          "avg": false,
          "current": false,
          "max": false,
          "min": false,
          "rightSide": false,
          "show": false,
          "total": false,
          "values": false
        },
        "lines": false,
        "linewidth": 1,
        "links": [],
        "nullPointMode": "null",
        "options": {
          "alertThreshold": true
        },
        "percentage": false,
        "pluginVersion": "10.0.3",
        "pointradius": 5,
        "points": false,
        "renderer": "flot",
        "seriesOverrides": [],
        "spaceLength": 10,
        "stack": false,
        "steppedLine": false,
        "targets": [
          {
            "datasource": {
              "uid": "$DS"
            },
            "editorMode": "code",
            "expr": "namada_tm_p2p_peer_send_bytes_total{chain_id=\"$chain_id\"}",
            "format": "time_series",
            "intervalFactor": 1,
            "legendFormat": "{{peer_id}}",
            "range": true,
            "refId": "A"
          }
        ],
        "thresholds": [],
        "timeRegions": [],
        "title": "Total Network Output",
        "tooltip": {
          "shared": false,
          "sort": 0,
          "value_type": "individual"
        },
        "type": "graph",
        "xaxis": {
          "mode": "series",
          "show": false,
          "values": [
            "current"
          ]
        },
        "yaxes": [
          {
            "format": "bytes",
            "logBase": 1,
            "show": true
          },
          {
            "format": "short",
            "logBase": 1,
            "show": false
          }
        ],
        "yaxis": {
          "align": false
        }
      }
    ],
    "refresh": "5s",
    "schemaVersion": 38,
    "style": "dark",
    "tags": [
      "Blockchain",
      "Namada"
    ],
    "templating": {
      "list": [
        {
          "current": {
            "selected": false,
            "text": "Prometheus",
            "value": "Prometheus"
          },
          "hide": 0,
          "includeAll": false,
          "label": "Datasource",
          "multi": false,
          "name": "DS",
          "options": [],
          "query": "prometheus",
          "queryValue": "",
          "refresh": 1,
          "regex": "",
          "skipUrlSync": false,
          "type": "datasource"
        },
        {
          "current": {
            "selected": true,
            "text": "CHAIN_ID",
            "value": "CHAIN_ID"
          },
          "hide": 0,
          "label": "Chain ID",
          "name": "chain_id",
          "options": [
            {
              "selected": true,
              "text": "CHAIN_ID",
              "value": "CHAIN_ID"
            }
          ],
          "query": "CHAIN_ID",
          "skipUrlSync": false,
          "type": "textbox"
        },
        {
          "allValue": "",
          "current": {
            "isNone": true,
            "selected": false,
            "text": "None",
            "value": ""
          },
          "datasource": {
            "uid": "$DS"
          },
          "definition": "label_values(tendermint_consensus_height{chain_id=\"$chain_id\"}, instance)",
          "hide": 0,
          "includeAll": false,
          "label": "Instance",
          "multi": false,
          "name": "instance",
          "options": [],
          "query": {
            "query": "label_values(tendermint_consensus_height{chain_id=\"$chain_id\"}, instance)",
            "refId": "Prometheus-instance-Variable-Query"
          },
          "refresh": 1,
          "regex": "",
          "skipUrlSync": false,
          "sort": 5,
          "tagValuesQuery": "",
          "tagsQuery": "",
          "type": "query",
          "useTags": false
        },
        {
          "current": {
            "selected": true,
            "text": "Validator_Address",
            "value": "Validator_Address"
          },
          "hide": 0,
          "name": "validator",
          "options": [
            {
              "selected": true,
              "text": "Validator_Address",
              "value": "Validator_Address"
            }
          ],
          "query": "Validator_Address",
          "skipUrlSync": false,
          "type": "textbox"
        }
      ]
    },
    "time": {
      "from": "now-5m",
      "to": "now"
    },
    "timepicker": {
      "refresh_intervals": [
        "5s",
        "10s",
        "30s",
        "1m",
        "5m",
        "15m",
        "30m",
        "1h",
        "2h",
        "1d"
      ],
      "time_options": [
        "5m",
        "15m",
        "1h",
        "6h",
        "12h",
        "24h",
        "2d",
        "7d",
        "30d"
      ]
    },
    "timezone": "",
    "title": "Namada",
    "uid": "UJyurCTWz",
    "version": 6,
    "weekStart": ""
  }
```
