# WireGuard Tunnel Connectivity Alerts

## Use-case
Suppose, you have multi-datacenter WireGuard setup, where you want to be alerted if WireGuard tunnel connectivity between them is lost or lagging. So, the problem is how can we be alerted in that case. 

In this guide, I'll be showing you how to setup the alert using:
1. [wireguard-prometheus-exporter](https://github.com/MindFlavor/prometheus_wireguard_exporter)
2. Prometheus + Grafana
3. Wireguard Grafana Dashboard with some customization.

## Step 1. Setup wireguard exporter on your Wireguard instance

### Step 1.1. Install wireguard prometheus exporter
NOTE: `yum` is used, you can use any package manager.
```
yum install cargo
cargo install prometheus_wireguard_exporter
install -m755 /root/.cargo/bin/prometheus_wireguard_exporter /usr/local/bin/
yum remove cargo
```

### Step 1.2. Install systemd service for the exporter
```
curl https://raw.githubusercontent.com/tuladhar/wireguard-alerts/main/prometheus-wireguard-exporter.service > /etc/systemd/system/prometheus-wireguard-exporter.service
```
And enable the exporter service.
```
systemctl enable --now prometheus-wireguard-exporter.service
```

### Step 1.3. Verify exporter works
```
curl localhost:9586/metrics
```

## Step 2. Configure Prometheus to scrape the exporter metrics
Add the following to `/etc/prometheus/prometheus.yaml`
```
  - job_name: wireguard-exporter
    static_configs:
    - labels:
        instance: my-wireguard-tunnel
      targets:
      - IP_OF_EXPORTER:9586
```

## Step 3. Import Wireguard Grafana Dashboard 
### Step 3.1. Import the following dashboard to Grafana as JSON
* https://raw.githubusercontent.com/tuladhar/wireguard-alerts/main/wireguard-grafana-dashboard.json

### Step 3.2. Duplicate the LastHandshake panel and customize
1. First, change the metrics to following:
```
time() - wireguard_latest_handshake_seconds{instance="my-wireguard-tunnel"}
```
2. Turn off the `Instant` metrics.
3. Choose the Graph Visualization
4. Goto `Field` next to Panel and change Unit to `short` from `From Now`

**You're all set.**

## Step 4. Create the alert
1. Click on Create Alert.
2. Set the condition as `WHEN avg() OF query(A, 1m, now) IS ABOVE 180`

Normally wireguard sends health check every 2 minutes, so it's safe to keep 3 minutes, i.e, 180 seconds as alerting threshold.
