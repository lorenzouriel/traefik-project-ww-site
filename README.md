# Traefik Save Walter White Stack

This project sets up a monitoring stack using Traefik, Prometheus, Grafana, and custom services. It leverages Docker Swarm to manage the deployment of the stack.

## Features
- **Traefik** as the reverse proxy and load balancer.
- **Prometheus** for metrics collection and monitoring.
- **Grafana** for data visualization and dashboards.
- **Custom Error Pages** for user experience.
- **SaveWW Service** Save Walter White site with middleware configurations for authentication, rate limiting, and retries.

## Prerequisites
1. Docker Engine installed with Swarm mode enabled.
2. Access to the command line to run `docker` commands.

## Services Overview
| Service     | Description                         | Hostname                  | Port  |
|-------------|-------------------------------------|---------------------------|-------|
| **Traefik** | Reverse proxy and load balancer     | `localhost:8080`   | 80, 443, 8080 |
| **Prometheus** | Metrics collection and monitoring | `prometheus.localhost`    | 9090  |
| **Grafana** | Data visualization dashboards       | `grafana.localhost`       | 3000  |
| **SaveWW**  | Custom service                      | `saveww.localhost`        | 80    |
| **Error**   | Custom error pages                 | `error.localhost`         | 80    |

## Deployment Instructions

1. Clone this repository:
```bash
   git clone https://github.com/lorenzouriel/traefik-project-ww-site.git
   cd traefik-project-ww-site
```

2. Deploy the stack using Docker Swarm:
```bash
docker stack deploy -c docker-compose.yml traefik
```

3. Verify the services are running:
```bash
docker service ls
```

4. Scale the `saveww` site
```bash
docker service scale traefik_saveww=4
```

5. To remove the stack and associated resources:
```bash
docker stack rm traefik
```

## Architecture

![architecture]
