# Import de la structure sportive via modèle CSV

## Objectif
Permettre l’import en masse des **catégories**, **ligues** et **équipes** dans la saison active du club à partir d’un modèle CSV.

Ce guide documente le **comportement actuellement implémenté** dans l’écran de structure sportive.

## Où c’est utilisé
- Écran : `SportStructureScreen`
- Bouton : **Importer le modèle CSV**
- Prérequis : utilisateur avec permissions de gestion du club et mode édition activé.

## Fichiers de référence
- Modèle de base : `docs/templates/sport_structure_import_template.csv`
- Exemple valide : `docs/templates/sport_structure_import_example_valid_en.csv`
- Exemple invalide : `docs/templates/sport_structure_import_example_invalid_en.csv`

## Format CSV requis
En-têtes requis (ordre recommandé) :

```csv
entity,name,category,league
```

### Colonnes
- `entity` : type de ligne. Valeurs autorisées :
  - `category`
  - `league`
  - `team`
- `name` : nom principal de l’entité.
- `category` : obligatoire uniquement quand `entity=team`.
- `league` : obligatoire uniquement quand `entity=team`.

## Règles de validation
La validation est effectuée avant insertion :

1. Le fichier doit contenir l’en-tête + au moins une ligne de données.
2. Les colonnes `entity` et `name` doivent exister.
3. Si `entity=team`, `category` et `league` sont obligatoires.
4. Toute valeur `entity` en dehors de `category|league|team` est invalide.
5. Pour les équipes, la catégorie/la ligue référencée doit exister :
   - soit déjà en base dans la saison active,
   - soit créée dans le même CSV via des lignes `category`/`league`.

S’il y a des erreurs de validation, **l’import n’est pas exécuté** et un résumé d’erreurs est affiché.

## Comportement d’insertion
- Nouvelles catégories : insérées si absentes dans la saison active (comparaison normalisée du nom).
- Nouvelles ligues : même comportement.
- Nouvelles équipes : insérées si le triplet `name+category+league` n’existe pas.
- Les données existantes ne sont pas supprimées.
- Les noms existants ne sont pas mis à jour (comportement orienté ajout).

## Normalisation du texte
Avant comparaison/insertion :
- application de `trim`
- espaces multiples réduits à un seul

Exemple :
- `"  Senior   A  "` → `"Senior A"`

## Exemple valide
```csv
entity,name,category,league
category,U18,,
category,U16,,
league,Premier,,
league,Regional,,
team,U18 A,U18,Premier
team,U18 B,U18,Regional
team,U16 A,U16,Regional
```

Résultat attendu :
- Catégories créées : 2 (si absentes)
- Ligues créées : 2 (si absentes)
- Équipes créées : 3 (si absentes)

## Exemple invalide
```csv
entity,name,category,league
team,Équipe sans ligue,U18,
foo,Type de ligne inconnu,,
team,,U18,Premier
team,Équipe avec référence manquante,U14,Premier
```

Erreurs attendues :
- ligne `team` sans `league` ;
- `entity` invalide (`foo`) ;
- `name` vide ;
- référence à une catégorie inexistante (`U14`).

## Flux opérationnel recommandé
1. Télécharger le modèle de base.
2. Compléter d’abord catégories et ligues.
3. Ajouter les équipes en référant exactement les noms de catégorie/ligue.
4. Importer en environnement de test.
5. Vérifier le résumé de création.
6. Répéter en production.

## Bonnes pratiques
- Garder des noms cohérents (ex. `U18 A`, `U18 B`).
- Éviter les variantes typographiques pour la même entité.
- Importer par blocs (structure de base, puis extensions incrémentales).

## Portée actuelle
Cet import couvre actuellement :
- catégories
- ligues
- équipes

Pas encore couvert :
- affectation massive de membres
- import massif de `player_profile`
- mise à jour/suppression en masse

## Dépannage
### "CSV requires columns: entity,name,category,league"
L’en-tête est incorrect. Vérifier la première ligne.

### "Template has no data"
Le fichier contient uniquement l’en-tête ou est vide.

### "team requires category and league"
La ligne d’équipe est incomplète.

### "category/league does not exist"
Ajouter des lignes de création dans le même CSV ou créer ces enregistrements dans l’app avant l’import.
