# **TP2 – Réplication et Tolérance aux Pannes avec MongoDB**

# Introduction

La réplication est un mécanisme essentiel dans les bases de données distribuées.
Elle permet :

* la **haute disponibilité**,
* la **tolérance aux pannes**,
* la **sécurité des données**,
* la **continuité de service**,
* une **meilleure répartition des charges**.

MongoDB utilise un système appelé **Replica Set** pour répliquer automatiquement les données entre plusieurs serveurs.
Ce rapport explique :

* le principe général de la réplication,
* un résumé clair des vidéos de démonstration,
* des schémas ASCII pour visualiser les mécanismes,
* puis les réponses détaillées à toutes les questions du TP.

---

#   Partie 1 — Compréhension du Replica Set

##  + Qu’est-ce qu’un Replica Set ?

Un **Replica Set** est un groupe de processus MongoDB contenant les mêmes données.
Il comporte au minimum :

* **1 Primary**
* **1 ou plusieurs Secondaries**
* **0 ou 1 Arbiter**

### ✔ Rôle des nœuds

| Type de nœud  | Rôle                                                         |
| ------------- | ------------------------------------------------------------ |
| **Primary**   | reçoit toutes les écritures et réplique vers les secondaires |
| **Secondary** | copie les données du Primary et peut répondre à des lectures |
| **Arbiter**   | ne stocke rien, vote aux élections                           |

---

## ##  Schéma du fonctionnement d’un Replica Set

```
                +--------------+
                |    Client    |
                +------+-------+
                       |
                       v
                 +-----+------+
                 |   PRIMARY  |
                 +-----+------+
                       |
        +--------------+--------------+
        |                             |
        v                             v
 +------+-------+             +-------+------+
 |  SECONDARY   |             |  SECONDARY   |
 +--------------+             +--------------+
```

* Le **Primary** est le seul à accepter les écritures.
* Les **Secondaries** répliquent les données via un flux **oplog**.
* En cas de panne du Primary → une **élection** est déclenchée automatiquement.

---

# Résumé des vidéos MongoDB 1 & 2

---

##  Vidéo MongoDB 1 — Mise en place d’un Replica Set

La vidéo montre :

1. Le lancement de plusieurs instances MongoDB sur des ports différents.
2. L’activation d’un Replica Set :

   ```
   rs.initiate()
   ```
3. L’ajout de membres :

   ```
   rs.add("localhost:27018")
   rs.add("localhost:27019")
   ```
4. Vérification de l’état du cluster :

   ```
   rs.status()
   ```

### ✔ Schéma du lancement des 3 nœuds

```
mongod --replSet rs0 --port 27017 --dbpath ./data1
mongod --replSet rs0 --port 27018 --dbpath ./data2
mongod --replSet rs0 --port 27019 --dbpath ./data3
```

---

## Vidéo MongoDB 2 — Gestion de panne et bascule automatique

La vidéo montre :

* l’arrêt manuel du Primary,
* l’élection d’un Secondary en Primary,
* la reprise automatique lorsqu’il revient.

### ✔ Schéma ASCII d’une bascule automatique

```
AVANT LA PANNE                 APRÈS LA PANNE
------------------            --------------------
PRIMARY (27017) X             PRIMARY (27018) ✓
SECONDARY (27018)             SECONDARY (27019)
SECONDARY (27019)             SECONDARY (27017) (retard)
```

Le cluster reste disponible car il y a **majorité**.

---

#  Partie 2 — Commandes & configuration



## **1. Qu’est-ce qu’un Replica Set ?**

Un ensemble de serveurs MongoDB répliquant automatiquement les mêmes données.

---

## **2. Rôle du Primary**

Il reçoit **toutes les écritures**, puis les envoie aux Secondaries.

---

## **3. Rôle des Secondaries**

* répliquer les données du Primary,
* servir des lectures,
* prendre le relais en cas de panne.

---

## **4. Pourquoi pas d'écriture sur Secondary ?**

Pour garantir la cohérence et éviter les conflits d’écriture.

---

## **5. Cohérence forte dans MongoDB**

Lecture sur le Primary → on voit **la version la plus récente et valide**.

---

## **6. Différence entre `readPreference: "primary"` et `"secondary"`**

| Mode      | Caractéristique                                    |
| --------- | -------------------------------------------------- |
| primary   | lecture cohérente garantie                         |
| secondary | charge distribuée mais risque de données obsolètes |

---

## **7. Pourquoi lire sur un Secondary ?**

* Décharger le Primary,
* Analytique, statistiques, reporting,
* Haute disponibilité en lecture.

---

## **8. Initialiser un Replica Set**

```js
rs.initiate()
```

---

## **9. Ajouter un nœud**

```js
rs.add("hostname:port")
```

---

## **10. Afficher l’état du Replica Set**

```js
rs.status()
```

---

## **11. Identifier le rôle d’un nœud**

```js
rs.isMaster()
```

ou en version récente :

```js
db.hello()
```

---

## **12. Forcer le basculement du Primary**

```js
rs.stepDown()
```

---

#  Partie 3 — Résilience & tolérance aux pannes

*(Réponses 13 à 22)*

---

## **13. Désigner un Arbiter**

```js
rs.addArb("hostname:27019")
```

Un arbiter **vote** mais ne stocke aucune donnée.

---

## **14. Ajouter un délai de réplication (slaveDelay)**

```js
cfg = rs.conf()
cfg.members[1].slaveDelay = 120
rs.reconfig(cfg)
```

---

## **15. Primary tombe sans majorité**

→ Aucun nouveau Primary n'est élu.
→ Cluster = **lecture seule**.

---

## **16. Comment MongoDB choisit un nouveau Primary ?**

* majorité disponible,
* fraîcheur des données,
* priorité,
* disponibilité.

---

## **17. Qu’est-ce qu’une élection ?**

Vote automatique entre les membres pour élire un Primary.

---

## **18. Auto-dégradation**

Un nœud devient Secondary lorsqu’il perd la majorité.

---

## **19. Pourquoi nombre impair de nœuds ?**

Pour faciliter l’obtention d’une majorité.

---

## **20. Partition réseau**

* Majorité → continue normalement
* Minorité → passe en mode Secondary

---

## **21. Primary devient injoignable dans un cluster à 3 nœuds (dont un arbiter)**

→ Une élection se produit.
→ Un Secondary devient Primary.

---

## **22. Importance d’un slaveDelay de 120s**

* permet de restaurer une mauvaise suppression,
* utile lors d’erreurs humaines.

---

#  Partie 4 — Scénarios pratiques

*(Réponses 23 à 32)*

---

## **23. Lecture toujours à jour après bascule**

Utiliser :

```js
readConcern: "majority"
writeConcern: { w: "majority" }
```

---

## **24. Exiger confirmation sur deux nœuds**

```js
writeConcern: { w: 2 }
```

---

## **25. Lecture obsolète sur Secondary — pourquoi ?**

Réplique asynchronously → retard possible.

Solutions :

* lire sur **Primary**
* ou utiliser `readConcern: "majority"`

---

## **26. Vérifier le Primary**

```js
rs.status()
```

---

## **27. Forcer une bascule manuelle**

```js
rs.stepDown(60)
```

---

## **28. Ajouter un Secondary**

```js
rs.add("host:port")
```

---

## **29. Retirer un nœud**

```js
rs.remove("host:port")
```

---

## **30. Rendre un Secondary caché**

```js
cfg = rs.conf()
cfg.members[1].hidden = true
cfg.members[1].priority = 0
rs.reconfig(cfg)
```

---

## **31. Modifier la priorité d’un nœud**

```js
cfg.members[1].priority = 5
rs.reconfig(cfg)
```

---

## **32. Vérifier le retard des Secondaries**

```js
rs.printSlaveReplicationInfo()
```

---

#  Partie 5 — Questions complémentaires

*(Réponses 33 à 46)*

---

## **33. Que fait `rs.freeze()` ?**

Empêche un Secondary de devenir Primary pendant un temps donné.

---

## **34. Redémarrer un Replica Set sans perdre la configuration**

La config est stockée dans les données → redémarrage sûr.

---

## **35. Surveiller la réplication**

* logs MongoDB
* commande :

  ```
  rs.printSlaveReplicationInfo()
  ```

---

## **37. Qu’est-ce qu’un Arbiter ?**

Nœud qui vote uniquement, ne stocke pas de données.

---

## **38. Vérifier la latence de réplication**

```
rs.printSlaveReplicationInfo()
```

---

## **39. Afficher le retard des membres**

Même commande.

---

## **40. Réplication synchrone vs asynchrone**

* **Synchrone** : confirmation par plusieurs nœuds avant validation
* **Asynchrone** : écriture validée sur Primary puis répliquée plus tard
  👉 MongoDB utilise **l'asynchrone**.

---

## **41. Modifier un Replica Set sans redémarrer ?**

Oui → `rs.reconfig()`.

---

## **42. Secondary en retard plusieurs minutes**

→ rattrapage des données
→ si trop en retard : resynchronisation complète.

---

## **43. Conflits de données**

Impossible car un seul nœud écrit.

---

## **44. Plusieurs Primary ?**

Non, protocole d’élection empêche cela.

---

## **45. Pourquoi ne pas écrire sur un Secondary ?**

Écritures risque de corruption, divergence, perte de données.

---

## **46. Réseau instable → conséquences**

* bascules répétées
* cluster instable
* perte de disponibilité

---

#  Conclusion

Ce TP montre comment MongoDB assure :

* la haute disponibilité,
* la redondance des données,
* la tolérance aux pannes,
* la continuité de service.

Les Replica Sets constituent la pierre angulaire de la résilience de MongoDB.
L’étude des bascules, priorités, retards et élections permet de comprendre en profondeur comment MongoDB garantit un fonctionnement fiable même en cas de défaillance d'un nœud.

---

