# Infrastructure Docker - Car Buyers Registration

## Structure

```
infra/
├── .env              # Variables d'environnement pour Docker Compose
├── compose.yml       # Configuration Docker Compose
└── README.md         # Ce fichier
```

## Services

- **MySQL** : Base de données (port 3306)
- **Backend** : API Laravel (port 8000)
- **Frontend** : Application Next.js (port 3000)

## Démarrage

### 1. Construire les images locales

```bash
cd infra
docker compose build
```

### 2. Démarrer les services

```bash
docker compose up -d
```

### 3. Vérifier l'état des services

```bash
docker compose ps
```

### 4. Voir les logs

```bash
# Tous les services
docker compose logs -f

# Un service spécifique
docker compose logs -f backend
docker compose logs -f frontend
docker compose logs -f mysql
```

### 4. crée le tunnel

```bash
minikube tunnel

```


### Accéder à ArgoCD

```bash
kubectl port-forward svc/argocd-server -n argocd 8282:443

```


## Arrêt

```bash
# Arrêter les services
docker compose stop

# Arrêter et supprimer les conteneurs
docker compose down

# Arrêter et supprimer les conteneurs + volumes
docker compose down -v
```

## Configuration

Toutes les variables sont dans le fichier `.env` :

- **APP_NAME** : Nom de l'application
- **APP_ENV** : Environnement (production, local, etc.)
- **APP_DEBUG** : Mode debug (true/false)
- **DB_*** : Configuration MySQL
- **CORS_ALLOWED_ORIGINS** : Origines CORS autorisées
- **NEXT_PUBLIC_*** : Variables frontend
- **BACKEND_PORT** : Port du backend (défaut: 8000)
- **FRONTEND_PORT** : Port du frontend (défaut: 3000)
- **MYSQL_PORT** : Port MySQL (défaut: 3306)

## Accès

- Frontend : http://localhost:3000
- Backend API : http://localhost:8000
- MySQL : localhost:3306

## Images locales

Les images sont construites localement :
- `carbuyers-backend:latest`
- `carbuyers-frontend:latest`

## Volumes

Les données persistantes sont stockées dans des volumes Docker :
- `mysql_data` : Données MySQL
- `backend_storage` : Stockage Laravel
- `backend_cache` : Cache Laravel

## Redémarrer avec une base de données propre

```bash
# Force la réexécution des migrations
docker compose exec backend sh -c "FORCE_SETUP=true /entrypoint.sh"
```
