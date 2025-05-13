# FClub Fitness Management System

![FClub Logo](https://ucarecdn.com/b3501875-1c4e-4e68-8641-493ccbdba71f/ChatGPTImage7202514_37_51fotorbgremover20250508235426.png)

FClub is a comprehensive microservice-based solution designed for modern fitness club management. The system provides end-to-end automation of club operations including member management, access control, and notifications through a distributed architecture that ensures scalability and reliability.

## System Architecture

The solution follows a microservices pattern with 3 core backend services and 2 client applications, orchestrated through Kubernetes. All external requests are routed via Nginx ingress controller, while internal service-to-service communication utilizes HTTP REST APIs.

![FClub Architecture Diagram](https://ucarecdn.com/15b9e9d9-585d-4cc1-8ee9-ea1243db9009/20250509154500.png)

Each microservice adheres to **Clean Architecture** principles, enforcing a strict separation of concerns through modular layers:
- **Domain Layer**: Contains core business logic, entities, and interfaces.  
- **Application Layer**: Implements use cases.  
- **Infrastructure Layer**: Handles external concerns (database access, messaging, APIs) via adapters.  
- **WebUI Layer (Presentation)**: Delivers HTTP endpoints (controllers, middleware) and serves as the entry point for external requests.  
- **Shared Layer**: Hosts cross-cutting utilities and common contracts consumed by other layers.  

Dependencies flow inward: outer layers (WebUI, Infrastructure) depend on inner layers (Application, Domain), while the **Domain layer remains isolated** from all external concerns. This ensures testability, flexibility, and maintainability by decoupling business rules from implementation details. 

![FClub Architecture Diagram Microservice](https://ucarecdn.com/e228e111-e652-4523-ade6-77d3a7ba0ba6/20250513174442.png)

## Core Components

| Component                | Description                                                                                     | Technology Stack                          |
|--------------------------|-------------------------------------------------------------------------------------------------|-------------------------------------------|
| Management Service       | Central business logic service handling main domain entities  | ASP.NET Core, C#, PostgreSQL|
| Access Control Service   | Access validation system integrating with turnstile hardware      | ASP.NET Core, C#, PostgreSQL             |
| Notification Service     | Manages email notifications for clients   | ASP.NET Core, C#, PostgreSQL     |
| Admin Dashboard          | Responsive management interface for staff with role-based access control                 | React, TypeScript       |
| Turnstile Interface      | Simulation of the club's access control system          | React, TypeScript           |

## Security Architecture

The system implements a robust **JWT-based authentication and authorization** mechanism with **role-based access control (RBAC)**. Each microservice validates incoming requests by verifying the `Bearer` JWT token in the `Authorization` header.

### Key Security Features

#### Strict Token Validation
- **All requests** (client-to-server and server-to-server) require a valid JWT.
- Tokens are signed with **different keys** for:
  - External requests (client-facing)
  - Internal service communication (server-to-server)
- Validation includes:
  - Issuer (`iss`) verification
  - Expiration time (`exp`) checks

#### Two-Tier Token System
| Token Type       | Lifetime | Purpose                          |
|------------------|----------|----------------------------------|
| **Access Token** | 60 days  | Short-lived API authorization    |
| **Refresh Token**| 360 days | Secure renewal of access tokens  |

#### Role-Based Access Control (RBAC)
- JWTs include **role claims** for granular permissions.
- Microservices enforce access based on:
  - User roles (e.g., `Admin`, `Manager`)

## Source Code Repositories

| Repository | Purpose |
|------------|---------|
| [FClub.Backend.SyncMessaging](https://github.com/denekben/FClub.Backend.SyncMessaging) | Micsrocesvices backend application |
| [FClub.Backend.Common](https://github.com/denekben/FClub.Backend.Common) | Common microservice class library |
| [FClub.Client.Management.SyncMessaging](https://github.com/denekben/FClub.Client.Management.SyncMessaging) | Admin dashboard frontend |
| [Club.Client.Turnstile](https://github.com/denekben/FClub.Client.Turnstile) | Turnstile access control frontend |
| [FClub.K8S.SyncMessaging](https://github.com/denekben/FClub.K8S.SyncMessaging) | Kubernetes deployment configurations |

## Deployment Commands

1. First, create PostgreSQL secrets:
```bash
kubectl create secret generic postgres-secrets --from-literal=POSTGRES_PASSWORD={YOUR_PASSWORD}
```
2. Create Persistent Volume Claims:
```bash
kubectl apply -f Server/Notifications/notifications-local-pvc.yaml
kubectl apply -f Server/Management/management-local-pvc.yaml
kubectl apply -f Server/AccessControl/accesscontrol-local-pvc.yaml
```

3. Deploy PostgreSQL databases:
```bash
kubectl apply -f Server/Notifications/notifications-postgres-depl.yaml
kubectl apply -f Server/Management/management-postgres-depl.yaml
kubectl apply -f Server/AccessControl/accesscontrol-postgres-depl.yaml
```

4. Deploy microservices:
```bash
kubectl apply -f Server/Notifications/notifications-depl.yaml
kubectl apply -f Server/Management/management-depl.yaml
kubectl apply -f Server/AccessControl/accesscontrol-depl.yaml
```

5. Deploy client applications:
```bash
kubectl apply -f Client/client-management-depl.yaml
kubectl apply -f Client/client-turnstile-depl.yaml
```

6. Set up ingress:
```bash
kubectl apply -f Nginx/ingress-srv.yaml
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

## How does it look like
Admin dashboard frontend
![Management-Client](https://ucarecdn.com/e22c606a-9064-4845-9469-331a451a6164/20250509191911.png)
Turnstile access control frontend
![Turnstile-Client](https://ucarecdn.com/d9e270a9-5574-440b-941d-7514da23cb52/20250509191945.png)
