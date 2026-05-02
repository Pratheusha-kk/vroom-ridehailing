# Ride-Hailing Platform

Local Docker Compose workspace for the VROOM ride-hailing microservices.

## Fresh Machine Setup

For a beginner-friendly setup from a new laptop, including clone commands, Docker startup, demo flow, logs, monitoring, and optional Minikube evidence, read:

- [VROOM VROOM Setup From Fresh Machine](docs/SETUP_FROM_FRESH_MACHINE.md)

## Services

- `rider-service` on `http://localhost:8081`
- `driver-service` on `http://localhost:3003`
- `trip-service` on `http://localhost:3002`
- `payment-service` on `http://localhost:8082`
- `vroom-gateway` on `http://localhost:8080`
- `vroom-ui` on `http://localhost:5173`
- Prometheus on `http://localhost:9090`
- Grafana on `http://localhost:3000`

## Bring All Services Down And Up

Run these commands from the workspace root:

```bash
docker compose down
docker compose up --build -d
docker compose ps
```

`docker compose down` stops and removes the running containers and network, but keeps named volumes such as the payment database volume. `docker compose up --build -d` rebuilds the service images and starts the full stack in the background.

## Clean Restart With Fresh Volumes

Use this only when you want to remove persisted Compose data, including the payment PostgreSQL volume:

```bash
docker compose down -v
docker compose up --build -d
docker compose ps
```

The driver and trip SQLite files are bind-mounted under their service `data/` directories, so remove those files separately if you need fully fresh local SQLite data.

## Health Checks

After the stack is running, verify the main services:

```bash
curl http://localhost:8081/actuator/health
curl http://localhost:3003/health
curl http://localhost:3002/health
curl http://localhost:8082/actuator/health
curl http://localhost:8080/health
curl http://localhost:5173/health
```
