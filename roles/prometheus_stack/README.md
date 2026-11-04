# Rôle : prometheus_stack

Déploie **Prometheus** sur Docker Swarm en HTTPS avec authentification basique, et active le récepteur `remote_write` utilisé par les agents Alloy.

**Playbook** : `prometheus-stack.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `prometheus_version` | `v3.13.5` | Version LTS 3.13 |
| `prometheus_stack_name` | `prometheus` | Nom de la stack Swarm |
| `prometheus_node` | `swarm-wrk01` | Nœud d'épinglage (TSDB sur volume local, jamais sur NFS) |
| `prometheus_https_port` | `9090` | Port HTTPS |
| `prometheus_external_url` | `https://172.27.248.111:9090/` | URL publique |
| `prometheus_retention_time` | `30d` | Rétention en durée |
| `prometheus_retention_size` | `5GB` | Rétention en taille (la première limite atteinte s'applique) |
| `prometheus_memory_limit` | `1G` | Limite mémoire (nœud partagé avec l'indexer Wazuh) |
| `prometheus_memory_reservation` | `512M` | Réservation mémoire |
| `prometheus_scrape_interval` | `15s` | Intervalle de collecte |
| `prometheus_tls_cert_secret` / `prometheus_tls_key_secret` | `infra_tls_*_20290113` | Secrets Swarm TLS |
| `prometheus_basic_auth_users` | `{}` | Comptes `{nom: hash bcrypt}` (inventaire) |

## Exemple

```yaml
prometheus_basic_auth_users:
  alloy: "$2y$10$..."
  grafana: "$2y$10$..."
```

```bash
ansible-playbook prometheus-stack.yml
```
