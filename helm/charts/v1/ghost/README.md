# Helm chart for Ghost CMS

## Prerequisites

### Duild dependencies

```bash
cd helm/charts/v1/ghost
helm dependency build .
```

### Prepare secrets

```bash
MARIADB_ROOT_PASSWORD='your_password_here'
MARIADB_PASSWORD='your_password_here'
kubectl create secret generic ghost-mariadb-secret \
  --from-literal=mariadb-root-password=$MARIADB_ROOT_PASSWORD \
  --from-literal=mariadb-password=$MARIADB_PASSWORD
```
