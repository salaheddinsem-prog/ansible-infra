# Rôle : ca_trust

Déploie la racine de la **CA interne** dans le magasin de confiance des hôtes Linux et Windows, pour que les services en HTTPS (Grafana, Prometheus, Zabbix, JumpServer) soient reconnus.

**Playbook** : `ca-trust.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `ca_trust_src` | `/root/ca-interne/ca/ca-v2.crt` | Racine de la CA sur le contrôleur |
| `ca_trust_name` | `infra-ops-ca` | Nom du certificat déployé |
| `ca_trust_windows_dir` | `C:\ProgramData\Infra-ops` | Dossier de dépôt sur Windows |

## Exemple

```bash
ansible-playbook ca-trust.yml
```
