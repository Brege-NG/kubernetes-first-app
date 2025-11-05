# 🚀 Kubernetes First App - Structure Complète (Production-Ready)

Cette structure utilise **Kustomize** pour gérer plusieurs environnements (dev, staging, prod).

## 📁 Structure du Projet

```
kubernetes-first-app/
├── README.md
├── .gitignore
│
├── argocd/                          # Configurations ArgoCD
│   ├── application-dev.yaml
│   ├── application-staging.yaml
│   └── application-prod.yaml
│
├── k8s/                             # Manifests Kubernetes
│   ├── base/                        # Configuration de base (commune)
│   │   ├── kustomization.yaml       # Liste des ressources de base
│   │   ├── namespace.yaml
│   │   ├── backend.yaml
│   │   ├── frontend.yaml
│   │   ├── mysql.yaml
│   │   └── configmap.yaml
│   │
│   └── overlays/                    # Configurations par environnement
│       ├── dev/
│       │   └── kustomization.yaml   # Surcharges pour dev (1 replica)
│       ├── staging/
│       │   └── kustomization.yaml   # Surcharges pour staging (2 replicas)
│       └── prod/
│           └── kustomization.yaml   # Surcharges pour prod (3 replicas, resources++)
│
├── secrets/                         # Secrets locaux (JAMAIS dans Git)
│   ├── .gitkeep
│   ├── secret_auth_github.yaml      # ⚠️ LOCAL UNIQUEMENT
│   └── secrets.yaml.example         # Template pour documentation
│
└── scripts/                         # Scripts utilitaires
    ├── deploy.sh                    # Script de déploiement
    └── setup-secrets.sh             # Script de création des secrets
```

## 🎯 Principe de Kustomize

**Base** = Configuration commune à tous les environnements  
**Overlays** = Personnalisations spécifiques par environnement

### Avantages
✅ Pas de duplication de code  
✅ Gestion facile de multiples environnements  
✅ Intégré nativement dans kubectl  
✅ Utilisé par ArgoCD

## 🔧 Utilisation

### 1. Visualiser les manifests générés

```bash
# Dev
kubectl kustomize k8s/overlays/dev

# Staging
kubectl kustomize k8s/overlays/staging

# Production
kubectl kustomize k8s/overlays/prod
```

### 2. Déployer directement

```bash
# Dev
kubectl apply -k k8s/overlays/dev

# Staging
kubectl apply -k k8s/overlays/staging

# Production
kubectl apply -k k8s/overlays/prod
```

### 3. Avec ArgoCD (Recommandé)

```bash
# Déployer l'application ArgoCD pour dev
kubectl apply -f argocd/application-dev.yaml

# ArgoCD va automatiquement :
# - Surveiller votre repo Git
# - Appliquer les manifests du dossier k8s/overlays/dev
# - Synchroniser automatiquement les changements
```

## 📝 Exemple de kustomization.yaml

### Base (k8s/base/kustomization.yaml)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - namespace.yaml
  - backend.yaml
  - frontend.yaml
  - mysql.yaml
  - configmap.yaml
```

### Overlay Dev (k8s/overlays/dev/kustomization.yaml)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base

namespace: myapp-dev

replicas:
  - name: backend
    count: 1
  - name: frontend
    count: 1

commonLabels:
  environment: dev
```

### Overlay Prod (k8s/overlays/prod/kustomization.yaml)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

bases:
  - ../../base

namespace: myapp-prod

replicas:
  - name: backend
    count: 3
  - name: frontend
    count: 3

commonLabels:
  environment: prod

# Augmenter les ressources en prod
patchesStrategicMerge:
  - |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: backend
    spec:
      template:
        spec:
          containers:
          - name: backend
            resources:
              requests:
                memory: "256Mi"
                cpu: "500m"
              limits:
                memory: "512Mi"
                cpu: "1000m"
```


## 🎓 Quand utiliser cette structure ?

✅ Applications avec plusieurs environnements  
✅ Équipes DevOps expérimentées  
✅ Déploiements en production  
✅ Besoin de DRY (Don't Repeat Yourself)

## 🔗 Ressources

- [Documentation Kustomize](https://kustomize.io/)
- [ArgoCD + Kustomize](https://argo-cd.readthedocs.io/en/stable/user-guide/kustomize/)
