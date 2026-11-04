# Rôle : wazuh_manager

Gère le **manager Wazuh**, publié en mode réseau `host` sur le nœud Swarm portant le label `wazuh.role=manager` (`swarm-wrk02`). L'indexer Wazuh est hébergé sur `swarm-wrk01`.

## Placement

| Composant | Nœud |
|---|---|
| Manager | `swarm-wrk02` (label `wazuh.role=manager`) |
| Indexer | `swarm-wrk01` |

## Dépendances

Les agents sont déployés par le rôle `wazuh_agent`, qui pointe vers l'adresse de `swarm-wrk02`.
