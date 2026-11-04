# Rôle : postgresql_app_db

Crée les bases applicatives sur le serveur PostgreSQL (propriétaire, mot de passe) et ouvre l'accès dans `pg_hba.conf` uniquement depuis les hôtes autorisés.

**Playbook** : `postgresql-app-db.yml` · **Cible** : `bdd-srv`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `postgresql_app_dbs` | `[]` | Liste des bases à créer (dans `host_vars/bdd-srv/vars.yml`) |
| `postgresql_hba_file` | `/etc/postgresql/16/main/pg_hba.conf` | Fichier d'accès |
| `postgresql_service` | `postgresql@16-main` | Service rechargé après modification |

## Exemple

Chaque base précise son propriétaire et les hôtes autorisés ; les mots de passe proviennent du Vault de `bdd-srv`.

```yaml
postgresql_app_dbs:
  - name: js7
    owner: js7
    allowed_hosts:
      - 172.27.248.112
```

```bash
ansible-playbook postgresql-app-db.yml
```

Bases gérées : `grafana`, `jumpserver`, `js7`.
