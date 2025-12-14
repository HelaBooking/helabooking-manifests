# How to Add a New Service

This guide outlines the steps to add a new microservice to the HelaBooking Kustomize manifests.

## Prerequisites
- The Docker image for the service must be available in the Harbor registry.
- You should know the port the service runs on.

## Step 1: Create Component Directory
Create a new directory in `components/` with the name of your service (e.g., `payment-service`).

```bash
mkdir components/payment-service
```

## Step 2: Create Kustomization File
Create a `kustomization.yaml` file inside that directory. This file will inherit from the base backend template.

**File:** `components/payment-service/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base/backends  # Inherit the generic backend template
  # - ../../base/databases # Uncomment if this service needs its own DB

patches:
  # Rename the Deployment and set the Port
  - target:
      group: apps
      version: v1
      kind: Deployment
      name: backend-app
    patch: |-
      - op: replace
        path: /metadata/name
        value: payment-service
      - op: replace
        path: /spec/template/spec/containers/0/ports/0/containerPort
        value: 8085 # <--- YOUR SERVICE PORT HERE

  # Rename the Service
  - target:
      version: v1
      kind: Service
      name: backend-app
    patch: |-
      - op: replace
        path: /metadata/name
        value: payment-service-svc
```

## Step 3: Add to Overlays
You need to register this new component in the environments where it should run (usually `env-dev` first).

Open `overlays/env-dev/kustomization.yaml`:

1.  **Add to resources list:**
    ```yaml
    resources:
    - ...
    - ../../components/payment-service
    ```

2.  **Add initial image tag:**
    ```yaml
    images:
    - ...
    - name: helabooking/payment-service
      newName: harbor.management.ezbooking.lk/helabooking/payment-service
      newTag: dev-latest # Or a specific hash
    ```

## Step 4: Commit and Push
Commit your changes. ArgoCD will pick up the new configuration and deploy the service.
