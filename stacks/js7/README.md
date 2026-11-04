# JS7 JobScheduler — instance de découverte (Infra locale)

## 1. Objet et statut

- Instance d'APPRENTISSAGE de JS7 (workflows, jobs, Agents, planification). Ce n'est PAS la prod.
- Décision d'un usage réel : à prendre avec le N+1. Lire la section 8 (points ouverts) avant toute mise en production.
- Installée le 2026-11-04.

## 2. Architecture

| Élément | Valeur |
|---|---|
| Version | JS7 2.9.1 (`sosberlin/js7:joc-2-9-1`, `sosberlin/js7:controller-2-9-1`) |
| Nœud | `swarm-wrk02` (172.27.248.112), épinglé, 1 réplica par service, pas de HA |
| Stack Portainer | `js7` : services `js7_joc`, `js7_controller`, réseau `js7_js7` |
| Accès JOC | https://172.27.248.112:8445 (8445 publié vers 4443, routing mesh : joignable via n'importe quel nœud) |
| JOC vers Controller | `http://controller:4444` dans l'overlay, port non publié |
| Base | PostgreSQL `js7` sur bdd-srv (172.27.248.150), accès uniquement depuis 172.27.248.112 |
| Controller ID | `js7-local` (définitif, inscrit dans le journal du Controller) |
| Utilisateur des conteneurs | `RUN_JS_USER_ID=1500:0` (l'UID 1000 de l'image est le compte personnel `adminuser` sur wrk02) |
| Mémoire | JOC `-Xmx512m`, limite 1280M ; Controller `-Xmx256m`, limite 768M |

## 3. Données et secrets

| Chemin sur wrk02 | Monté sur | Contenu |
|---|---|---|
| `/srv/js7/joc/config` | `/var/sos-berlin.com/js7/joc/resources/joc` | config JOC : hibernate, joc.properties, magasins TLS, start.d/ssl-context.ini |
| `/srv/js7/controller/data` | `/var/sos-berlin.com/js7/controller` (= `JS7_CONTROLLER_DATA`) | config, state (journal), logs du Controller |

Secrets : les valeurs sont uniquement dans le Vault, jamais dans Git.

| Variable Vault | Fichier Vault | Utilisée dans (sur wrk02) |
|---|---|---|
| `vault_js7_db_password` | `inventory/host_vars/bdd-srv/vault.yml` | `hibernate.cfg.xml` |
| `vault_js7_keystore_password` | `inventory/group_vars/swarm/vault.yml` | `https-*.p12`, `start.d/ssl-context.ini` (3 valeurs), `joc.properties` (3 valeurs) |

Sur wrk02, ces fichiers contiennent les mots de passe en clair (simplification acceptée pour une instance de découverte) : propriétaire `1500:0`, droits `660`, dossiers parents en `750 root`. Ne jamais faire `cat` dessus.

## 4. Règle de déploiement

- Git (ce dossier) = référence. Portainer = outil de déploiement.
- Toute modification : d'abord `js7-stack.yml` dans Git (commit), puis coller dans Portainer (Stacks > js7 > Editor > Update the stack, sans « Re-pull image »).
- Ne JAMAIS redéployer cette stack par `docker stack deploy` ni par Ansible.
- Redémarrer un service sans changer sa définition : `docker service update --force js7_joc` (ne touche pas à la stack).
- Les commandes `ansible` et `ansible-vault` se lancent depuis `/root/ansible-infra` : c'est le `ansible.cfg` du projet qui charge `/root/.vault_pass`.

## 5. Installation (procédure suivie)

### 5.1 Base PostgreSQL (maestro, Ansible)

Entrée `js7` dans `inventory/host_vars/bdd-srv/vars.yml` (rôle `postgresql_app_db`, owner `js7`, `allowed_hosts` : 172.27.248.112). Commit `f554d0a`.

### 5.2 Dossiers (wrk02)

```bash
install -d -o root -g root -m 750 /srv/js7 /srv/js7/joc /srv/js7/controller
install -d -o 1500 -g 0 -m 770 /srv/js7/joc/config /srv/js7/controller/data
```

### 5.3 Pré-remplir la config de JOC depuis l'image (wrk02)

Un montage vide masquerait les fichiers par défaut de l'image.

```bash
docker run --rm --network none --entrypoint /bin/sh \
  -v /srv/js7/joc/config:/target sosberlin/js7:joc-2-9-1 \
  -c 'cp -a /var/sos-berlin.com/js7/joc/resources/joc/. /target/'
chown -R 1500:0 /srv/js7/joc/config
chmod -R o= /srv/js7/joc/config
```

### 5.4 hibernate.cfg.xml (wrk02, puis maestro)

Sur wrk02 : sauvegarde hors de `config/`, puis passage de MariaDB à PostgreSQL, avec un marqueur à la place du mot de passe.

```bash
cp -a /srv/js7/joc/config/hibernate.cfg.xml /srv/js7/joc/hibernate.cfg.xml.orig-mariadb
sed -i \
  -e 's|\(name="hibernate.connection.driver_class">\)[^<]*|\1org.postgresql.Driver|' \
  -e 's|\(name="hibernate.connection.url">\)[^<]*|\1jdbc:postgresql://172.27.248.150:5432/js7|' \
  -e 's|\(name="hibernate.connection.username">\)[^<]*|\1js7|' \
  -e 's|\(name="hibernate.connection.password">\)[^<]*|\1__JS7_DB_PASSWORD__|' \
  -e 's|\(name="hibernate.dialect">\)[^<]*|\1org.hibernate.dialect.PostgreSQLDialect|' \
  /srv/js7/joc/config/hibernate.cfg.xml
```

Sur maestro : injection depuis le Vault (d'abord avec `--check`, puis sans). Jamais `--diff` ni `-v`.

```bash
ansible swarm-wrk02 -b -m ansible.builtin.replace \
  -a "path=/srv/js7/joc/config/hibernate.cfg.xml regexp=__JS7_DB_PASSWORD__ replace={{ hostvars['bdd-srv'].vault_js7_db_password }}"
```

### 5.5 Magasins TLS PKCS12 (maestro)

Certificat existant de la CA interne : `serveur/infra.crt` (clé EC, SAN 172.27.248.110/.111/.112, valide jusqu'au 2029-01-13), racine `ca/ca-v2.crt`. Le mot de passe est lu dans le Vault vers un fichier en RAM, jamais affiché.

```bash
mkdir -m 700 /dev/shm/js7-tls
cd /root/ansible-infra
(umask 077; ansible-vault view inventory/group_vars/swarm/vault.yml \
  | sed -n 's/^vault_js7_keystore_password: "\([0-9a-f]*\)"$/\1/p' > /dev/shm/js7-tls/pass)
cd /root/ca-interne
openssl pkcs12 -export -in serveur/infra.crt -inkey serveur/infra.key -certfile ca/ca-v2.crt \
  -name js7-joc -passout file:/dev/shm/js7-tls/pass -out /dev/shm/js7-tls/https-keystore.p12
openssl pkcs12 -export -nokeys -in ca/ca-v2.crt -caname ca-infra-ops-v2 \
  -jdktrust anyExtendedKeyUsage -passout file:/dev/shm/js7-tls/pass -out /dev/shm/js7-tls/https-truststore.p12
```

- `-caname` (et non `-name`) pour nommer un certificat sans clé.
- `-jdktrust` : sans cet attribut, Java voit le truststore comme vide.

Transfert vers wrk02, puis nettoyage :

```bash
cd /root/ansible-infra
ansible swarm-wrk02 -b -m ansible.builtin.copy \
  -a "src=/dev/shm/js7-tls/https-keystore.p12 dest=/srv/js7/joc/config/https-keystore.p12 owner=1500 group=0 mode=0660"
ansible swarm-wrk02 -b -m ansible.builtin.copy \
  -a "src=/dev/shm/js7-tls/https-truststore.p12 dest=/srv/js7/joc/config/https-truststore.p12 owner=1500 group=0 mode=0660"
shred -u /dev/shm/js7-tls/pass /dev/shm/js7-tls/https-keystore.p12 /dev/shm/js7-tls/https-truststore.p12
rmdir /dev/shm/js7-tls
```

### 5.6 TLS de JOC : serveur (Jetty) et client (JOC)

JOC a DEUX configurations TLS qui pointent sur les mêmes `.p12`. Les deux doivent avoir le mot de passe du Vault, sinon : `keystore password was incorrect` / `Client certificate not loaded` dans `joc-error.log`.

Sur wrk02, `ssl-context.ini` (serveur Jetty), recopié vers Jetty à chaque démarrage :

```bash
install -d -o 1500 -g 0 -m 770 /srv/js7/joc/config/start.d
docker run --rm --network none --entrypoint cat sosberlin/js7:joc-2-9-1 \
  /var/sos-berlin.com/js7/joc/start.d/ssl-context.ini \
  | install -o 1500 -g 0 -m 660 /dev/stdin /srv/js7/joc/config/start.d/ssl-context.ini
sed -i -E 's/^(jetty\.sslContext\.(keyStore|keyManager|trustStore)Password=).*/\1__JS7_KEYSTORE_PASSWORD__/' \
  /srv/js7/joc/config/start.d/ssl-context.ini
```

Sur wrk02, `joc.properties` (JOC en client, pour ses connexions HTTPS sortantes). Lors de l'installation initiale, cette étape a été faite après coup.

```bash
cp -a /srv/js7/joc/config/joc.properties /srv/js7/joc/joc.properties.orig
sed -i -E 's/^([[:space:]]*(truststore_password|keystore_password|key_password)[[:space:]]*=[[:space:]]*).*/\1__JS7_KEYSTORE_PASSWORD__/' \
  /srv/js7/joc/config/joc.properties
```

Sur maestro : injection (3 remplacements par fichier).

```bash
ansible swarm-wrk02 -b -m ansible.builtin.replace \
  -a "path=/srv/js7/joc/config/start.d/ssl-context.ini regexp=__JS7_KEYSTORE_PASSWORD__ replace={{ vault_js7_keystore_password }}"
ansible swarm-wrk02 -b -m ansible.builtin.replace \
  -a "path=/srv/js7/joc/config/joc.properties regexp=__JS7_KEYSTORE_PASSWORD__ replace={{ vault_js7_keystore_password }}"
```

### 5.7 Pré-remplir les données du Controller (wrk02)

Le Controller utilise `JS7_CONTROLLER_DATA=/var/sos-berlin.com/js7/controller` (fixé dans `controller_instance.sh`), et NON `.../controller/var`.

```bash
docker run --rm --network none --entrypoint /bin/sh \
  -v /srv/js7/controller/data:/target sosberlin/js7:controller-2-9-1 \
  -c 'cp -a /var/sos-berlin.com/js7/controller/. /target/'
chown -R 1500:0 /srv/js7/controller/data
chmod -R o= /srv/js7/controller/data
```

### 5.8 Déploiement

Portainer > Stacks > Add stack > nom `js7` > Web editor > coller `js7-stack.yml` > Deploy the stack. Le démarrage de JOC prend environ 3 min 30 (voir section 8).

### 5.9 Tables de JOC (wrk02, une seule fois)

```bash
docker exec -u jobscheduler $(docker ps -q -f name=js7_joc) \
  /opt/sos-berlin.com/js7/joc/install/joc_install_tables.sh 2>&1 \
  | tee /root/js7-install-tables.log | tail -40
grep -E '^\.\.\.|\[success=[0-9]+\]\[error=[0-9]+\]' /root/js7-install-tables.log
docker service update --force js7_joc
```

Résultat le 2026-11-04 : tables 12/12, modifications 13/13, procédures 4/4, données initiales 5/5, 0 erreur. Mode par défaut (`--install`) uniquement : ne jamais utiliser `--rescue`. JOC doit être redémarré après la création des tables.

### 5.10 Première connexion

- https://172.27.248.112:8445, `root` / `root`, mot de passe changé immédiatement.
- Enregistrement du Controller : URL `http://controller:4444`, Test Connection, puis Submit. L'ID `js7-local` est lu auprès du Controller.

## 6. Exploitation

```bash
# État des services (wrk02)
docker service ps js7_joc js7_controller --format 'table {{.Name}}\t{{.CurrentState}}\t{{.Error}}'

# Les logs Docker ne montrent que le script de démarrage ; les logs applicatifs sont dans des fichiers
docker service logs -t --tail 60 js7_joc
docker exec $(docker ps -q -f name=js7_joc) ls -l /var/log/sos-berlin.com/js7/joc/
tail -50 /srv/js7/controller/data/logs/controller.log

# Mémoire
docker stats --no-stream --format 'table {{.Name}}\t{{.MemUsage}}\t{{.MemPerc}}' | grep -E 'NAME|js7'
```

Mesure du 2026-11-04 (au repos, juste après l'installation) : JOC 476 Mo, Controller 230 Mo. La mémoire `available` de wrk02 est passée d'environ 3,0 Go à 1,9 Go.

## 7. Désinstallation

Rien d'autre n'est touché.

1. Portainer : Stacks > js7 > Delete.
2. wrk02, une fois la stack supprimée : `rm -r /srv/js7` et `rm /root/js7-install-tables.log`.
3. wrk02 : `docker image rm sosberlin/js7:joc-2-9-1 sosberlin/js7:controller-2-9-1`.
4. bdd-srv : supprimer la base et l'utilisateur `js7`, et l'entrée `js7` de `inventory/host_vars/bdd-srv/vars.yml` (vérifier si le rôle `postgresql_app_db` sait supprimer, sinon le faire à la main).
5. Vault : retirer `vault_js7_db_password` et `vault_js7_keystore_password`. Git : supprimer `stacks/js7/`.

## 8. Points ouverts

### Avant tout usage réel

- Le Controller est en `public=true` (`config/private/private.conf`) : il accepte toute requête sur le port 4444, sans authentification. Passer à `false` et changer les mots de passe par défaut des comptes `JOC` et `History`, côté Controller et côté JOC.

### Durcissement

- HTTPS entre JOC et le Controller ; chiffrement de l'overlay `js7`.
- Certificat client dédié pour JOC : le certificat `Outils` n'est prévu que pour l'usage serveur (`TLS Web Server Authentication`).
- `mode: ingress` : le journal d'audit de JOC voit une IP interne Swarm et non celle du poste client (mode `host` à envisager).

### Exploitation

- `RUN_JS_USER_ID=1500` : à chaque nouveau conteneur, JOC recopie environ 209 Mo de l'image (changement de propriétaire sur overlayfs). Conséquences : démarrage d'environ 3 min 30, et espace disque consommé par conteneur, y compris par les conteneurs arrêtés que Swarm conserve. Piste : une image dérivée avec l'UID 1500 intégré. Le Controller n'est pas concerné.
- Logs de JOC non persistants (`/var/log/sos-berlin.com/js7/joc`).
- Les fichiers créés par le Controller sont en `644` (umask du processus) ; sans risque grâce aux droits des dossiers parents.
- Mémoire : refaire la mesure après les premiers workflows (pire cas : environ 600 Mo disponibles sur wrk02, sans swap).
- Avertissement `Job Resource eMailDefault not found` : les notifications par e-mail ne sont pas configurées.

### Hors installation

- L'enregistrement DNS `js7.example.org` pointe vers le site OVH : un JS7 y existe peut-être (à demander).

## 9. Pièges rencontrés

- Le dossier de données du Controller est `.../js7/controller`, et non `.../controller/var` (vérifié avec `ps -ef` dans le conteneur).
- Un montage vide masque les fichiers par défaut de l'image : toujours pré-remplir depuis l'image.
- `--check` d'Ansible n'écrit rien : relancer sans `--check`, puis vérifier sur la cible.
- Les commandes `ansible` et `ansible-vault` lancées en dehors de `/root/ansible-infra` ne chargent pas `/root/.vault_pass`.
- Jetty redirige ses logs vers un fichier : Portainer et `docker logs` ne montrent que le script de démarrage.
- Après la création des tables, JOC doit être redémarré.
