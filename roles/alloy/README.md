# Rôle : alloy

Installe et configure l'agent **Grafana Alloy** sur les hôtes Linux et Windows. Les métriques sont envoyées à Prometheus en `remote_write` (HTTPS, authentification basique, CA interne).

**Playbook** : `alloy.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `alloy_version` | `1.21.2` | Version épinglée (sans le `v`) |
| `alloy_prometheus_url` | `https://172.27.248.111:9090/api/v1/write` | Point d'écriture Prometheus |
| `alloy_prometheus_user` | `alloy` | Utilisateur basic auth |
| `alloy_prometheus_password` | `{{ vault_prometheus_alloy_password }}` | Mot de passe (Vault) |
| `alloy_scrape_interval` | `15s` | Intervalle de collecte |
| `alloy_ca_file_linux` | `/usr/local/share/ca-certificates/infra-ops-ca.crt` | CA interne (Linux) |
| `alloy_ca_file_windows` | `C:/ProgramData/Infra-ops/infra-ops-ca.crt` | CA interne (Windows) |
| `alloy_windows_install_dir` | `C:\Program Files\GrafanaLabs\Alloy` | Dossier d'installation |
| `alloy_windows_service` | `Alloy` | Nom du service Windows |
| `alloy_windows_data_dir` | `C:\ProgramData\Infra-ops` | Données locales (mot de passe) |
| `alloy_windows_collectors` | cpu, memory, logical_disk, net, os, service… | Collecteurs `windows_exporter` activés |

## Dépendances

- Rôle `ca_trust` (racine de la CA interne déposée sur l'hôte)
- Rôle `prometheus_stack` (récepteur `remote_write` activé)

## Exemple

```bash
ansible-playbook alloy.yml --limit windows
```
