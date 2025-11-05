# 🚀 Kubernetes First App - Structure Simplifiée (Recommandée pour Débutants)

Structure simple et claire pour apprendre Kubernetes et ArgoCD sans complexité inutile.

## 📁 Structure du Projet

```
kubernetes-first-app/
├── README.md
├── .gitignore                       # Protection des secrets
│
├── k8s/                             # Tous les manifests Kubernetes
│   ├── namespace.yaml               # Namespace de l'application
│   ├── backend.yaml                 # Deployment + Service backend
│   ├── frontend.yaml                # Deployment + Service frontend
│   ├── mysql.yaml                   # StatefulSet + Service MySQL
│   └── configmap.yaml               # Variables de configuration
│
├── argocd/                          # Configuration ArgoCD
│   └── application.yaml             # Application ArgoCD
│
└── secrets/                         # Secrets (JAMAIS dans Git !)
    ├── .gitkeep                     # Fichier vide pour garder le dossier
    ├── secret_auth_github.yaml      # ⚠️ LOCAL UNIQUEMENT (pour pull images)
    └── secrets.yaml.example         # Template sans vraies valeurs
```

## 🎯 Principe

**Simple et Direct** : Tous les manifests Kubernetes dans un seul dossier `k8s/`.  
Parfait pour apprendre et débuter avec Kubernetes.


## 🔧 Utilisation

### 1. Déployer manuellement

```bash
# Déployer tous les manifests
kubectl apply -f k8s/

# Ou fichier par fichier
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/mysql.yaml
kubectl apply -f k8s/backend.yaml
kubectl apply -f k8s/frontend.yaml

# Appliquer les secrets (localement)
kubectl apply -f secrets/secret_auth_github.yaml
```

### 2. Vérifier le déploiement

```bash
# Voir tous les pods
kubectl get pods -n myapp

# Voir les services
kubectl get svc -n myapp

# Voir les logs
kubectl logs -f deployment/backend -n myapp
kubectl logs -f deployment/frontend -n myapp
```

### 3. Avec ArgoCD (GitOps - Recommandé)

#### a) Installer ArgoCD

```bash
# Créer le namespace ArgoCD
kubectl create namespace argocd

# Installer ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Exposer l'UI (en local)
kubectl port-forward svc/argocd-server -n argocd 8080:443

# Récupérer le mot de passe admin
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
```

#### b) Déployer votre application

```bash
# Appliquer la configuration ArgoCD
kubectl apply -f argocd/application.yaml

# ArgoCD va automatiquement :
# ✅ Cloner votre repo Git
# ✅ Déployer les manifests du dossier k8s/
# ✅ Synchroniser automatiquement les changements
# ✅ Réparer automatiquement si quelqu'un modifie manuellement (selfHeal)
```

#### c) Voir votre application dans ArgoCD UI

1. Ouvrir http://localhost:8080
2. Login : `admin` / mot de passe récupéré
3. Voir votre application `myapp-argo-application`

## 📝 Fichier ArgoCD Important

### argocd/application.yaml

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp-argo-application
  namespace: argocd
spec:
  project: default
  
  source:
    repoURL: https://github.com/votre-username/votre-repo.git
    targetRevision: HEAD
    path: k8s                      # ← Dossier contenant les manifests
  
  destination:
    server: https://kubernetes.default.svc
    namespace: myapp
  
  syncPolicy:
    syncOptions:
    - CreateNamespace=true         # Crée le namespace automatiquement
    
    automated:
      selfHeal: true               # Répare automatiquement les drifts
      prune: true                  # Supprime les ressources obsolètes
```

**⚠️ N'oubliez pas** de remplacer l'URL du repo par le vôtre !

### ⚠️ RÈGLE D'OR : JAMAIS de secrets en clair dans Git !


#### Comment gérer les secrets ?

**En local (développement) :**
```bash
# Appliquer manuellement
kubectl apply -f secrets/secret_auth_github.yaml
```

**En production (solutions professionnelles) :**
- **Sealed Secrets** : Chiffre les secrets pour Git
- **External Secrets Operator** : Synchronise avec Vault/AWS/Azure
- **HashiCorp Vault** : Coffre-fort de secrets centralisé

#### Créer un template de secrets

```bash
# secrets/secrets.yaml.example
apiVersion: v1
kind: Secret
metadata:
  name: ghcr-secret-auth
  namespace: default
type: kubernetes.io/dockerconfigjson
data:
  .dockerconfigjson: <VOTRE_SECRET_BASE64_ICI>
```

## 🚦 Workflow GitOps Complet

```
1. Code → Git Push
         ↓
2. ArgoCD détecte le changement
         ↓
3. ArgoCD applique automatiquement
         ↓
4. Application déployée ✅
```

## 🎓 Quand utiliser cette structure ?

✅ Apprentissage de Kubernetes  
✅ Petites applications ou MVP  
✅ Projets personnels  
✅ Équipes qui débutent avec K8s  
✅ Un seul environnement (dev ou prod)

## 📈 Évolution future

Quand votre application grandit :
- Passez à la **structure complète avec Kustomize** (plusieurs environnements)
- Utilisez **Helm** pour des applications plus complexes
- Implémentez **Sealed Secrets** ou **External Secrets**

## 🔗 Ressources

- [Kubernetes Documentation](https://kubernetes.io/docs/)
- [ArgoCD Getting Started](https://argo-cd.readthedocs.io/en/stable/getting_started/)
- [GitOps Principles](https://www.gitops.tech/)

## 💡 Commandes Utiles

```bash
# Voir tout dans le namespace
kubectl get all -n myapp

# Supprimer tout
kubectl delete -f k8s/

# Redémarrer un déploiement
kubectl rollout restart deployment/backend -n myapp

# Accéder à un pod
kubectl exec -it <pod-name> -n myapp -- /bin/bash

# Port-forward pour tester localement
kubectl port-forward svc/frontend 3000:80 -n myapp
```

