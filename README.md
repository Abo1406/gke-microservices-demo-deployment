# GKE Microservices Demo Deployment  
[![License](https://img.shields.io/badge/license-Apache--2.0-blue.svg)](LICENSE)  
> Kubernetes deployment of Google's microservices demo on Google Kubernetes Engine (GKE).

This project deploys a cloud-native e-commerce application using Kubernetes on GKE. The deployment is based on Google's [microservices-demo](https://github.com/GoogleCloudPlatform/microservices-demo) with custom configurations optimized for GKE.

## 📦 Included Services
```mermaid
flowchart TD
  %% Legend
  subgraph Legend
    direction TB
    HTTP[<b>HTTP</b>] -->|Requests| dest1[( )]
    gRPC[<b>gRPC</b>] -->|RPC| dest2[( )]
    TCP[<b>TCP</b>] -->|Cache| dest3[( )]
  end

  %% Load Generator
  subgraph "Load Generator"
    direction LR
    LG[Load Generator]
    LG -- HTTP --> APIGW
  end

  %% API Gateway
  subgraph "API Gateway"
    direction LR
    APIGW[API Gateway]
  end

  %% Frontend
  subgraph "Edge / Client"
    direction LR
    FE[Web & Mobile Frontend]
    FE -- HTTP --> APIGW
  end

  %% Backend Services
  subgraph "Backend Services"
    direction LR
    PC[Product Catalog]
    REC[Recommendation]
    CART[Cart]
    SHIP[Shipping]
    CURR[Currency]
    PAY[Payment]
    EMAIL[Email Notification]
  end

  %% Data Layer
  subgraph "Data Layer"
    direction LR
    RC[(Redis Cache)]
  end

  %% Service Connections
  APIGW -- gRPC --> PC
  APIGW -- gRPC --> REC
  APIGW -- gRPC --> CART
  APIGW -- gRPC --> SHIP
  APIGW -- gRPC --> CURR
  APIGW -- gRPC --> PAY
  APIGW -- gRPC --> EMAIL

  CART  -- TCP --> RC
  REC   -- TCP --> RC
  PC    -- TCP --> RC
  SHIP  -- TCP --> RC
  CURR  -- TCP --> RC
  PAY   -- TCP --> RC
  EMAIL -- TCP --> RC

  %% Styling
  classDef loadgen fill:#FFA726,stroke:#FB8C00,stroke-width:2px;
  classDef gateway  fill:#FFCC80,stroke:#EF6C00,stroke-width:2px;
  classDef edge     fill:#A5D6A7,stroke:#388E3C,stroke-width:2px;
  classDef backend  fill:#90CAF9,stroke:#1E88E5,stroke-width:2px;
  classDef database fill:#FFAB91,stroke:#D84315,stroke-width:2px;

  class LG loadgen;
  class APIGW gateway;
  class FE edge;
  class PC,REC,CART,SHIP,CURR,PAY,EMAIL backend;
  class RC database;

```
| Service                  | Port  | Replicas | Function                         |
|--------------------------|-------|----------|----------------------------------|
| `frontend`               | 8080  | 2        | Web interface (LoadBalancer)     |
| `checkoutservice`        | 5050  | 2        | Order processing                 |
| `cartservice`            | 7070  | 2        | Shopping cart management         |
| `productcatalogservice`  | 3550  | 2        | Product database                 |
| `currencyservice`        | 7000  | 2        | Currency conversion              |
| `paymentservice`         | 50051 | 2        | Payment processing               |
| `shippingservice`        | 50051 | 2        | Shipping logic                   |
| `emailservice`           | 8080  | 1        | Email notifications              |
| `recommendationservice`  | 8080  | 2        | Product recommendations          |
| `adservice`              | 9555  | 2        | Advertising promotions           |
| `redis-cart`             | 6379  | 2        | Redis cache for carts            |

## 🚀 Deployment Steps
### Prerequisites
- Google Cloud Project with billing enabled
- [gcloud CLI](https://cloud.google.com/sdk/docs/install) installed
- Kubernetes cluster running on GKE

### 1. Apply Kubernetes Configuration
```bash
kubectl apply -f microservices-demo.yaml
```
### 2. Verify Deployment Status
```bash
kubectl get deployments
kubectl get services  # Note the EXTERNAL-IP of `frontend`
```
### 3. Access the Application
```bash
kubectl get service frontend
```
Open http://<EXTERNAL_IP> in your browser.

## 🔧 Configuration Highlights
* Resource Constraints: CPU/memory limits defined for all services (e.g., emailservice capped at 200m CPU/128Mi memory).
* Health Checks: Liveness/readiness probes using gRPC or HTTP.
* Redis Persistence: emptyDir volume for cart data.
* Environment Variables: Service discovery via Kubernetes DNS (e.g., CART_SERVICE_ADDR: "cartservice:7070").
## 🧹 Cleanup
```bash
kubectl delete -f microservices-demo.yaml
```
## 📜 License
* This deployment configuration is licensed under the Apache License 2.0.
* The microservices application is owned by Google. See original [repo](https://github.com/GoogleCloudPlatform/microservices-demo) for details.
  
