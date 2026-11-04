# Rôle : grafana_dashboards

Importe les dashboards de supervision dans Grafana via l'API, dans un dossier dédié.

**Playbook** : `grafana-dashboards.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `grafana_api_url` | `https://172.27.248.110:3000` | URL de l'API Grafana |
| `grafana_api_token` | `{{ vault_grafana_sa_token }}` | Jeton du compte de service (Vault) |
| `grafana_dashboards_folder_uid` | `infra` | UID du dossier |
| `grafana_dashboards_folder_title` | `Infra` | Titre du dossier |
| `grafana_dashboards` | voir ci-dessous | Fichiers à importer depuis `files/` |
| `grafana_dashboards_force` | `false` | Réimporte les dashboards existants |

Dashboards fournis :

- `node-exporter-full.json` : serveurs Linux (dashboard communautaire 1860)
- `windows-servers.yml` : serveurs Windows
- `proxmox-hyperviseur.yml` : hyperviseur Proxmox

Chaque fichier doit avoir un `uid` fixe pour garantir l'idempotence.

## Exemple

```bash
ansible-playbook grafana-dashboards.yml
```
