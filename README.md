# Container Monitoring Stack (Extended Version)

This project is a container-based monitoring stack using Prometheus, Grafana, Alertmanager, and Node Exporter.  
I extended the original project by adding NGINX monitoring.

## Key Enhancements 
I have implemented the following features to simulate a professional production environment:

- Nginx Web Server Integration: Added a live Nginx service to act as the monitored application.

- Application-Level Metrics: Integrated nginx-prometheus-exporter to track real-time traffic, including active connections and request rates.

- Custom Alerting Rules: Added specialized Prometheus rules in rules.yml to alert for Nginx downtime and traffic spikes.

- Optimized Configurations: Created a custom nginx.conf to expose the stub_status metrics securely.


## Stack Components
- **Prometheus** → Metrics collection
- **Grafana** → Visualization dashboards
- **Alertmanager** → Alert handling
- **Node Exporter** → Host-level metrics
- **NGINX** → Sample service to monitor
- **NGINX Exporter** → Exposes NGINX metrics for Prometheus
