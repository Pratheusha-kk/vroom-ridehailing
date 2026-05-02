# Minikube Deployment Runbook

This workspace contains four independently versioned microservice repositories:

- `Rider_Service` - Spring Boot Rider Service, H2 database-per-service
- `driver-service` - Node.js Driver Service, SQLite database-per-service
- `trip-service` - Node.js Trip Service, SQLite database-per-service
- `payment-service` - Spring Boot Payment Service, PostgreSQL database-per-service
- `vroom-gateway` - Node.js API Gateway for UI-to-service calls
- `vroom-ui` - Node.js static UI service for VROOM VROOM

The Kubernetes manifests in `k8s/` deploy the services into a local `ride-hailing` namespace. Services use Kubernetes DNS for loose coupling:

- Trip Service calls Driver Service through `http://driver-service:3003`
- Trip Service calls Payment Service through `http://payment-service:8082`
- Payment Service calls Trip Service through `http://trip-service:3002/v1/trips`
- VROOM UI calls VROOM Gateway through its UI-server proxy

## Build Images For Minikube

Start Minikube, then build each service image inside Minikube's Docker daemon:

```bash
minikube start
eval $(minikube docker-env)

docker build -t rider-service:latest ./Rider_Service
docker build -t driver-service:latest ./driver-service
docker build -t trip-service:latest ./trip-service
docker build -t payment-service:latest ./payment-service
docker build -t vroom-gateway:latest ./vroom-gateway
docker build -t vroom-ui:latest ./vroom-ui
```

The manifests use `imagePullPolicy: Never` so Kubernetes uses these local Minikube images.

## Deploy

```bash
kubectl apply -f k8s/
kubectl get pods -n ride-hailing
kubectl get svc -n ride-hailing
```

Wait until the service pods are `Running` and ready.

## Access Services

Use Minikube service URLs:

```bash
minikube service rider-service -n ride-hailing --url
minikube service trip-service -n ride-hailing --url
minikube service driver-service -n ride-hailing --url
minikube service payment-service -n ride-hailing --url
minikube service vroom-gateway -n ride-hailing --url
minikube service vroom-ui -n ride-hailing --url
```

Or use the fixed NodePorts:

- Rider Service: `30081`
- Trip Service: `30082`
- Driver Service: `30083`
- Payment Service: `30084`
- VROOM Gateway: `30080`
- VROOM UI: `30085`

## Smoke Tests

```bash
curl "$(minikube service rider-service -n ride-hailing --url)/actuator/health"
curl "$(minikube service driver-service -n ride-hailing --url)/health"
curl "$(minikube service trip-service -n ride-hailing --url)/health"
curl "$(minikube service payment-service -n ride-hailing --url)/actuator/health"
curl "$(minikube service vroom-gateway -n ride-hailing --url)/health"
curl "$(minikube service vroom-ui -n ride-hailing --url)/health"
```

## Clean Up

```bash
kubectl delete -f k8s/
eval $(minikube docker-env -u)
```
