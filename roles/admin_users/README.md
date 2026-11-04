# Rôle : admin_users

Crée les comptes administrateurs nominatifs (utilisateur, groupe, clé SSH) sur les nœuds Swarm et les serveurs de bases de données.

**Playbook** : `admin_users.yml` · **Cibles** : `swarm`, `databases`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `admin_users_enforce_ids` | `true` | Impose les UID/GID définis (harmonisation entre hôtes) |
| `admin_users` | voir ci-dessous | Liste des comptes à créer |

Chaque compte accepte `name`, `uid`, `gid` et `ssh_key`. Un mot de passe peut être fourni via une variable Vault nommée `<name>_password_hash`.

## Exemple

```yaml
admin_users:
  - name: jdupont
    uid: 1002
    gid: 1002
    ssh_key: "ssh-ed25519 AAAA... jdupont@poste"
```

```bash
ansible-playbook admin_users.yml
```
