# HelaBooking Manifests

This repository contains the Kubernetes manifests for the **HelaBooking** application, managed using **Kustomize**. It follows a structured GitOps approach and serves as the **Single Source of Truth** for ArgoCD deployments across on-premise and cloud clusters.

## 📂 Project Structure

The project is organized into three main layers following Kustomize best practices. This hierarchical structure ensures DRY (Don't Repeat Yourself) principles and easy management.

```
.
├── base/               # Generic templates (Blueprints)
├── components/         # Service-specific definitions (Implementations)
└── overlays/           # Environment-specific configurations (Deployments)
```

### 1. Base (The Blueprints)

This directory contains the generic templates for resources. It defines _what_ a resource looks like generally, but not _who_ it is.

- **`base/backends`**: A generic Spring Boot deployment and service definition.
- **`base/databases`**: A generic PostgreSQL StatefulSet definition.
- **`base/networking`**: Common Gateway and VirtualService configurations.

### 2. Components (The Implementations)

This directory defines the actual microservices by inheriting from `base` and applying specific patches.

- It takes a `base` resource (e.g., `backend-app`).
- It renames it (e.g., to `booking-service`).
- It configures specific ports (e.g., `8083`).
- It applies service-specific configurations.
  _Think of this as "Instantiating a Class" in programming._

### 3. Overlays (The Environments)

This directory aggregates components to form complete environments (Dev, QA, Prod).

- **Namespace Isolation**: Defines the namespace (e.g., `env-dev`).
- **Image Pinning**: Specifies the exact container image versions to run.
- **Environment Patching**: Customizes replicas, CPU/Memory limits, and environment variables for that specific environment.

## 🚀 Services

The application consists of the following microservices:

- **User Service**: Manages user accounts and authentication.
- **Event Service**: Handles event creation and management.
- **Booking Service**: Core logic for booking events.
- **Ticketing Service**: Manages ticket generation and validation.
- **Notification Service**: Handles email/SMS notifications.
- **Audit Service**: Logs system activities.
- **Frontend**: The web user interface.

## 🛠 Deployment (GitOps)

**We do not perform manual deployments.**

This repository is monitored by **ArgoCD**. Any change merged into the `main` branch is automatically detected and synced to the respective Kubernetes clusters (On-Prem or Cloud).

- **Development**: Deployed to the on-premise cluster.
- **QA/Prod**: Deployed to the cloud cluster.

### CI/CD Integration

- **Jenkins**: Automatically builds Docker images and pushes them to the Harbor registry.
- **Automation**: Upon a successful build, Jenkins automatically updates the image tags in the `overlays/<env>/kustomization.yaml` files in this repository.
- **ArgoCD**: Detects the commit with the new image tag and syncs the cluster state.

## 📝 Documentation & Guides

Detailed guides for managing this repository can be found in the `_docs/` directory:

- [**How to Add a New Service**](_docs/add-new-service.md)
- [**How to Add a New Environment**](_docs/add-new-environment.md)
