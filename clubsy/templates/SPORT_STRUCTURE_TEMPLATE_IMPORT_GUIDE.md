# Carga de estructura deportiva por plantilla (CSV)

> English version: `docs/SPORT_STRUCTURE_TEMPLATE_IMPORT_GUIDE_EN.md`

## Objetivo
Permitir la carga masiva de **categorías**, **ligas** y **equipos** en la temporada activa del club desde una plantilla CSV.

Esta guía documenta el comportamiento **real implementado** en la pantalla de estructura deportiva.

## Dónde se usa
- Pantalla: `SportStructureScreen`
- Botón: **Importar plantilla CSV**
- Requisito: usuario con permisos de gestión del club y pantalla en modo edición.

## Archivos de referencia
- Plantilla base: `docs/templates/sport_structure_import_template.csv`
- Ejemplo válido: `docs/templates/sport_structure_import_example_valid.csv`
- Ejemplo con errores: `docs/templates/sport_structure_import_example_invalid.csv`

## Formato obligatorio del CSV
Encabezados requeridos (en este orden recomendado):

```csv
entity,name,category,league
```

### Columnas
- `entity`: tipo de fila. Valores permitidos:
  - `category`
  - `league`
  - `team`
- `name`: nombre principal de la entidad.
- `category`: solo obligatorio cuando `entity=team`.
- `league`: solo obligatorio cuando `entity=team`.

## Reglas de validación
La importación valida antes de insertar:

1. El archivo debe tener al menos encabezado + 1 fila de datos.
2. Deben existir las columnas `entity` y `name`.
3. Si `entity=team`, `category` y `league` son obligatorios.
4. `entity` fuera de `category|league|team` se considera error.
5. Para equipos, la categoría/liga deben existir:
   - en BD de la temporada activa, o
   - en el mismo archivo (si se crean en filas `category`/`league`).

Si hay errores de validación, **no se ejecuta la importación** y se muestra un resumen de errores.

## Comportamiento de inserción
- Categorías nuevas: se insertan si no existen en la temporada activa (comparación normalizada por nombre).
- Ligas nuevas: mismo criterio.
- Equipos nuevos: se insertan si no existe combinación equivalente de nombre+categoría+liga.
- No se eliminan registros existentes.
- No actualiza nombres existentes (modo append/upsert parcial orientado a altas).

## Normalización de texto
Antes de comparar/insertar:
- se aplica `trim`;
- espacios múltiples se colapsan a uno.

Ejemplo:
- `"  Senior   A  "` → `"Senior A"`

## Ejemplo válido
```csv
entity,name,category,league
category,Senior,,
category,Juvenil,,
league,Oro,,
league,Plata,,
team,Equipo Senior A,Senior,Oro
team,Equipo Senior B,Senior,Plata
team,Equipo Juvenil A,Juvenil,Plata
```

Resultado esperado:
- Categorías creadas: 2 (si no existían)
- Ligas creadas: 2 (si no existían)
- Equipos creados: 3 (si no existían)

## Ejemplo inválido
```csv
entity,name,category,league
team,Equipo sin liga,Senior,
foo,Registro desconocido,,
team,,Senior,Oro
team,Equipo con referencia faltante,Sub13,Oro
```

Errores esperados:
- fila con `team` sin `league`;
- `entity` inválido (`foo`);
- `name` vacío;
- referencia a categoría inexistente (`Sub13`).

## Flujo recomendado de operación
1. Descargar plantilla base.
2. Completar categorías y ligas primero.
3. Completar equipos referenciando nombres exactos de categoría/liga.
4. Importar en entorno de prueba.
5. Revisar resumen de creación.
6. Repetir en producción.

## Buenas prácticas
- Mantener nombres consistentes (ej. `Juvenil A`, `Juvenil B`).
- Evitar variantes tipográficas para la misma entidad.
- Cargar por bloques (primero estructura simple, luego ampliaciones).

## Alcance actual
Esta importación cubre:
- categorías
- ligas
- equipos

No cubre todavía:
- asignación masiva de miembros
- carga de `player_profile`
- actualización/borrado masivo

## Troubleshooting
### "CSV requiere columnas: entity,name,category,league"
El encabezado no coincide. Verificar primera fila.

### "Plantilla sin datos"
El archivo solo tiene encabezado o está vacío.

### "team requiere category y league"
Fila de equipo incompleta.

### "categoría/liga no existe"
Agregar fila de creación en el mismo CSV o crearla antes en la app.
