# Rôle : jumpserver_stack

Déploie le bastion d'accès **JumpServer** (édition communautaire) sur Docker Swarm, avec Redis, base PostgreSQL externe et HTTPS via la CA interne.

**Playbook** : `jumpserver-stack.yml`

## Composants et placement

| Composants | Nœud | Raison |
|---|---|---|
| core, celery, web, redis | `swarm-wrk02` | Partagent les mêmes données locales |
| koko (SSH), lion (RDP/VNC), chen (bases de données) | `swarm-mgr01` | Connecteurs d'accès |

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `jumpserver_version` | `v4.10.19-ce` | Version des images |
| `jumpserver_redis_image` | `redis:7.4-alpine` | Image Redis |
| `jumpserver_stack_name` | `jumpserver` | Nom de la stack Swarm |
| `jumpserver_secrets_version` | `3` | À incrémenter pour publier un nouveau secret |
| `jumpserver_node_core` | `swarm-wrk02` | Nœud des composants principaux |
| `jumpserver_node_connectors` | `swarm-mgr01` | Nœud des connecteurs |
| `jumpserver_data_root` | `/srv/jms` | Données persistantes sur l'hôte |
| `jumpserver_http_port` | `8082` | Port HTTP |
| `jumpserver_https_port` | `8444` | Port HTTPS |
| `jumpserver_ssh_port` | `2222` | Port SSH (koko) |
| `jumpserver_tls_cert_secret` / `jumpserver_tls_key_secret` | `infra_tls_*_20290113` | Secrets Swarm TLS |
| `jumpserver_web_https_conf_version` | `1` | Version de la config nginx (immuable) |
| `jumpserver_db_host` / `_port` / `_name` / `_user` | `""` / `5432` / `jumpserver` / `jumpserver` | Base de données |
| `jumpserver_domains` | `""` | Domaines et adresses autorisés (inventaire) |
| `jumpserver_chen_memory_limit` | `1G` | Limite mémoire de chen |
| `jumpserver_log_level` | `ERROR` | Niveau de logs |
| `jumpserver_component_registration` | `false` | Enregistrement des composants : `false`, `auto` ou `true` |

## Secrets (à définir dans l'inventaire / Vault)

- `jumpserver_db_password`
- `jumpserver_secret_key`
- `jumpserver_bootstrap_token`
- `jumpserver_redis_password`

## Dépendances

- Rôle `postgresql_app_db` (base `jumpserver` sur bdd-srv)
- Secrets TLS Swarm créés au préalable

## Exemple

```bash
ansible-playbook jumpserver-stack.yml
```
