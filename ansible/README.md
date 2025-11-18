# Ansible Deployment

This directory contains Ansible playbooks and roles for deploying the Learning Pipelines React application.

## Prerequisites

- Ansible 2.9 or higher
- SSH access to the production server
- SSH key configured for authentication

## Directory Structure

```
ansible/
├── ansible.cfg              # Ansible configuration
├── deploy.yml              # Main deployment playbook
├── inventory/
│   └── production.yml      # Production inventory file
└── roles/
    ├── nginx/              # Nginx web server configuration
    │   ├── handlers/
    │   ├── tasks/
    │   └── templates/
    └── deploy/             # Application deployment tasks
        └── tasks/
```

## Configuration

### Required Environment Variables

Set these in your GitHub repository secrets or export them before running:

- `PRODUCTION_SERVER`: IP address or hostname of your production server (required)
- `PRODUCTION_DOMAIN`: Domain name for your application (optional - uses IP if not set)
- `SSH_PRIVATE_KEY`: SSH private key for authentication (required)
- `SSH_PORT`: Custom SSH port (optional - defaults to 22)

### Inventory Configuration

Edit `inventory/production.yml` to match your infrastructure:

```yaml
ansible_host: your-server-ip
ansible_user: deploy
app_domain: your-domain.com
```

## Usage

### Deploy from GitHub Actions

The deployment runs automatically when pushing to the `main` branch via GitHub Actions.

### Manual Deployment

```bash
# Install Ansible
pip install ansible

# Export environment variables
export PRODUCTION_SERVER=192.168.1.100
export PRODUCTION_DOMAIN=app.example.com  # Optionnel - utilise l'IP si non défini

# Run deployment (port SSH par défaut: 22)
ansible-playbook -i inventory/production.yml deploy.yml \
  --private-key ~/.ssh/your-key \
  --extra-vars "build_artifact=../build-version.tar.gz" \
  --extra-vars "app_version=v1.0.0"

# Run deployment avec port SSH personnalisé (ex: homelab)
ansible-playbook -i inventory/production.yml deploy.yml \
  --private-key ~/.ssh/your-key \
  --extra-vars "build_artifact=../build-version.tar.gz" \
  --extra-vars "app_version=v1.0.0" \
  --extra-vars "ansible_port=8022"
```

### Deploy specific roles

```bash
# Only setup nginx
ansible-playbook -i inventory/production.yml deploy.yml --tags nginx

# Only deploy application
ansible-playbook -i inventory/production.yml deploy.yml --tags deploy
```

## Roles

### nginx

Installs and configures nginx web server:
- Installs nginx package
- Configures virtual host
- Sets up gzip compression
- Configures caching headers
- Creates health check endpoint

### deploy

Handles application deployment:
- Creates release directories
- Extracts build artifacts
- Updates symlinks
- Maintains rollback capability (keeps last 5 releases)
- Sets proper file permissions

## Rollback

To rollback to a previous release:

```bash
ssh deploy@your-server
cd /var/www/learning-pipelines
ln -sfn releases/previous-release-folder current
sudo systemctl reload nginx
```

## Server Setup

Before first deployment, ensure the server has:

1. Ubuntu/Debian OS
2. User `deploy` with sudo privileges
3. SSH key authentication configured
4. Python 3 installed

```bash
# On the server
sudo adduser deploy
sudo usermod -aG sudo deploy
sudo apt update
sudo apt install python3 python3-pip
```

## Troubleshooting

### Check nginx configuration
```bash
sudo nginx -t
```

### View nginx logs
```bash
sudo tail -f /var/log/nginx/learning-pipelines_error.log
```

### Check application files
```bash
ls -la /var/www/learning-pipelines/current/
```

### Verify deployment
```bash
curl http://your-domain.com/health
```
