# E-Commerce Microservices

This repository contains a simple e-commerce example split into several Spring Boot microservices:

- **product-service** – CRUD API for products (port 8081)
- **order-service** – CRUD API for orders (port 8082)
- **user-service** – CRUD API for users (port 8083)
- **discovery-service** – Eureka server (port 8761)
- **api-gateway** – Spring Cloud Gateway (port 8080)

## Running Locally

Each module is a standalone Spring Boot application. You can start them in any order, but `discovery-service` should run before others if you want them to register with Eureka.

```bash
# example using Maven
cd product-service && mvn spring-boot:run
cd ../order-service && mvn spring-boot:run
cd ../user-service && mvn spring-boot:run
cd ../discovery-service && mvn spring-boot:run
cd ../api-gateway && mvn spring-boot:run
```

Open <http://localhost:8761> to see registered services.

## Development

Each service uses an in-memory H2 database. Sample data is loaded from `data.sql` on startup. You can hit endpoints directly or via the gateway, e.g.:  
`GET http://localhost:8080/product-service/products`.

### Inter-service communication

Services are independent Spring Boot applications that register with the Eureka discovery server. They don’t automatically invoke one another; interaction happens in two primary ways:

1. **Via the API gateway** – external clients call `http://localhost:8080/<service-name>/...`; the gateway looks up the service in Eureka and forwards the request.
2. **Service-to-service calls** – a service can resolve another by name using a load-balanced `RestTemplate`, `WebClient`, or a Feign client. Eureka and Spring Cloud LoadBalancer handle discovery and load distribution.

On Kubernetes the same patterns apply, though you can also target the cluster DNS names directly (e.g. `http://product-service:8081`).

## Docker / Compose

A `docker-compose.yml` is included for orchestrating all containers; adjust ports as needed before using.

### Kubernetes deployment

Manifest files live in the `k8s` directory. Each service has a `Deployment` and `Service` resource. Apply them with:

```bash
kubectl apply -f k8s/discovery-service-deployment.yaml
kubectl apply -f k8s/product-service-deployment.yaml
kubectl apply -f k8s/order-service-deployment.yaml
kubectl apply -f k8s/user-service-deployment.yaml
kubectl apply -f k8s/api-gateway-deployment.yaml
```

Adjust container image references (registry/tag) before pushing.

### Jenkins CI pipelines

Every module includes a `Jenkinsfile` demonstrating a simple declarative pipeline:
1. checkout source
2. build with Maven
3. build & push a Docker image (uses `REGISTRY` env var)
4. deploy to Kubernetes via `kubectl apply`

Ensure your Jenkins agents have Docker and `kubectl` access to the cluster.
