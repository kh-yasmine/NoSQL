# Rapport TP MongoDB

## Introduction
Ce rapport présente les réponses aux différentes questions du TP *Prise en main de MongoDB* ainsi que l’explication du rôle et du fonctionnement des requêtes associées.  
Les exemples de requêtes proviennent du sujet fourni.  

---

## Partie 1 – Filtrer et projeter les données

### 1. Afficher les 5 films sortis depuis 2015
```js
db.movies.find({ year: { $gte: 2015 } }).limit(5)
```
**Explication :**  
- `$gte` signifie *greater than or equal*, donc « supérieur ou égal ».  
- On filtre ici les films dont l’année est ≥ 2015.  
- `limit(5)` réduit le résultat aux 5 premiers documents trouvés.

---

### 2. Trouver tous les films dont le genre est "Comedy"
```js
db.movies.find({ genres: "Comedy" })
```
**Explication :**  
- Une recherche simple sur un champ tableau. MongoDB renvoie tous les films où le tableau `genres` contient l’élément "Comedy".

---

### 3. Afficher les films sortis entre 2000 et 2005
```js
db.movies.find({ year: { $gte: 2000, $lte: 2005 }}, { title: 1, year: 1 })
```
**Explication :**  
- `$gte` et `$lte` permettent de définir un intervalle.  
- La projection `{title:1, year:1}` limite les champs retournés.

---

### 4. Films de genres “Drama” ET “Romance”
```js
db.movies.find({ genres: { $all: ["Drama", "Romance"] }})
```
**Explication :**  
- `$all` impose que le tableau contienne **tous** les éléments listés.

---

### 5. Films sans champ `rated`
```js
db.movies.find({ rated: { $exists: false }})
```
**Explication :**  
- `$exists:false` sélectionne les documents où le champ n’existe pas.

---

## Partie 2 – Agrégation

### 6. Nombre de films par année
```js
db.movies.aggregate([
  { $group: { _id: "$year", total: { $sum: 1 }}},
  { $sort: { _id: 1 }}
])
```
**Explication :**  
- `$group` regroupe par année.  
- `$sum:1` compte les documents dans chaque groupe.  
- `$sort` trie les résultats.

---

### 7. Moyenne des notes IMDb par genre
```js
db.movies.aggregate([
  { $unwind: "$genres" },
  { $group: { _id: "$genres", moyenne: { $avg: "$imdb.rating" }}},
  { $sort: { moyenne: -1 }}
])
```
**Explication :**  
- `$unwind` déplie un tableau : un document par genre.  
- `$avg` calcule la moyenne.  
- Tri décroissant sur la moyenne.

---

### 8. Nombre de films par pays
```js
db.movies.aggregate([
  { $unwind: "$countries" },
  { $group: { _id: "$countries", total: { $sum: 1 }}},
  { $sort: { total: -1 }}
])
```
**Explication :**  
- Même logique que la question précédente mais appliquée aux pays.

---

## Partie 3 – Mises à jour

### 9. Top 5 réalisateurs
```js
db.movies.aggregate([
  { $unwind: "$directors" },
  { $group: { _id: "$directors", total: { $sum: 1 }}},
  { $sort: { total: -1 }},
  { $limit: 5 }
])
```
**Explication :**  
- Compte les occurrences de chaque réalisateur.  

---

### 10. Films triés par note IMDb
```js
db.movies.aggregate([
  { $sort: { "imdb.rating": -1 }},
  { $project: { title: 1, "imdb.rating": 1 }}
])
```
**Explication :**  
- `sort` classe les films du meilleur au moins bon selon IMDb.

---

### 11. Ajouter un champ `etat`
```js
db.movies.updateOne({ title: "Jaws" }, { $set: { etat: "culte" }})
```
**Explication :**  
- `$set` ajoute ou modifie un champ.

---

### 12. Incrémenter les votes IMDb
```js
db.movies.updateOne({ title: "Inception" }, { $inc: { "imdb.votes": 100 }})
```
**Explication :**  
- `$inc` incrémente la valeur numérique du champ.

---

### 13. Supprimer le champ `poster`
```js
db.movies.updateMany({}, { $unset: { poster: "" }})
```
**Explication :**  
- `$unset` supprime un champ.  

---

### 14. Modifier le réalisateur
```js
db.movies.updateOne({ title: "Titanic" }, { $set: { directors: ["James Cameron"] }})
```

---

## Partie 4 – Requêtes complexes

### 15. Meilleurs films par décennie
```js
db.movies.aggregate([
  { $match: { "imdb.rating": { $exists: true }}},
  { $project: { title: 1, decade: { $subtract: ["$year", { $mod: ["$year", 10 ]} ]}, "imdb.rating": 1 }},
  { $group: { _id: "$decade", maxRating: { $max: "$imdb.rating" }}},
  { $sort: { _id: 1 }}
])
```
**Explication :**  
- Calcul de la décennie avec `$mod`.  
- Sélection du film au meilleur score par décennie.

---

### 16. Films commençant par “Star”
```js
db.movies.find({ title: /^Star/ })
```
**Explication :**  
- Expression régulière (`regex`) pour filtrer les titres commençant par *Star*.

---

### 17. Films avec plus de 2 genres
```js
db.movies.find({ $where: "this.genres.length > 2" })
```
**Explication :**  
- `$where` exécute du JavaScript.  
- À éviter en production car peu performant.

---

### 18. Films de Christopher Nolan
```js
db.movies.find({ directors: "Christopher Nolan" })
```

---

## Partie 5 – Indexation

### 19. Créer un index sur year
```js
db.movies.createIndex({ year: 1 })
```
**Explication :**  
- Index croissant sur l’année : accélère les recherches par année.

---

### 20. Vérifier les index
```js
db.movies.getIndexes()
```

---

### 21. Comparer deux requêtes avec explain()
```js
db.movies.find({ year: 1995 }).explain("executionStats")
```
**Explication :**  
- Permet d’observer `totalDocsExamined` et `executionTimeMillis`.  
- Idéal pour comprendre l’impact des index.

---

### 22. Supprimer un index
```js
db.movies.dropIndex({ year: 1 })
```

---

### 23. Créer un index composé
```js
db.movies.createIndex({ year: 1, "imdb.rating": -1 })
```
**Explication :**  
- Utile pour des tri/recherches combinant année et note.

---

## Conclusion
Ce rapport détaille les requêtes MongoDB utilisées et leur objectif pédagogique pour comprendre les filtres, agrégations, mises à jour et index dans MongoDB.
