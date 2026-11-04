# Rôle : grafana_stack

Déploie **Grafana** sur Docker Swarm : HTTPS avec la CA interne, base PostgreSQL externe, envoi d'e-mails par relais SMTP et plugins préinstallés.

**Playbook** : `grafana-stack.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `grafana_version` | `13.2.4` | Version de l'image |
| `grafana_stack_name` | `grafana` | Nom de la stack Swarm |
| `grafana_secrets_version` | `1` | À incrémenter pour publier un nouveau secret |
| `grafana_node` | `swarm-mgr01` | Nœud d'épinglage (volume local des plugins) |
| `grafana_https_port` | `3000` | Port publié (ingress) |
| `grafana_root_url` | `https://172.27.248.110:3000/` | URL publique |
| `grafana_tls_cert_secret` / `grafana_tls_key_secret` | `infra_tls_*_20290113` | Secrets Swarm TLS (créés hors Ansible) |
| `grafana_db_host` | `""` | Hôte PostgreSQL (inventaire) |
| `grafana_db_port` / `grafana_db_name` / `grafana_db_user` | `5432` / `grafana` / `grafana` | Base de données |
| `grafana_smtp_host` | `smtp-relay.gmail.com:587` | Relais SMTP (STARTTLS) |
| `grafana_smtp_from_address` | `supervision@example.com` | Expéditeur |
| `grafana_smtp_from_name` | `Supervision Example` | Nom de l'expéditeur |
| `grafana_smtp_ehlo_identity` | `example.com` | Identité EHLO |
| `grafana_plugins_preinstall` | `alexanderzobnin-zabbix-app@6.9.1` | Plugins installés au démarrage |

## Secrets (à définir dans l'inventaire / Vault)

- `grafana_db_password`
- `grafana_admin_password`

Ils sont publiés comme secrets Docker et lus par Grafana via les variables `__FILE`.

## Dépendances

- Rôle `postgresql_app_db` (base `grafana` sur bdd-srv)
- Secrets TLS Swarm créés au préalable

## Exemple

```bash
ansible-playbook grafana-stack.yml
```
