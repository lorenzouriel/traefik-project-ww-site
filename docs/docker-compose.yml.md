# `docker-compose.yml` documentation

This `docker-compose.yml` file sets up a monitoring and reverse proxy system using Traefik, Prometheus, Grafana, and custom services. Below is a detailed explanation of the configuration:

## Services

### Traefik
```yml
  traefik:
    image: traefik:v2.3
    networks:
      - traefik
      - inbound
    ports:
      - "80:80"
      - "443:443"
      - "8080:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - ./traefik.yml:/etc/traefik/traefik.yml
    healthcheck:
      test: ["CMD", "traefik", "healthcheck"]
      interval: 10s
      timeout: 2s
      retries: 3
      start_period: 5s
```

- **Image:** Traefik v2.3 is used as the reverse proxy and load balancer.
- **etworks:** Connected to traefik (internal) and inbound (external-facing) overlay networks.
- **Ports:**
    - **80:** Handles HTTP traffic.
    - **443:** Handles HTTPS traffic.
    - **8080:** Exposes the Traefik dashboard and API.
- **Volumes:**
  - **docker.sock:** Allows Traefik to interact with the Docker engine and detect services.
  - **traefik.yml:** Traefik's configuration file.
- **healthcheck:** for the Traefik service to ensure it is running and healthy.

### Prometheus
```yml
  prometheus:
    image: prom/prometheus:v2.22.1
    networks:
      - inbound
    volumes:
      - ./prometheus/:/etc/prometheus/
      - prometheus_data:/prometheus
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
      - '--storage.tsdb.path=/prometheus'
      - '--web.console.libraries=/usr/share/prometheus/console_libraries'
      - '--web.console.templates=/usr/share/prometheus/consoles'
    deploy:
      labels:
        - "traefik.http.routers.prometheus.rule=Host(`prometheus.localhost`)"
        - "traefik.http.routers.prometheus.service=prometheus"
        - "traefik.http.routers.prometheus.entrypoints=web"
        - "traefik.http.services.prometheus.loadbalancer.server.port=9090"
        - "traefik.docker.network=inbound"
      placement:
        constraints:
          - node.role==manager
      restart_policy:
        condition: on-failure
```

- **Image:** Prometheus for metrics collection and monitoring.
- **Volumes:**
  - Prometheus configuration files and data persistence.
- **Command:** Specifies paths for configuration and storage.
- **Deploy:**
  - **Labels:** Configure Traefik to route `prometheus.localhost` to this service on port 9090.
  - **Placement:** Restricts the service to manager nodes in the Docker Swarm.
  - **Restart Policy:** Restarts the service on failure.

### Grafana
```yml
  grafana:
    image: grafana/grafana:7.3.1
    networks:
      - inbound
    depends_on:
      - prometheus
    volumes:
      - grafana_data:/var/lib/grafana
      - ./grafana/provisioning/:/etc/grafana/provisioning/
    env_file:
      - ./grafana/config.monitoring
    user: "104"
    deploy:
      labels:
        - "traefik.http.routers.grafana.rule=Host(`grafana.localhost`)"
        - "traefik.http.routers.grafana.service=grafana"
        - "traefik.http.routers.grafana.entrypoints=web"
        - "traefik.http.services.grafana.loadbalancer.server.port=3000"
        - "traefik.docker.network=inbound"
      placement:
        constraints:
          - node.role == manager
      restart_policy:
        condition: on-failure
```

- **Image:** Grafana for visualization and dashboards.
- **Depends On:** Starts only after Prometheus is running.
- **Deploy:**
  - **Labels:** Routes `grafana.localhost` to Grafana (port 3000).

### SaveWW
```yml
  saveww:
    image: lorenzouriel147/save-ww:v1
    networks:
      - inbound
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.saveww.rule=Host(`saveww.localhost`)"
        - "traefik.http.routers.saveww.service=saveww"
        - "traefik.http.routers.saveww.entrypoints=web"
        - "traefik.http.routers.saveww.middlewares=saveww-auth,saveww-compress,saveww-errorpages,saveww-ratelimit,saveww-retry"
        - "traefik.http.services.saveww.loadbalancer.server.port=80"
        - "traefik.http.middlewares.saveww-auth.basicauth.users=whoisheisenberg:$$apr1$$oP8nSKa.$$TITERRdBsJCxX7VkfB8WL1" # whoisheisenberg:walterwhite
        - "traefik.http.middlewares.saveww-compress.compress=true"
        - "traefik.http.middlewares.saveww-errorpages.errors.status=400-599"
        - "traefik.http.middlewares.saveww-errorpages.errors.service=error"
        - "traefik.http.middlewares.saveww-errorpages.errors.query=/{status}.html"
        - "traefik.http.middlewares.saveww-ratelimit.ratelimit.average=2"
        - "traefik.http.middlewares.saveww-retry.retry.attempts=4"
        - "traefik.docker.network=inbound"
```

- **Image:** A custom service (`save-ww`) configured with several middlewares:
  - **BasicAuth:** Protects routes with username and password.
  - **Compress:** Enables response compression.
  - **Error Pages:** Routes errors to a dedicated service (`error`).
  - **Rate Limiting:** Limits requests to 2 per second.
  - **Retry:** Retries failed requests up to 4 times.

### Error Pages
```yml
  error:
    image: guillaumebriday/traefik-custom-error-pages
    networks:
      - inbound
    deploy:
      labels:
        - "traefik.enable=true"
        - "traefik.http.routers.error.rule=Host(`error.localhost`)"
        - "traefik.http.routers.error.service=error"
        - "traefik.http.services.error.loadbalancer.server.port=80"
        - "traefik.http.routers.error.entrypoints=web"
        - "traefik.docker.network=inbound"
```

- **Image:**Displays custom error pages for failed requests.

### Networks
```yml
networks:
  traefik:
    driver: overlay
    name: traefik
  inbound:
    driver: overlay
    name: inbound
```

- **Overlay Networks:** Ensure communication between containers across Swarm nodes.

### Volumes
```yml
volumes:
  prometheus_data: {}
  grafana_data: {}
```

- **Persistent Storage:** Keeps Prometheus and Grafana data across container restarts.