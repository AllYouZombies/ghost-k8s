# Helm chart for Ghost CMS

## Prerequisites

### Build dependencies

```bash
cd helm/charts/v1/ghost
helm dependency build .
```

### Prepare secrets

#### MariaDB secret (required)

```bash
MARIADB_ROOT_PASSWORD=$(openssl rand -hex 32)
MARIADB_PASSWORD=$(openssl rand -hex 32)

kubectl create secret generic ghost-mariadb-secret \
  --from-literal=mariadb-root-password=$MARIADB_ROOT_PASSWORD \
  --from-literal=mariadb-password=$MARIADB_PASSWORD
```

#### Mail secret (optional)

Required if you want Ghost to send transactional emails (member signups, password resets, staff invitations, Stripe receipts).

```bash
kubectl create secret generic ghost-mail-secret \
  --from-literal=mail-user='your-smtp-username' \
  --from-literal=mail-password='your-smtp-password'
```

Then enable mail in your values:

```yaml
mail:
  enabled: true
  transport: "SMTP"
  from: "'My Blog' <noreply@example.com>"
  options:
    host: smtp.mailgun.org  # or smtp.sendgrid.net, email-smtp.us-east-1.amazonaws.com, etc.
    port: 587
    secure: false  # true for port 465
  auth:
    existingSecret: "ghost-mail-secret"
    secretKeys:
      userKey: "mail-user"
      passwordKey: "mail-password"
```

## Installation

```bash
helm install ghost . -f values-prod.yaml
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `image.repository` | Ghost image | `ghost` |
| `image.tag` | Ghost version | `6.10.3-alpine3.23` |
| `ingress.enabled` | Enable ingress | `true` |
| `ingress.host` | Blog hostname | `blog.example.com` |
| `ingress.className` | Ingress class | `""` |
| `ingressTLS.enabled` | Enable TLS | `false` |
| `ingressTLS.secretName` | TLS secret name | `""` |
| `ghost.url` | Site URL (auto from ingress.host if empty) | `""` |
| `ghost.adminUrl` | Separate admin URL | `""` |
| `mail.enabled` | Enable SMTP | `false` |
| `mail.from` | From address | `"'Ghost' <noreply@example.com>"` |
| `mail.options.host` | SMTP host | `""` |
| `mail.options.port` | SMTP port | `587` |
| `persistence.enabled` | Enable PVC | `true` |
| `persistence.size` | PVC size | `10Gi` |
| `persistence.storageClassName` | Storage class | `""` |
| `resources.limits.memory` | Memory limit | `512Mi` |
| `resources.limits.cpu` | CPU limit | `500m` |
| `mariadb.enabled` | Deploy MariaDB | `true` |

## Example: Production with TLS

```yaml
# values-prod.yaml
ingress:
  enabled: true
  className: nginx
  host: blog.mysite.com
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod

ingressTLS:
  enabled: true
  secretName: ghost-tls

mail:
  enabled: true
  from: "'My Blog' <noreply@mysite.com>"
  options:
    host: smtp.mailgun.org
    port: 587
  auth:
    existingSecret: ghost-mail-secret

resources:
  limits:
    memory: 1Gi
    cpu: "1"
  requests:
    memory: 512Mi
    cpu: 250m
```
