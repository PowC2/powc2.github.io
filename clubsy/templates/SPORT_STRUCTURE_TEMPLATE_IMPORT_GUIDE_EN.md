# Sport Structure Import via CSV Template

## Goal
Enable bulk upload of **categories**, **leagues**, and **teams** into the club's active season using a CSV template.

This guide documents the **current implemented behavior** in the sport structure screen.

## Where it is used
- Screen: `SportStructureScreen`
- Button: **Import CSV template**
- Requirement: user with club management permissions and edit mode enabled.

## Reference files
- Base template: `docs/templates/sport_structure_import_template.csv`
- Valid example: `docs/templates/sport_structure_import_example_valid_en.csv`
- Invalid example: `docs/templates/sport_structure_import_example_invalid_en.csv`

## Required CSV format
Required headers (recommended order):

```csv
entity,name,category,league
```

### Columns
- `entity`: row type. Allowed values:
  - `category`
  - `league`
  - `team`
- `name`: main entity name.
- `category`: required only when `entity=team`.
- `league`: required only when `entity=team`.

## Validation rules
Validation runs before inserts:

1. File must include header + at least one data row.
2. `entity` and `name` columns must exist.
3. If `entity=team`, `category` and `league` are required.
4. Any `entity` outside `category|league|team` is invalid.
5. For teams, referenced category/league must exist:
   - either already in active season DB, or
   - created in the same CSV through `category`/`league` rows.

If there are validation errors, **import does not run** and an error summary is shown.

## Insert behavior
- New categories: inserted if not present in active season (normalized name comparison).
- New leagues: same behavior.
- New teams: inserted if equivalent tuple `name+category+league` does not exist.
- Existing data is not deleted.
- Existing names are not updated (append-oriented behavior).

## Text normalization
Before comparison/insert:
- `trim` is applied
- multiple spaces are collapsed into one

Example:
- `"  Senior   A  "` → `"Senior A"`

## Valid example
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

Expected result:
- Categories created: 2 (if missing)
- Leagues created: 2 (if missing)
- Teams created: 3 (if missing)

## Invalid example
```csv
entity,name,category,league
team,Team without league,U18,
foo,Unknown row type,,
team,,U18,Premier
team,Team with missing reference,U14,Premier
```

Expected errors:
- team row missing `league`;
- invalid `entity` (`foo`);
- empty `name`;
- reference to non-existing category (`U14`).

## Recommended operational flow
1. Download the base template.
2. Fill categories and leagues first.
3. Add teams referencing exact category/league names.
4. Import in staging.
5. Review creation summary.
6. Repeat in production.

## Best practices
- Keep naming consistent (e.g. `U18 A`, `U18 B`).
- Avoid typographic variants for the same entity.
- Import in blocks (base structure first, then incremental additions).

## Current scope
This import currently supports:
- categories
- leagues
- teams

Not included yet:
- bulk member assignment
- `player_profile` bulk load
- mass update/delete

## Troubleshooting
### "CSV requires columns: entity,name,category,league"
Header is incorrect. Verify first row.

### "Template has no data"
File contains only header or is empty.

### "team requires category and league"
Team row is incomplete.

### "category/league does not exist"
Add creation rows in the same CSV or create those records in the app first.
