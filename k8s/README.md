# Atelier Kubernetes — Spring Boot + MySQL

## Ressources Kubernetes (rôle)

- **Deployment** : déploie et maintient des Pods (replicas), gère les mises à jour (rollout/rollback).
- **Service** : point d’accès stable (IP/DNS) vers un ensemble de Pods (load-balancing interne).
- **ConfigMap** : configuration non sensible (variables d’environnement, fichiers de conf).
- **Secret** : données sensibles (mots de passe, tokens), injectées en env/volume.
- **PersistentVolume (PV)** : ressource de stockage « côté cluster ».
- **PersistentVolumeClaim (PVC)** : demande de stockage « côté application » (le Pod monte un PVC).

## Déploiement (manifestes YAML)

Ces manifestes déploient :

- MySQL avec persistance (PV/PVC)
- L’application Spring Boot connectée à MySQL

### Appliquer

```sh
kubectl apply -f k8s/00-namespace.yaml
kubectl apply -n student-management -f k8s
```

### Vérifier l’état

```sh
kubectl -n student-management get all
kubectl -n student-management get pods -o wide
kubectl -n student-management get svc
kubectl -n student-management get cm,secret
kubectl get pv,pvc -A
```

### Logs & diagnostic

```sh
kubectl -n student-management logs deploy/mysql
kubectl -n student-management logs deploy/student-management
kubectl -n student-management describe pod -l app=student-management
kubectl -n student-management get events --sort-by=.metadata.creationTimestamp
```

### Tester l’accès à l’application

1. Trouver le port NodePort :

```sh
kubectl -n student-management get svc student-management
```

1. Depuis votre machine (cluster local type Minikube/Docker Desktop), appeler :

- `http://<IP_NODE>:<NODEPORT>/student/...`

Alternative (simple) : port-forward

```sh
kubectl -n student-management port-forward svc/student-management 8089:8089
```

Puis : `http://localhost:8089/student/...`

### Tester MySQL

```sh
kubectl -n student-management exec -it deploy/mysql -- bash -lc 'mysql -uroot -p"$MYSQL_ROOT_PASSWORD" -e "SHOW DATABASES;"'
```

## Notes PV/PVC

- Le PV fourni utilise `hostPath: /mnt/data/mysql` (adapté à un cluster local / atelier).
- Sur un cluster cloud, utilisez plutôt une **StorageClass** (provisionnement dynamique) et un PVC sans `volumeName`.
