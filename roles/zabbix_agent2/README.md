# Rôle : zabbix_agent2

Installe **Zabbix Agent 2** depuis le dépôt officiel Zabbix et configure les serveurs autorisés pour les checks passifs.

**Playbook** : `zabbix-agent2.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `zabbix_major` | `7.0` | Version majeure (LTS) |
| `zabbix_agent2_version` | `1:7.0.31-1+debian13` | Version du paquet |
| `zabbix_release_url` | dépôt `repo.zabbix.com` | Paquet `zabbix-release` adapté à la version Debian |
| `zabbix_agent2_servers` | IP des nœuds du groupe `swarm` | Sources autorisées (calculées depuis l'inventaire) |
| `zabbix_agent2_extra_servers` | `[]` | Sources supplémentaires |
| `zabbix_agent2_userparameters` | `[]` | UserParameters personnalisés |

## Particularité Docker Swarm

Le serveur Zabbix tourne dans Swarm. Les requêtes vers les hôtes externes sont traduites (SNAT `docker_gwbridge`) et arrivent avec l'IP d'un nœud, d'où la liste des nœuds Swarm dans `Server=`. Sur un nœud Swarm, la requête vers son propre agent n'est pas traduite et arrive avec l'IP `docker_gwbridge` : ajoutez-la dans `zabbix_agent2_extra_servers` pour ces nœuds.

## Exemple

```yaml
zabbix_agent2_userparameters:
  - key: "custom.uptime"
    command: "cat /proc/uptime | cut -d' ' -f1"
```

```bash
ansible-playbook zabbix-agent2.yml
```
