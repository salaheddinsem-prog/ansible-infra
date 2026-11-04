# Rôle : grafana_zabbix_datasource

Active le plugin **Zabbix** dans Grafana et crée la datasource associée, authentifiée par un jeton API Zabbix en lecture seule.

**Playbook** : `grafana-zabbix-datasource.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `grafana_zbx_api_url` | `https://172.27.248.110:3000` | URL de l'API Grafana |
| `grafana_zbx_api_token` | `{{ vault_grafana_sa_token }}` | Jeton du compte de service Grafana (Vault) |
| `grafana_zbx_app_id` | `alexanderzobnin-zabbix-app` | Identifiant du plugin |
| `grafana_zbx_ds_uid` | `zabbix-infra` | UID fixe de la datasource |
| `grafana_zbx_ds_name` | `Zabbix` | Nom affiché |
| `grafana_zbx_ds_url` | `https://172.27.248.110:8443/api_jsonrpc.php` | API Zabbix |
| `grafana_zbx_ds_api_token` | `{{ vault_zabbix_grafana_api_token }}` | Jeton du compte Zabbix `grafana-api` (Vault) |
| `grafana_zbx_ds_ca_src` | `/root/ca-interne/ca/ca-v2.crt` | CA interne |
| `grafana_zbx_ds_force_update` | `false` | Réécrit une datasource existante |

## Dépendances

- Rôle `grafana_stack` (plugin préinstallé via `grafana_plugins_preinstall`)

## Exemple

```bash
ansible-playbook grafana-zabbix-datasource.yml
```
