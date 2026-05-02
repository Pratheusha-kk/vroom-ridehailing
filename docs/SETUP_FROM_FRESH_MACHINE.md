# VROOM VROOM Setup From Fresh Machine

This guide is for someone who has no local project setup yet and needs to bring up the VROOM VROOM ride-hailing app for a demo recording.

## 1. Install Required Tools

Install these first:

- Git
- Docker Desktop or Rancher Desktop
- A browser such as Chrome
- A screen recorder such as QuickTime, OBS, or Zoom recording

Optional, only for the Kubernetes part of the recording:

- Minikube
- kubectl

Start Docker Desktop or Rancher Desktop before running any Docker commands.

## 2. Create A Working Folder

Open Terminal and run:

```bash
mkdir vroom-vroom-demo
cd vroom-vroom-demo
```

## 3. Clone The Orchestration Repo

Clone the repo that contains Docker Compose, Kubernetes manifests, Postman collection, and setup documentation:

```bash
git clone https://github.com/Pratheusha-kk/vroom-ridehailing.git
cd vroom-ridehailing
```

## 4. Clone The Service Repos Inside `vroom-ridehailing`

Run these commands from inside the `vroom-ridehailing` folder:

```bash
git clone https://github.com/ruchitabokde/Rider_Service.git
git clone https://github.com/Pratheusha-kk/driver-service.git
git clone https://github.com/AmruthaParwatikar/trip-service.git
git clone https://github.com/Sradhya1992/payment-service.git
git clone https://github.com/Sradhya1992/rating-feedback-service.git
git clone https://github.com/Pratheusha-kk/vroom-gateway.git
git clone https://github.com/Pratheusha-kk/vroom-ui.git
```

If a clone command fails with an authentication or permission error, ask the repo owner to give your GitHub account access.

After cloning, the folder should look like this:

```text
vroom-ridehailing/
  docker-compose.yml
  MINIKUBE_DEPLOYMENT.md
  README.md
  k8s/
  postman/
  Rider_Service/
  driver-service/
  trip-service/
  payment-service/
  rating-feedback-service/
  vroom-gateway/
  vroom-ui/
```

## 5. Start The Full App With Docker Compose

From inside the `vroom-ridehailing` folder, run:

```bash
docker compose down
docker compose up --build -d
docker compose ps
```

The first build can take several minutes.

Main URLs:

```text
VROOM UI:   http://localhost:5173
Gateway:    http://localhost:8080
Prometheus: http://localhost:9090
Grafana:    http://localhost:3000
```

Grafana login:

```text
Username: admin
Password: admin
```

## 6. Verify Health Checks

Run these commands:

```bash
curl http://localhost:8081/actuator/health
curl http://localhost:3003/health
curl http://localhost:3002/health
curl http://localhost:8082/actuator/health
curl http://localhost:8080/health
curl http://localhost:5173/health
```

For the recording, show that each service returns a healthy response.

## 7. Record The Main UI Demo

Open two browser tabs:

```text
http://localhost:5173/?role=driver
http://localhost:5173/?role=rider
```

Use this flow:

1. In the Driver tab, create or select a driver.
2. Set the driver city to `Bengaluru`.
3. Click `Set Active`.
4. In the Rider tab, create or select a rider.
5. Set the trip city to `Bengaluru`.
6. Enter pickup and drop, for example:

```text
Pickup: Indiranagar
Drop: MG Road
```

7. Click `Schedule Trip`.
8. Go to the Driver tab and accept the trip popup.
9. Go back to the Rider tab.
10. Click `Start`.
11. Click `Complete And Pay`.
12. Submit a rider rating.
13. Refresh and show ride history in both Rider and Driver tabs.

This proves the main workflow: rider, driver, active driver matching, trip request, acceptance, trip start, completion, payment, rating, and ride history.

## 8. Show Logs During Recording

Run these commands after completing the ride:

```bash
docker compose logs --tail=80 vroom-gateway
docker compose logs --tail=80 trip-service
docker compose logs --tail=80 driver-service
docker compose logs --tail=80 payment-service
docker compose logs --tail=80 rating-feedback-service
```

Mention that these logs show service-to-service communication through the gateway, trip service, driver service, payment service, and rating service.

## 9. Show Monitoring

Open Grafana:

```text
http://localhost:3000
```

Login with:

```text
admin / admin
```

Show the available ride-hailing dashboards, such as service health, endpoint usage, gateway, trip service, payment service, driver service, and rating service.

Also open Prometheus:

```text
http://localhost:9090
```

This proves monitoring is running.

## 10. Optional Minikube Recording

Do this only if Kubernetes evidence is required.

From inside the `vroom-ridehailing` folder, run:

```bash
minikube start
eval $(minikube docker-env)

docker build -t rider-service:latest ./Rider_Service
docker build -t driver-service:latest ./driver-service
docker build -t trip-service:latest ./trip-service
docker build -t payment-service:latest ./payment-service
docker build -t rating-feedback-service:latest ./rating-feedback-service
docker build -t vroom-gateway:latest ./vroom-gateway
docker build -t vroom-ui:latest ./vroom-ui

kubectl apply -f k8s/
kubectl get pods -n ride-hailing
kubectl get svc -n ride-hailing
```

Open the Kubernetes UI service:

```bash
minikube service vroom-ui -n ride-hailing --url
```

Copy the URL printed by Minikube and open it in the browser.

Useful Kubernetes evidence commands:

```bash
kubectl logs -n ride-hailing deployment/vroom-gateway --tail=80
kubectl logs -n ride-hailing deployment/trip-service --tail=80
kubectl logs -n ride-hailing deployment/driver-service --tail=80
```

## 11. Troubleshooting

If ports are already in use, stop old containers:

```bash
docker compose down
docker ps
```

If the app has old data and you want a fresh demo:

```bash
docker compose down -v
docker compose up --build -d
```

If the UI cannot find drivers, make sure at least one driver is set to active and the driver city matches the rider trip city.

If Docker builds fail, make sure all service repos were cloned inside the `vroom-ridehailing` folder and the folder names match exactly.

## 12. Suggested 10-15 Minute Recording Order

1. Show the cloned repo structure.
2. Start Docker Compose and show `docker compose ps`.
3. Run health checks.
4. Demonstrate the full UI ride workflow.
5. Show service logs.
6. Show Grafana and Prometheus.
7. Show Minikube pods and services if required.
8. Close with a short summary of the microservices and database-per-service design.
