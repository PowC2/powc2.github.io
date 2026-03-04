# Importação da estrutura esportiva por modelo CSV

## Objetivo
Permitir o carregamento em massa de **categorias**, **ligas** e **equipes** na temporada ativa do clube usando um modelo CSV.

Este guia documenta o **comportamento atualmente implementado** na tela de estrutura esportiva.

## Onde é usado
- Tela: `SportStructureScreen`
- Botão: **Importar modelo CSV**
- Requisito: usuário com permissões de gestão do clube e modo de edição habilitado.

## Arquivos de referência
- Modelo base: `docs/templates/sport_structure_import_template.csv`
- Exemplo válido: `docs/templates/sport_structure_import_example_valid_en.csv`
- Exemplo inválido: `docs/templates/sport_structure_import_example_invalid_en.csv`

## Formato CSV obrigatório
Cabeçalhos obrigatórios (ordem recomendada):

```csv
entity,name,category,league
```

### Colunas
- `entity`: tipo de linha. Valores permitidos:
  - `category`
  - `league`
  - `team`
- `name`: nome principal da entidade.
- `category`: obrigatório apenas quando `entity=team`.
- `league`: obrigatório apenas quando `entity=team`.

## Regras de validação
A validação ocorre antes da inserção:

1. O arquivo deve conter cabeçalho + pelo menos uma linha de dados.
2. As colunas `entity` e `name` devem existir.
3. Se `entity=team`, `category` e `league` são obrigatórios.
4. Qualquer `entity` fora de `category|league|team` é inválido.
5. Para equipes, categoria/liga referenciada deve existir:
   - já no banco da temporada ativa, ou
   - criada no mesmo CSV por linhas `category`/`league`.

Se houver erros de validação, **a importação não é executada** e um resumo de erros é exibido.

## Comportamento de inserção
- Novas categorias: inseridas se não existirem na temporada ativa (comparação por nome normalizado).
- Novas ligas: mesmo comportamento.
- Novas equipes: inseridas se a tupla `name+category+league` não existir.
- Dados existentes não são removidos.
- Nomes existentes não são atualizados (comportamento orientado a append).

## Normalização de texto
Antes de comparar/inserir:
- aplica-se `trim`
- espaços múltiplos são reduzidos a um

Exemplo:
- `"  Senior   A  "` → `"Senior A"`

## Exemplo válido
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

Resultado esperado:
- Categorias criadas: 2 (se faltantes)
- Ligas criadas: 2 (se faltantes)
- Equipes criadas: 3 (se faltantes)

## Exemplo inválido
```csv
entity,name,category,league
team,Equipe sem liga,U18,
foo,Tipo de linha desconhecido,,
team,,U18,Premier
team,Equipe com referência ausente,U14,Premier
```

Erros esperados:
- linha `team` sem `league`;
- `entity` inválido (`foo`);
- `name` vazio;
- referência para categoria inexistente (`U14`).

## Fluxo operacional recomendado
1. Baixar o modelo base.
2. Preencher primeiro categorias e ligas.
3. Adicionar equipes referenciando nomes exatos de categoria/liga.
4. Importar em ambiente de homologação.
5. Revisar o resumo de criação.
6. Repetir em produção.

## Boas práticas
- Manter nomenclatura consistente (ex.: `U18 A`, `U18 B`).
- Evitar variações tipográficas para a mesma entidade.
- Importar em blocos (estrutura base primeiro, depois incrementos).

## Escopo atual
Esta importação cobre atualmente:
- categorias
- ligas
- equipes

Ainda não cobre:
- atribuição em massa de membros
- carga em massa de `player_profile`
- atualização/remoção em massa

## Solução de problemas
### "CSV requires columns: entity,name,category,league"
O cabeçalho está incorreto. Verifique a primeira linha.

### "Template has no data"
O arquivo contém apenas o cabeçalho ou está vazio.

### "team requires category and league"
A linha de equipe está incompleta.

### "category/league does not exist"
Adicione linhas de criação no mesmo CSV ou crie esses registros no app antes da importação.
