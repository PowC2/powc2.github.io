# Importació de l'estructura esportiva amb plantilla CSV

## Objectiu
Permetre la càrrega massiva de **categories**, **lligues** i **equips** a la temporada activa del club mitjançant una plantilla CSV.

Aquesta guia documenta el **comportament implementat actualment** a la pantalla d’estructura esportiva.

## On s’utilitza
- Pantalla: `SportStructureScreen`
- Botó: **Importar plantilla CSV**
- Requisit: usuari amb permisos de gestió del club i mode d’edició actiu.

## Fitxers de referència
- Plantilla base: `docs/templates/sport_structure_import_template.csv`
- Exemple vàlid: `docs/templates/sport_structure_import_example_valid.csv`
- Exemple amb errors: `docs/templates/sport_structure_import_example_invalid.csv`

## Format CSV obligatori
Capçaleres requerides (ordre recomanat):

```csv
entity,name,category,league
```

### Columnes
- `entity`: tipus de fila. Valors permesos:
  - `category`
  - `league`
  - `team`
- `name`: nom principal de l’entitat.
- `category`: obligatori només quan `entity=team`.
- `league`: obligatori només quan `entity=team`.

## Regles de validació
La validació s’executa abans d’inserir:

1. El fitxer ha de tenir capçalera + almenys una fila de dades.
2. Han d’existir les columnes `entity` i `name`.
3. Si `entity=team`, `category` i `league` són obligatoris.
4. Qualsevol `entity` fora de `category|league|team` és invàlid.
5. Per a equips, la categoria/lliga referenciada ha d’existir:
   - o bé ja a la BD de la temporada activa,
   - o bé creada al mateix CSV amb files `category`/`league`.

Si hi ha errors de validació, **la importació no s’executa** i es mostra un resum d’errors.

## Comportament d’inserció
- Categories noves: s’insereixen si no existeixen a la temporada activa (comparació de nom normalitzat).
- Lligues noves: mateix criteri.
- Equips nous: s’insereixen si no existeix la tupla `name+category+league`.
- No s’eliminen registres existents.
- No s’actualitzen noms existents (comportament orientat a afegir).

## Normalització de text
Abans de comparar/inserir:
- s’aplica `trim`
- els espais múltiples es redueixen a un

Exemple:
- `"  Senior   A  "` → `"Senior A"`

## Exemple vàlid
```csv
entity,name,category,league
category,Senior,,
category,Juvenil,,
league,Or,,
league,Plata,,
team,Equip Senior A,Senior,Or
team,Equip Senior B,Senior,Plata
team,Equip Juvenil A,Juvenil,Plata
```

Resultat esperat:
- Categories creades: 2 (si no existien)
- Lligues creades: 2 (si no existien)
- Equips creats: 3 (si no existien)

## Exemple invàlid
```csv
entity,name,category,league
team,Equip sense lliga,Senior,
foo,Registre desconegut,,
team,,Senior,Or
team,Equip amb referència absent,Sub13,Or
```

Errors esperats:
- fila `team` sense `league`;
- `entity` invàlid (`foo`);
- `name` buit;
- referència a categoria inexistent (`Sub13`).

## Flux operatiu recomanat
1. Descarregar la plantilla base.
2. Omplir primer categories i lligues.
3. Afegir equips referenciant noms exactes de categoria/lliga.
4. Importar en entorn de proves.
5. Revisar el resum de creació.
6. Repetir en producció.

## Bones pràctiques
- Mantenir noms coherents (p. ex. `Juvenil A`, `Juvenil B`).
- Evitar variants tipogràfiques per a la mateixa entitat.
- Importar per blocs (primer estructura base, després ampliacions).

## Abast actual
Aquesta importació cobreix actualment:
- categories
- lligues
- equips

Encara no cobreix:
- assignació massiva de membres
- càrrega massiva de `player_profile`
- actualització/esborrat massiu

## Resolució de problemes
### "CSV requires columns: entity,name,category,league"
La capçalera no és correcta. Verificar la primera fila.

### "Template has no data"
El fitxer només té capçalera o és buit.

### "team requires category and league"
La fila d’equip és incompleta.

### "category/league does not exist"
Afegir files de creació al mateix CSV o crear aquests registres a l’app abans d’importar.
