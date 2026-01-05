# 📋 ClientSynchrones - Travaux Pratiques

## 🎯 Objectif du TP

Ce projet vise à démontrer l'architecture d'une **application microservices distribuée** avec les composants suivants :

### Objectifs principaux :
1. **Comprendre les microservices** : Déployer et gérer plusieurs services Spring Boot indépendants
2. **Communication inter-services** : Utiliser OpenFeign pour appeler d'autres microservices
3. **Service Discovery** : Configurer Eureka et Consul pour l'enregistrement dynamique des services
4. **Monitoring & Observabilité** : Monitorer les services avec Prometheus et Grafana
5. **Orchestration** : Utiliser Docker Compose pour orchestrer tous les services

### Services inclus :
- **Eureka Server** : Service de découverte (port 8761)
- **Service Client** : Microservice client qui appelle le service voiture (port 8083)
- **Service Voiture** : Microservice exposant l'API des véhicules (port 8081)
- **Prometheus** : Collecte et stockage des métriques
- **Grafana** : Visualisation des métriques et dashboards

---

## 📊 Architecture et Flux de Données

### Image 1 : Architecture générale du système

![Architecture Microservices](./1.png)

**Explication :**
Cette image montre l'architecture complète du système microservices :
- Les **clients** envoient des requêtes au Service Client (port 8083)
- Le **Service Client** utilise OpenFeign pour communiquer avec le Service Voiture (port 8081)
- Les deux services s'enregistrent auprès d'**Eureka Server** (port 8761) pour la découverte dynamique
- Chaque service expose des **métriques** à l'endpoint `/actuator/prometheus`
- **Prometheus** scrape ces métriques à intervalles réguliers (toutes les 5 secondes)
- Les données sont stockées dans Prometheus et visualisées par **Grafana** (port 3000)

---

### Image 2 : Flux de communication et monitoring

![Flux Communication](./2.png)

**Explication :**
Cette image illustre le flux complet des requêtes et du monitoring :
1. **Requête Client** : Une requête HTTP arrive au Service Client
2. **Appel inter-services** : Le Service Client appelle le Service Voiture via OpenFeign
3. **Réponse** : Le Service Voiture retourne les données
4. **Métriques** : Chaque service génère automatiquement des métriques (temps de réponse, nombre de requêtes, etc.)
5. **Prometheus Scrape** : Prometheus collecte les métriques via `/actuator/prometheus` toutes les 5 secondes
6. **Stockage** : Les données sont stockées dans la base de données temporelle de Prometheus
7. **Grafana Query** : Grafana requête Prometheus pour afficher les métriques en temps réel

---

### Image 3 : Dashboard Grafana et indicateurs clés

![Grafana Dashboard](./3.png)

**Explication :**
Cette image montre un exemple de dashboard Grafana avec les indicateurs clés :
- **État des services** : Affiche le statut "UP" ou "DOWN" de chaque service
- **Utilisation mémoire** : Graphique montrant la mémoire JVM utilisée par chaque microservice
- **Requêtes HTTP** : Taux de requêtes par seconde pour chaque service
- **Threads actifs** : Nombre de threads en cours d'exécution
- **Latence** : Temps de réponse des requêtes
- **Taux d'erreur** : Nombre d'erreurs (status 5xx) par service

---

## 🚀 Démarrage rapide

### 1️⃣ Démarrer tous les services avec Docker Compose
```bash
docker-compose up -d
```

Cela démarre :
- Service Voiture (port 8081)
- Service Client (port 8083)
- Eureka Server (port 8761)
- Prometheus (port 9090)
- Grafana (port 3000)

### 2️⃣ Vérifier l'état des services

**Eureka Dashboard** : http://localhost:8761
- Affiche les services enregistrés

**Prometheus Targets** : http://localhost:9090/targets
- Montre le statut de scrape (UP/DOWN)

**Grafana** : http://localhost:3000
- Login : `admin` / `admin`
- Affiche les dashboards et métriques

### 3️⃣ Tester les APIs

**Appel Service Voiture directement** :
```bash
curl http://localhost:8081/cars
```

**Appel Service Client** (qui appelle Service Voiture) :
```bash
curl http://localhost:8083/clients/cars
```

**Vérifier les métriques** :
```bash
curl http://localhost:8081/actuator/prometheus
curl http://localhost:8083/actuator/prometheus
```

---

## 📈 Requêtes Prometheus utiles

| Requête | Description |
|---------|-------------|
| `up{job="service-client"}` | Vérifie si service-client est accessible |
| `up{job="service-voiture"}` | Vérifie si service-voiture est accessible |
| `jvm_memory_used_bytes{application="service-client"}` | Mémoire utilisée par service-client |
| `rate(http_server_requests_seconds_count[5m])` | Taux de requêtes (par 5 min) |
| `sum by (application) (rate(http_server_requests_seconds_total[5m]))` | Taux combiné de tous les services |

---

## 🔧 Configuration clé

### Prometheus (`prometheus.yml`)
- Scrape interval : **5 secondes**
- Targets : `host.docker.internal:8081` (Service Voiture) et `host.docker.internal:8083` (Service Client)
- Métriques path : `/actuator/prometheus`

### Services (`application.yml`)
- Actuator exposé : `health`, `metrics`, `prometheus`
- Micrometer Prometheus enabled
- Eureka registration enabled

### Grafana
- Datasource Prometheus : `http://prometheus:9090`
- Dashboards provisionnés depuis `/var/lib/grafana/dashboards`

---

## 🛠️ Dépannage

| Problème | Solution |
|---------|----------|
| Services DOWN dans Prometheus | Vérifier que les services sont lancés : `curl http://localhost:8081/actuator/health` |
| Grafana ne se connecte pas à Prometheus | Vérifier que Prometheus est accessible : `curl http://prometheus:9090/-/healthy` |
| Pas de métriques dans Prometheus | Vérifier `/actuator/prometheus` sur chaque service |
| Port déjà utilisé | Modifier `docker-compose.yml` ou tuer les processus existants |

---

## 📚 Technologies utilisées

| Technologie | Rôle |
|-------------|------|
| **Spring Boot 3.2.3** | Framework pour les microservices |
| **Spring Cloud Eureka** | Service Discovery |
| **OpenFeign** | Communication inter-services |
| **Micrometer + Prometheus** | Métriques et monitoring |
| **Prometheus** | Base de données temporelle pour les métriques |
| **Grafana** | Visualisation et dashboards |
| **Docker & Docker Compose** | Containerisation et orchestration |

---

## 📝 Conclusion

Ce TP démontre comment construire une architecture microservices complète avec :
- ✅ Services découplés et registrable via Eureka
- ✅ Communication inter-services via OpenFeign
- ✅ Monitoring complet avec Prometheus et Grafana
- ✅ Déploiement simplifié avec Docker Compose

Cette architecture est **scalable**, **observable** et **maintainable** en production.

---
