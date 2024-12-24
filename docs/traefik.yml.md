# `traefik.yml` documentation

The `traefik.yml` file is the main configuration file for Traefik.

## API and Dashboard Configuration
```yml
api:
  dashboard: true
  insecure: true
```
- **Dashboard:** Enables Traefik's web-based dashboard to monitor and configure its components in real-time.
- **Insecure:** Allows access to the dashboard without authentication or HTTPS. 
    - *This is not safe for production environments as it exposes sensitive information.*

## Enable Healthcheck
```yml
ping: {}
```
- **Ping:** Enables the health check endpoint for Traefik to confirm that it is running and healthy.

## Docker Configuration Backend
```yml
providers:
  docker:
    swarmMode: true
```
- **Providers:** Configures Traefik to use Docker as the source of dynamic configuration.
- **SwarmMode:** Activates support for Docker Swarm, allowing Traefik to discover services deployed in a Swarm cluster.

## Access Logging
```yml
accessLog: {}

# accessLog:
#   filters:    
#     statusCodes:
#       - "404"
#     retryAttempts: true
#     minDuration: "10ms"
```
- **AccessLog:** Enables logging of HTTP requests handled by Traefik, useful for auditing and debugging.

**Commented Example:**
- **Filters:**
    - **statusCodes:** Logs only specific HTTP status codes (e.g., 404).
    - **retryAttempts:** Logs requests where retries were attempted.
    - **minDuration:** Logs only requests that took longer than a specified duration.

## Traefik Logging
```yml
log:
  level: INFO
```

- **Log Level:** Sets the level of verbosity for Traefik logs. Levels include `DEBUG`, `PANIC`, `FATAL`, `ERROR`, `WARN`, and `INFO` (default: ERROR).

## Prometheus Metrics
```yml
metrics:
  prometheus:
    buckets:
      - 0.1
      - 0.3
      - 1.2
      - 5.0
```
- **Prometheus:** Configures Traefik to expose metrics in Prometheus format for monitoring.
- **Buckets:** Defines latency buckets (in seconds) for request durations.

## Entrypoints
```yml
entryPoints:
  web:
    address: ":80"
  websecure:
    address: ":443"
```

- **EntryPoints:** Defines listening ports for incoming traffic.
    - **web:** Listens on port 80 for HTTP traffic.
    - **websecure:** Listens on port 443 for HTTPS traffic.

## DNS Challenge 
```yml
# certificatesResolvers:
#   myresolver:
#     acme:
#       email: lorenzo@gmail.com
#       storage: acme.json
#       dnsChallenge:
#         provider: azure
#         delayBeforeCheck: 0
```

- **Certificates Resolvers:** Configures automatic HTTPS using Let's Encrypt via ACME protocol.
    - **DNS Challenge:** A method to verify domain ownership by updating DNS records.
    - **Email:** Email used for Let's Encrypt notifications.
    - **Storage:** Saves the certificates in the acme.json file.

*This section is commented out, meaning it's inactive.*

## Key Features Summary
- Enables Traefik’s dashboard and health check functionality.
- Configures Traefik to integrate with Docker in Swarm mode.
- Supports logging of requests and Prometheus metrics for monitoring.
- Defines HTTP and HTTPS entrypoints for incoming traffic.
- Provides a template for enabling Let's Encrypt with DNS challenge (currently disabled).

**Production Note:**
- Set up proper authentication for the dashboard (insecure: false).
- Configure HTTPS using the certificates resolver for secure traffic.