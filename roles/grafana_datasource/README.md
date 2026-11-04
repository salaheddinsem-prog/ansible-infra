# Rôle : grafana_datasource

Crée la datasource **Prometheus** dans Grafana via l'API (HTTPS, authentification basique, CA interne).

**Playbook** : `grafana-datasource.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `grafana_api_url` | `https://172.27.248.110:3000` | URL de l'API Grafana |
| `grafana_api_token` | `{{ vault_grafana_sa_token }}` | Jeton du compte de service (Vault) |
| `grafana_ds_uid` | `prometheus-infra` | UID fixe de la datasource |
| `grafana_ds_name` | `Prometheus` | Nom affiché |
| `grafana_ds_url` | `https://172.27.248.111:9090` | URL de Prometheus |
| `grafana_ds_user` | `grafana` | Utilisateur basic auth |
| `grafana_ds_password` | `{{ vault_prometheus_grafana_password }}` | Mot de passe (Vault) |
| `grafana_ds_ca_src` | `/root/ca-interne/ca/ca-v2.crt` | CA interne transmise à Grafana |
| `grafana_ds_force_update` | `false` | Réécrit une datasource existante |

## Exemple

```bash
ansible-playbook grafana-datasource.yml
```
