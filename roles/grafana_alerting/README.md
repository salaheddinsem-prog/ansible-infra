# Rôle : grafana_alerting

Configure l'alerting Grafana via l'API : point de contact e-mail, politique de notification et groupe de règles d'alerte.

**Playbook** : `grafana-alerting.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `grafana_api_url` | `https://172.27.248.110:3000` | URL de l'API Grafana |
| `grafana_api_token` | `{{ vault_grafana_sa_token }}` | Jeton du compte de service `ansible` (Vault) |
| `grafana_alerting_contact_uid` | `supervision-email` | UID du point de contact |
| `grafana_alerting_contact_name` | `Supervision (email)` | Nom affiché |
| `grafana_alerting_contact_addresses` | `supervision@example.com` | Destinataires (groupe de diffusion) |
| `grafana_alerting_group_by` | `[grafana_folder, alertname]` | Regroupement des notifications |
| `grafana_alerting_group_wait` | `30s` | Attente avant le premier envoi |
| `grafana_alerting_group_interval` | `5m` | Intervalle entre envois d'un même groupe |
| `grafana_alerting_repeat_interval` | `4h` | Répétition d'une alerte active |
| `grafana_alerting_folder_uid` | `infra` | Dossier des règles |
| `grafana_alerting_rule_group` | `infrastructure` | Nom du groupe de règles |
| `grafana_alerting_rule_interval` | `60` | Évaluation (secondes) |
| `grafana_alerting_rules_file` | `rules.yml` | Fichier des règles dans `files/` |
| `grafana_alerting_force` | `false` | Réécrit le groupe même sans changement |

## Dépendances

- Rôle `grafana_stack` (SMTP configuré pour l'envoi des e-mails)
- Rôle `grafana_datasource` (règles basées sur Prometheus)

## Exemple

```bash
ansible-playbook grafana-alerting.yml -e grafana_alerting_force=true
```
