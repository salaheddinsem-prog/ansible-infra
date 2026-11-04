# Rôle : wazuh_manager

Règles Wazuh personnalisées pour l'analyse des journaux du pare-feu **FortiGate**, à charger sur le manager Wazuh.

Le manager est déployé sur Docker Swarm, publié en mode réseau `host` sur le nœud portant le label `wazuh.role=manager` (`swarm-wrk02`). L'indexer est hébergé sur `swarm-wrk01`. Les agents sont installés par le rôle `wazuh_agent`.

## Contenu

```
roles/wazuh_manager/
└── files/
    └── rules/
        └── fortigate_rules.xml
```

Ce rôle ne contient pas de tâches : le fichier de règles est déployé manuellement dans le conteneur du manager (voir plus bas).

## Objectif

Le décodeur FortiGate fourni par Wazuh génère beaucoup de bruit (chaque application autorisée, chaque session). Ces règles réduisent ce bruit tout en gardant visibles les événements utiles à la sécurité : accès à distance, usage d'IA générative, appareils qui insistent sur des sites bloqués et transferts volumineux vers Internet.

## Règles

| ID | Niveau | Parent | Déclencheur |
|---|---|---|---|
| 100500 | 5 | 81603 | Échec d'authentification SNMP sur le FortiGate |
| 81633 | 0 | 81603 | **Surcharge** : trafic applicatif autorisé, plus d'alerte (définition d'origine conservée) |
| 100501 | 6 | 81633 | Application d'accès à distance autorisée (catégorie `Remote.Access`) |
| 100502 | 5 | 81633 | Application d'IA générative utilisée (catégorie `GenAI`) |
| 100503 | 8 | 81644 | 20 requêtes web bloquées ou plus depuis la même IP en 10 minutes |
| 100504 | 6 | 81618 | Envoi de 100 Mo ou plus vers le WAN dans une session |
| 100505 | 5 | 81618 | Réception de 100 Mo ou plus depuis le WAN dans une session |

Détails de conception :

- **Surcharge de 81633** (`overwrite="yes"`) : la règle d'origine est reprise à l'identique, seul le niveau passe à 0. Ses règles enfants 100501 et 100502 continuent de fonctionner et font ressortir les catégories intéressantes.
- **Corrélation 100503** : chaque blocage web (81644) reste visible individuellement ; la règle ajoute une alerte de niveau supérieur quand une même source insiste. `ignore="600"` évite de répéter l'alerte pendant 10 minutes.
- **Gros transferts 100504 / 100505** : seuil de 100 Mo, exprimé par un nombre d'octets d'au moins 9 chiffres. Seul le trafic vers l'interface de rôle `wan` est concerné, et les journaux de `logid` `0000000020` sont exclus. L'upload est déclaré en premier : si une session dépasse le seuil dans les deux sens, c'est l'alerte d'envoi, la plus sensible, qui est retenue.

## Prérequis

- Le FortiGate envoie ses journaux en syslog vers le manager Wazuh.
- Les identifiants personnalisés utilisent la plage `100500` à `100505` ; vérifier qu'ils ne sont pas déjà pris par d'autres règles locales.

## Déploiement

Sur `swarm-wrk02` :

```bash
# Identifier le conteneur du manager
CID=$(docker ps --filter name=wazuh --format '{{.ID}} {{.Names}}' | grep -i manager | cut -d' ' -f1)

# Copier les règles et appliquer les droits attendus par Wazuh
docker cp fortigate_rules.xml "$CID":/var/ossec/etc/rules/fortigate_rules.xml
docker exec "$CID" chown wazuh:wazuh /var/ossec/etc/rules/fortigate_rules.xml
docker exec "$CID" chmod 660 /var/ossec/etc/rules/fortigate_rules.xml

# Redémarrer l'analyse
docker exec "$CID" /var/ossec/bin/wazuh-control restart
```

Pour que les règles survivent à une recréation du conteneur, le dossier `/var/ossec/etc` doit être sur un volume persistant.

## Tester une règle

```bash
docker exec -it "$CID" /var/ossec/bin/wazuh-logtest
```

Coller ensuite une ligne de journal FortiGate : l'outil affiche le décodeur utilisé, la règle déclenchée et son niveau.
