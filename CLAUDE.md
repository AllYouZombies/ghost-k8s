# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Kubernetes Helm chart for deploying Ghost CMS (professional publishing platform) with MariaDB database. The chart packages Ghost v6.10.3 with all supporting infrastructure for Kubernetes clusters.

## Common Commands

```bash
# Build Helm dependencies (required before install)
cd helm/charts/v1/ghost
helm dependency build .

# Install/upgrade the chart
helm upgrade --install ghost . -n ghost

# Install with custom values
helm upgrade --install ghost . -n ghost -f values-prod.yaml

# Create required MariaDB secret before deployment
kubectl -n ghost create secret generic ghost-mariadb-secret \
  --from-literal=mariadb-root-password=$(openssl rand -hex 32) \
  --from-literal=mariadb-password=$(openssl rand -hex 32)

# Validate templates without installing
helm template ghost . --debug

# Lint the chart
helm lint .
```

## Architecture

The Helm chart deploys:

1. **Ghost CMS Deployment** - Single replica pod running Ghost on port 2368 with security hardening (non-root UID 1000, dropped capabilities, read-only root filesystem)

2. **MariaDB** - Subchart dependency from `oci://registry-1.docker.io/cloudpirates/mariadb` providing the database backend

3. **PersistentVolumeClaim** - Storage for Ghost content directory (`/var/lib/ghost/content`)

4. **Ingress** - HTTP/HTTPS routing with TLS support and cert-manager integration

5. **ConfigMap** - Environment variables for Ghost configuration (database, mail, URLs)

6. **Service** - ClusterIP service for internal pod discovery

## Key Files

- `helm/charts/v1/ghost/Chart.yaml` - Chart metadata and MariaDB dependency declaration
- `helm/charts/v1/ghost/values.yaml` - All configurable parameters with defaults
- `helm/charts/v1/ghost/templates/_helpers.tpl` - Template functions including `app.ghostUrl` and `app.mariadbHost`
- `helm/charts/v1/ghost/templates/deployment.yaml` - Pod spec with health probes checking `/ghost/api/admin/site/`

## Configuration Highlights

Key values to customize in deployments:
- `ingress.host` - Blog hostname (default: `blog.example.com`)
- `ingress.className` - Ingress controller class
- `ingressTLS.enabled` - Enable HTTPS
- `mariadb.auth.existingSecret` - Reference to database credentials secret
- `persistence.size` - Content storage size (default: 10Gi)
- `resources.limits` - Container resource constraints (default: 512Mi memory, 500m CPU)

## Kubernetes Requirements

- Kubernetes 1.19+ (uses Ingress API v1)
- Pre-existing storage class for PVC
- MariaDB secret must be created before chart installation
