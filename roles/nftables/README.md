# Rôle : nftables

Déploie un pare-feu **nftables** par hôte, à partir d'un modèle Jinja2 dédié, avec une politique « tout bloquer sauf ce qui est explicitement autorisé ». Les accès sont regroupés dans des ensembles d'adresses nommés pour rester lisibles.

**Playbook** : `nftables.yml`

## Structure

```
roles/nftables/
├── handlers/      # rechargement de nftables après modification
├── tasks/
└── templates/
    └── bdd-srv.nft.j2   # un modèle par hôte protégé
```

## Ensembles d'adresses

| Ensemble | Adresses | Usage |
|---|---|---|
| `ADMIN_NET` | 172.27.248.100, 172.27.248.190, 172.27.248.195 | Administration : VPN via NAT du pare-feu, VPN, poste de bureau |
| `ANSIBLE_CTRL` | 172.27.248.114 | Contrôleur Ansible (maestro) |
| `SWARM_NODES` | 172.27.248.110, 172.27.248.111, 172.27.248.112 | Nœuds du cluster Docker Swarm |

## Règles appliquées à bdd-srv

Chaîne `input`, politique par défaut **drop** :

| Port | Source | Usage |
|---|---|---|
| — | `lo` | Boucle locale |
| — | connexions établies / liées | Réponses aux flux sortants |
| ICMP / ICMPv6 | toutes | Diagnostic réseau |
| 22/tcp | `ADMIN_NET`, `ANSIBLE_CTRL` | SSH d'administration et d'automatisation |
| 5432/tcp | `SWARM_NODES` | PostgreSQL (Grafana, JumpServer, JS7) |
| 10050/tcp | `SWARM_NODES` | Zabbix Agent 2 (checks passifs) |

Les paquets en état `invalid` sont rejetés, et les tentatives SSH refusées sont journalisées avec le préfixe `ssh-drop`. La chaîne `forward` est en **drop** (le serveur ne route rien) et la chaîne `output` en **accept**.

Le modèle commence par `flush ruleset` : le fichier décrit l'intégralité du pare-feu, aucune règle résiduelle ne subsiste.

## Ajouter un hôte

1. Créer `templates/<hôte>.nft.j2` en reprenant les ensembles d'adresses et en adaptant les ports ouverts.
2. Ajouter l'hôte à la cible du playbook `nftables.yml`.

## Utilisation

```bash
ansible-playbook nftables.yml --check --diff   # vérifier avant d'appliquer
ansible-playbook nftables.yml
```

Consulter les tentatives SSH refusées :

```bash
journalctl -k | grep ssh-drop
```

> Toujours tester avec `--check --diff` et garder une session SSH ouverte pendant l'application : une erreur dans `ADMIN_NET` ou `ANSIBLE_CTRL` peut couper l'accès au serveur.
