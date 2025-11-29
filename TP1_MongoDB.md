# Rapport TP MongoDB – Base *lesfilms*

## Introduction

Ce rapport présente les réponses aux différentes questions du TP portant sur la base NoSQL *lesfilms*, ainsi qu’une explication du rôle des requêtes MongoDB utilisées.  
Chaque question contient la requête, le résultat du terminal (à remplir ou déjà fourni), et une explication détaillée.

---

# 1. Vérifier que les données ont été importées

## Requête
```js
db.films.find().count()
```

## Résultat du terminal
```
278
```

## Explication
Cette commande compte le nombre total de documents présents dans la collection `films`.  
Elle confirme que l’importation depuis `films.json` a été effectuée avec succès.

---

# 2. Comprendre la structure d’un document avec `findOne()`

## Requête
```js
db.films.findOne()
```

## Résultat du terminal
```json
{
  "_id": "movie:28",
  "title": "Apocalypse Now",
  "year": 1979,
  "genre": "Drame",
  "summary": "L'état-major américain confie au jeune capitaine Willard...",
  "country": "US",
  "director": {},
  "actors": [],
  "grades": []
}
```

## Explication
`findOne()` renvoie un seul document de la collection.  
Cette commande est essentielle pour comprendre la structure interne des données : objets imbriqués, tableaux, champs simples…  
Elle permet de construire des requêtes adaptées aux données.

---

# 3. Afficher la liste des films d’action

## Requête
```js
db.films.find({ genre: "Action" })
```
## Explication
Filtre tous les documents dont le champ `genre` vaut `"Action"`.

---

# 4. Afficher le nombre de films d’action

## Requête
```js
db.films.countDocuments({ genre: "Action" })
```
## Explication
Compte le nombre de films correspondant au genre Action.

---

# 5. Afficher les films d’action produits en France

## Requête
```js
db.films.find({ genre: "Action", country: "France" })
```

## Explication
Sélectionne uniquement les films qui sont du genre Action **et** produits en France.

---

# 6. Films d’action produits en France en 1963

## Requête
```js
db.films.find({
  genre: "Action",
  country: "France",
  year: 1963
})
```

## Explication
Ajoute un critère supplémentaire (`year: 1963`) au filtre précédent.

---

# 7. Projection : films d’action FR sans les grades

## Requête
```js
db.films.find(
  { genre: "Action", country: "France" },
  { grades: 0 }
)
```

## Explication
La projection `{ grades: 0 }` masque le champ `grades` dans les résultats.

---

# 8. Masquer `_id` dans les résultats

## Requête
```js
db.films.find(
  { genre: "Action", country: "France" },
  { grades: 0, _id: 0 }
)
```

## Explication
Permet d’afficher uniquement les champs pertinents en supprimant également le champ `_id`.

---

# 9. Titres + grades des films d’action FR, sans `_id`

## Requête
```js
db.films.find(
  { genre: "Action", country: "France" },
  { title: 1, grades: 1, _id: 0 }
)
```

## Explication
Projection positive : seuls les champs listés en `1` seront affichés.

---

# 10. Films d’action FR avec au moins une note > 10

## Requête
```js
db.films.find(
  { genre: "Action", country: "France", grades: { $gt: 10 } },
  { title: 1, grades: 1, _id: 0 }
)
```

## Explication
`$gt: 10` teste si au moins une des notes du tableau `grades` est supérieure à 10.

---

# 11. Films dont *toutes* les notes sont > 10

## Requête
```js
db.films.find(
  {
    genre: "Action",
    country: "France",
    grades: { $not: { $lte: 10 } }
  },
  { title: 1, grades: 1, _id: 0 }
)
```

## Explication
`$lte: 10` repère les notes ≤ 10, `"$not"` exclut les films contenant au moins une note ≤ 10.

---

# 12. Afficher les différents genres

## Requête
```js
db.films.distinct("genre")
```

## Explication
Renvoie une liste des genres uniques dans la collection.

---

# 13. Afficher les différentes valeurs de grades

## Requête
```js
db.films.distinct("grades.grade")
```

## Explication
Extraction de toutes les valeurs uniques du champ `grade` dans le tableau `grades`.

---

# 14. Films avec au moins un artiste parmi 3 IDs

## Requête
```js
db.films.find({
  "actors._id": { $in: ["artist:4", "artist:18", "artist:11"] }
})
```

## Explication
`$in` vérifie que **au moins un acteur** correspond à l’un des identifiants.

---

# 15. Films sans résumé

## Requête
```js
db.films.find({ summary: { $exists: false } })
```

## Explication
`$exists: false` filtre les documents dépourvus du champ `summary`.

---

# 16. Films avec Leonardo DiCaprio en 1997

## Requête
```js
db.films.find({
  actors: { $elemMatch: { last_name: "DiCaprio" } },
  year: 1997
})
```

## Explication
`$elemMatch` permet de rechercher dans un tableau.  
Ici : un acteur dont le nom de famille est `"DiCaprio"` + année = 1997.

---

# 17. Films avec Leonardo DiCaprio **ou** en 1997

## Requête
```js
db.films.find({
  $or: [
    { actors: { $elemMatch: { last_name: "DiCaprio" } } },
    { year: 1997 }
  ]
})
```

## Explication
Retourne les films qui satisfont au moins une des deux conditions.

---

# Conclusion

Ce TP a permis de manipuler les fonctionnalités essentielles de MongoDB :  
requêtes simples, filtres, projections, opérateurs logiques, gestion des tableaux et requêtes complexes.  
La flexibilité de MongoDB permet de représenter et d’interroger facilement des données semi‑structurées.
