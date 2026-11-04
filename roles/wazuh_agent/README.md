# Rôle : wazuh_agent

Installe l'agent **Wazuh** (SIEM / XDR) sur les hôtes Linux et Windows et l'enregistre auprès du manager.

**Playbook** : `wazuh-agent.yml`

## Variables

| Variable | Défaut | Description |
|---|---|---|
| `wazuh_agent_version` | `4.14.8-1` | Version de l'agent |
| `wazuh_manager_address` | `{{ hostvars['swarm-wrk02']['ansible_host'] }}` | Adresse du manager |
| `wazuh_agent_msi_url` | `https://packages.wazuh.com/4.x/windows/wazuh-agent-<version>.msi` | Installateur Windows |

Sous Linux, l'agent est installé depuis le dépôt APT officiel de Wazuh.

## Dépendances

- Rôle `wazuh_manager` (manager publié en mode `host` sur `swarm-wrk02`)

## Exemple

```bash
ansible-playbook wazuh-agent.yml
```
