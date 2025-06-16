# Kustomize Template

This repository provides a reusable, modular structure for deploying to Kubernetes using **Kustomize**. It's designed as a **remote base** that can be referenced from other Kustomize projects to promote DRY, scalable configurations.

## 📁 Structure

```
.
├── backend/
│ ├── backend-deployment.yaml
│ ├── backend-service.yaml
│ └── kustomization.yaml
├── frontend/
│ ├── frontend-deployment.yaml
│ ├── frontend-service.yaml
│ └── kustomization.yaml
```

Each directory is a standalone Kustomize base and can be included as a remote resource.


## 🔗 How to Use (as a Remote Base)

You can reference the bases from this repository in your own Kustomize setup like so:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - namespace.yaml
  - github.com/AdhamBasheir/k8s-kustomize-template//backend?ref=main
  - github.com/AdhamBasheir/k8s-kustomize-template//frontend?ref=main

namespace: my-namespace     # Project namespace

namePrefix: my-namespace-   # Prefix added to resource names

labels:
  - pairs:
      app: my-app           # Application label
    includeSelectors: true
    includeTemplates: true

patches:
  # Frontend Deployment Patch
  - patch: |-
      - op: add
        path: /spec/replicas
        value: 2                      # Number of frontend replicas
      - op: add
        path: /spec/template/spec/containers/0
        value:
          name: frontend-app          # Frontend container name
          image: myrepo/frontend:1.0  # Frontend Docker image
          ports:
            - containerPort: 80       # Frontend container port
    target:
      kind: Deployment
      labelSelector: "tier=frontend"

  # Frontend Service Patch
  - patch: |-
      - op: add
        path: /spec/ports/0
        value:
          protocol: TCP
          port: 80                    # Frontend service port
          targetPort: 80              # Frontend target port
      - op: add
        path: /spec/type
        value: ClusterIP              # Frontend service type
    target:
      kind: Service
      labelSelector: "tier=frontend"

  # Backend Deployment Patch
  - patch: |-
      - op: add
        path: /spec/replicas
        value: 2                      # Number of backend replicas
      - op: add
        path: /spec/template/spec/containers/0
        value:
          name: backend-api           # Backend container name
          image: myrepo/backend:1.0   # Backend Docker image
          ports:
            - containerPort: 3000     # Backend container port
    target:
      kind: Deployment
      labelSelector: "tier=backend"

  # Backend Service Patch
  - patch: |-
      - op: add
        path: /spec/ports/0
        value:
          protocol: TCP
          port: 3000                  # Backend service port
          targetPort: 3000            # Backend target port
      - op: add
        path: /spec/type
        value: ClusterIP              # Backend service type
    target:
      kind: Service
      labelSelector: "tier=backend"
```

### 🧾 Required Files

You must provide your own `namespace.yaml` like this:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: my-namespace    # your project namespace
```

## ✅ Features

- Modular and reusable via Kustomize remote bases
- Separate directories for each tier
- Easy to plug into existing Kubernetes projects
- Suitable for demos, CI/CD pipelines, or production bootstraps


## 🛠️ Prerequisites

- Kubernetes cluster (e.g., Minikube, Kind, EKS)
- `kubectl` installed

## 🙌 Contributing

Feel free to fork and modify to fit your specific technologies, add overlays, gateway api, config maps, secrets, or other enhancements.
