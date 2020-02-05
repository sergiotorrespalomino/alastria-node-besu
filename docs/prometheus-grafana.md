# Prometheus and Grafana Monitoring
## Prometheus setup

> :warning: Note: Option **metrics-enabled=true** must be included in besu node’s config.toml.

### Get Prometheus Docker Image
```
$ docker image pull prom/prometheus:v2.15.2
$ docker image tag prom/prometheus:v2.15.2 prometheus
```

### Set Prometheus configuration:
```
$ cd alastria-red-b/besu-node
$ mkdir prometheus
$ vi /prometheus/prometheus.yml
```

### Run Prometheus.
```
$ docker container run -d --network host --name prometheus -v `pwd`/prometheus:/etc/prometheus/ prometheus
```
Prometheus will be running at http://localhost:9090

## Grafana setup

Note: Prometheus must be running.

### Get and run Grafana in your monitoring PC
```
$ docker run -d --name=grafana -p 3000:3000 grafana/grafana
```

### Grafana configuration
- Go to localhost:3000 to see Grafana´s interface.
- Login with `admin` user and `admin` password.
- Update your password.
- Set a new Prometheus Data Source with your node IP and port 9090. (http://<node_ip>:9090)
- Import Besu’s Dashboard (dashboard_id: 10273) and use the previously created Prometeus Data source as data soure for this dashboard.
- In a few minutes it should start monitoring.

