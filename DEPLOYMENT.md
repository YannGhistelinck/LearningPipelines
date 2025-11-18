# Guide de Déploiement en Production

Ce document décrit le pipeline de déploiement automatique pour l'application Learning Pipelines.

## Vue d'ensemble

Le pipeline de déploiement utilise **GitHub Actions** pour l'orchestration CI/CD et **Ansible** pour le déploiement sur les serveurs de production.

### Architecture du Pipeline

```
GitHub Push (main) → Tests → Build → Deploy (Ansible) → Verify
```

1. **Tests**: Exécution des tests unitaires et génération du rapport de couverture
2. **Build**: Construction de l'application React pour la production
3. **Deploy**: Déploiement via Ansible sur le serveur de production
4. **Verify**: Vérification de l'état de santé de l'application

## Configuration Initiale

### 1. Secrets GitHub

Configurez les secrets suivants dans votre repository GitHub (`Settings → Secrets and variables → Actions`):

| Secret | Description | Requis | Exemple |
|--------|-------------|--------|---------|
| `PRODUCTION_SERVER` | Adresse IP ou hostname du serveur | ✅ Oui | `192.168.1.100` ou `server.example.com` |
| `PRODUCTION_DOMAIN` | Nom de domaine de l'application | ❌ Non* | `app.example.com` |
| `SSH_PRIVATE_KEY` | Clé SSH privée pour l'authentification | ✅ Oui | Contenu de `~/.ssh/id_rsa` |
| `SSH_PORT` | Port SSH personnalisé | ❌ Non** | `8022` |

**Notes:**
- \* Si `PRODUCTION_DOMAIN` n'est pas défini, le système utilisera `PRODUCTION_SERVER` (adresse IP)
- \** Par défaut, le port 22 est utilisé. Définissez `SSH_PORT` si vous utilisez un port différent (ex: 8022 pour un homelab)

### 2. Préparation du Serveur

Sur votre serveur de production, exécutez:

```bash
# Créer l'utilisateur de déploiement
sudo adduser deploy
sudo usermod -aG sudo deploy

# Installer les dépendances
sudo apt update
sudo apt install -y python3 python3-pip nginx

# Configurer l'authentification SSH
sudo su - deploy
mkdir -p ~/.ssh
chmod 700 ~/.ssh
# Ajoutez votre clé publique dans ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
```

### 3. Configuration DNS (optionnel)

**Pour un serveur avec nom de domaine:**

Configurez un enregistrement DNS de type A pointant vers votre serveur:

```
app.example.com → IP_DU_SERVEUR
```

**Pour un homelab sans nom de domaine:**

Vous pouvez ignorer cette étape. L'application sera accessible directement via l'adresse IP:

```
http://192.168.1.100
```

### 4. Exemple de Configuration Homelab

Pour un homelab typique (IP locale, port SSH personnalisé):

**Secrets GitHub à configurer:**
```
PRODUCTION_SERVER = 192.168.1.100
SSH_PORT = 8022
SSH_PRIVATE_KEY = [votre clé privée]
# PRODUCTION_DOMAIN non défini - l'IP sera utilisée
```

**Connexion SSH manuelle pour tester:**
```bash
ssh -p 8022 deploy@192.168.1.100
```

**Accès à l'application après déploiement:**
```
http://192.168.1.100
```

## Utilisation du Pipeline

### Déploiement Automatique

Le déploiement se déclenche automatiquement lors d'un push sur la branche `main`:

```bash
git add .
git commit -m "feat: nouvelle fonctionnalité"
git push origin main
```

### Déploiement Manuel

Depuis l'interface GitHub:
1. Allez dans `Actions`
2. Sélectionnez le workflow "Production Deployment"
3. Cliquez sur "Run workflow"
4. Sélectionnez la branche `main`
5. Cliquez sur "Run workflow"

## Structure des Fichiers

```
.github/
└── workflows/
    └── production-deploy.yml    # Pipeline GitHub Actions

ansible/
├── ansible.cfg                  # Configuration Ansible
├── deploy.yml                   # Playbook principal
├── inventory/
│   └── production.yml          # Inventaire des serveurs
└── roles/
    ├── nginx/                  # Configuration nginx
    │   ├── handlers/
    │   │   └── main.yml
    │   ├── tasks/
    │   │   └── main.yml
    │   └── templates/
    │       └── nginx-site.conf.j2
    └── deploy/                 # Déploiement de l'application
        └── tasks/
            └── main.yml
```

## Fonctionnalités du Pipeline

### Tests Automatiques
- Exécution de tous les tests unitaires
- Génération du rapport de couverture
- Upload vers Codecov (optionnel)

### Build Optimisé
- Construction en mode production
- Minification et optimisation
- Création d'un artefact tar.gz

### Déploiement Ansible
- Installation et configuration de nginx
- Déploiement de l'application
- Gestion des releases (garde les 5 dernières)
- Rollback facile en cas de problème

### Vérification Post-déploiement
- Health check HTTP
- Vérification du statut 200
- Notifications de succès/échec

## Configuration Nginx

Le template nginx configure automatiquement:

- **Compression gzip** pour les assets statiques
- **Headers de sécurité** (X-Frame-Options, CSP, etc.)
- **Cache** pour les fichiers statiques (1 an)
- **Endpoint de santé** à `/health`
- **Support SPA** avec fallback vers `index.html`

## Gestion des Releases

Les releases sont stockées dans `/var/www/learning-pipelines/releases/` avec le format:

```
/var/www/learning-pipelines/
├── current → releases/abc123           # Symlink vers la version active
├── releases/
│   ├── abc123/                        # Release actuelle
│   ├── def456/                        # Release précédente
│   └── ...
└── shared/                            # Fichiers partagés entre releases
```

### Rollback Manuel

En cas de problème, vous pouvez revenir à une version précédente:

```bash
ssh deploy@server.example.com
cd /var/www/learning-pipelines
ln -sfn releases/VERSION_PRECEDENTE current
sudo systemctl reload nginx
```

## Environnements GitHub

Le pipeline utilise l'environnement GitHub `production` qui permet:
- Protection de branche
- Approbation manuelle avant déploiement (optionnel)
- Tracking des déploiements

Pour activer l'approbation manuelle:
1. Allez dans `Settings → Environments → production`
2. Activez "Required reviewers"
3. Ajoutez des reviewers

## Monitoring et Logs

### Logs Nginx
```bash
# Logs d'accès
sudo tail -f /var/log/nginx/learning-pipelines_access.log

# Logs d'erreur
sudo tail -f /var/log/nginx/learning-pipelines_error.log
```

### État du Service
```bash
# Statut nginx
sudo systemctl status nginx

# Recharger la configuration
sudo systemctl reload nginx

# Redémarrer nginx
sudo systemctl restart nginx
```

### Vérification de Santé
```bash
# Test local
curl http://localhost/health

# Test distant
curl http://app.example.com/health
```

## Sécurité

### Bonnes Pratiques Implémentées

- ✅ Clés SSH pour l'authentification (pas de mot de passe)
- ✅ Secrets GitHub pour les données sensibles
- ✅ Headers de sécurité HTTP
- ✅ Utilisateur dédié pour le déploiement
- ✅ Permissions appropriées sur les fichiers

### Pour activer HTTPS

1. Installez Certbot:
```bash
sudo apt install certbot python3-certbot-nginx
```

2. Obtenez un certificat SSL:
```bash
sudo certbot --nginx -d app.example.com
```

3. Décommentez la section SSL dans `ansible/roles/nginx/templates/nginx-site.conf.j2`

## Dépannage

### Le déploiement échoue à l'étape SSH
- Vérifiez que `SSH_PRIVATE_KEY` est correctement configuré
- Testez la connexion SSH manuellement: `ssh deploy@SERVER`

### Erreur "nginx configuration test failed"
- Connectez-vous au serveur et exécutez: `sudo nginx -t`
- Vérifiez les logs: `sudo journalctl -u nginx`

### L'application ne se charge pas
- Vérifiez que les fichiers sont bien déployés: `ls -la /var/www/learning-pipelines/current/`
- Vérifiez les permissions: `sudo chown -R www-data:www-data /var/www/learning-pipelines/current/`

### Health check échoue
- Vérifiez que nginx écoute sur le bon port: `sudo netstat -tlnp | grep nginx`
- Testez localement sur le serveur: `curl localhost/health`

## Améliorations Futures

- [ ] Ajout du support SSL/TLS automatique
- [ ] Mise en place de notifications Slack/Discord
- [ ] Ajout de tests d'intégration
- [ ] Configuration de monitoring (Prometheus/Grafana)
- [ ] Blue-green deployment
- [ ] Canary releases

## Support

Pour toute question ou problème:
1. Consultez les logs GitHub Actions
2. Vérifiez les logs du serveur
3. Consultez la documentation Ansible: ansible/README.md
