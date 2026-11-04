# Rôle : nftables

Configure le pare-feu **nftables** des hôtes, en n'autorisant les accès d'administration qu'à des ensembles d'adresses définis.

**Playbook** : `nftables.yml`

## Ensembles d'adresses

| Ensemble | Contenu |
|---|---|
| `ADMIN_NET` | Accès d'administration (VPN, poste de bureau) |
| `ANSIBLE_CTRL` | Contrôleur Ansible |
| `SWARM_NODES` | Nœuds du cluster Docker Swarm |

## Principe

- Politique par défaut restrictive en entrée.
- SSH autorisé uniquement depuis `ADMIN_NET` et `ANSIBLE_CTRL`.
- Flux applicatifs (PostgreSQL, supervision) limités aux nœuds qui en ont besoin.

## Exemple

```bash
ansible-playbook nftables.yml --check --diff   # vérifier avant d'appliquer
ansible-playbook nftables.yml
```

> Toujours tester avec `--check --diff` et garder une session ouverte : une règle erronée peut couper l'accès SSH.
