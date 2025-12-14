# How to Add a New Environment

This guide explains how to create a new environment overlay (e.g., `env-staging` or `env-demo`).

## Step 1: Create Overlay Directory
Create a new directory in `overlays/` matching your environment name.

```bash
mkdir overlays/env-staging
```

## Step 2: Create Kustomization File
Create a `kustomization.yaml` file in the new directory.

**File:** `overlays/env-staging/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

namespace: env-staging

# 1. Select which components to include
resources:
- ../../components/user-service
- ../../components/booking-service
- ../../components/frontend
- ../../base/networking

# 2. Define Image Versions for this environment
images:
- name: helabooking/user-service
  newName: harbor.management.ezbooking.lk/helabooking/user-service
  newTag: staging-v1.0.0
- name: helabooking/booking-service
  newName: harbor.management.ezbooking.lk/helabooking/booking-service
  newTag: staging-v1.0.0

# 3. Environment Specific Patches
patches:
  # Example: Increase replicas for Staging
  - target:
      kind: Deployment
      name: .* # Matches all deployments
    patch: |-
      - op: replace
        path: /spec/replicas
        value: 2
```

## Step 3: Create Environment Patches (Optional)
If you need complex patching (like different environment variables), create a separate patch file (e.g., `patch-env.yaml`) and reference it in the `patches` section of your `kustomization.yaml`.

## Step 4: Configure ArgoCD
Once the files are committed, you must configure a new Application in ArgoCD pointing to this path (`overlays/env-staging`) and the target cluster.
