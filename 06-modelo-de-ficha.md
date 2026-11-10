# Modelo de ficha

Formato do `ModeloFicha` (e do `ModeloFichaSimplificada`, que usa o mesmo formato) guardado em `jsonb` no `Template` e na `Mesa`. O backend valida o modelo em `ModelosFicha/ValidacaoModeloFicha.cs`; os testes ficam em `tests/ProjetoRpg.Testes/ModelosFicha/`.

## Estrutura

```json
{
  "versao": 1,
  "detalhes": [
    { "id": "nome", "rotulo": "Nome", "tipo": "texto" },
    { "id": "classe", "rotulo": "Classe", "tipo": "lista", "opcoes": ["Guerreiro", "Mago"] },
    { "id": "nivel", "rotulo": "Nível", "tipo": "numero", "minimo": 1, "maximo": 20 }
  ],
  "atributos": [
    { "id": "forca", "rotulo": "Força", "minimo": 1, "maximo": 30, "padrao": 10, "rolagem": "1d20 + mod(forca)" }
  ],
  "pericias": [
    { "id": "atletismo", "rotulo": "Atletismo", "atributo": "forca", "rolagem": "1d20 + mod(forca)" }
  ],
  "barras": [ { "id": "pv", "rotulo": "Pontos de vida", "cor": "vida", "maximoPadrao": 10 } ],
  "condicoes": [ { "id": "envenenado", "rotulo": "Envenenado" } ],
  "combate": { "colunas": [ { "id": "arma", "rotulo": "Arma", "tipo": "texto" }, { "id": "dano", "rotulo": "Dano", "tipo": "rolagem" } ] }
}
```

| Seção | Campos | Regras |
| --- | --- | --- |
| `versao` | número | Obrigatório; hoje só `1` |
| `detalhes` | `id`, `rotulo`, `tipo` (`texto`, `numero`, `lista`), `opcoes?`, `minimo?`, `maximo?` | Lista: de 1 a 50 opções sem repetir; número: mínimo ≤ máximo |
| `atributos` | `id`, `rotulo`, `minimo`, `maximo`, `padrao`, `rolagem?` | mínimo ≤ padrão ≤ máximo |
| `pericias` | `id`, `rotulo`, `atributo?`, `rolagem?` | `atributo` precisa ser o id de um atributo |
| `barras` | `id`, `rotulo`, `cor`, `maximoPadrao` | Cor da paleta: `vida`, `sanidade`, `esforco`, `arcano`, `essencia`, `mare` |
| `condicoes` | `id`, `rotulo` | Marcadas ou não na ficha |
| `combate.colunas` | `id`, `rotulo`, `tipo` (`texto`, `numero`, `rolagem`) | Colunas da tabela de ataques |
| `secoesExtras` | `id`, `rotulo`, `formato` e a lista do formato | Opcional, até 20. Formatos: `campos` (lista `campos`, como os detalhes), `valores` (lista `valores`, como os atributos, com fórmula), `barras`, `marcadores` (lista `marcadores`), `tabela` (lista `colunas`) e `anotacoes` (sem itens). Só a lista do próprio formato pode ter itens. O `id` da seção não pode ser de uma seção padrão |
| `estilo` | `tema`, `cor`, `fonteTitulo`, `fonteTexto`, `cantos`, `borda`, `titulo`, `densidade`, `secoes` | Opcional, tudo opcional. `tema`: padrao, pergaminho, grimorio, floresta, brasa, gelo. `cor`: `#rrggbb` (destaque). Fontes: medieval, classica, moderna, maquina. `cantos`: retos, suaves, redondos. `borda`: fina, grossa, destaque, nenhuma. `titulo`: simples, linha, faixa. `densidade`: compacta, normal, espacosa. `secoes`: `{ <id da seção>: { cor?, borda?, titulo? } }`, até 40, só seções que existem |
| `layout` | `linhas: [{ divisao, colunas: [[secao, ...], ...] }]` | Opcional. Até 12 linhas. `divisao`: `uma`, `duas`, `larga-estreita`, `estreita-larga` (2 colunas) ou `tres`; o número de colunas precisa bater. Seções: `detalhes`, `barras`, `atributos`, `condicoes`, `pericias`, `combate` e os ids das `secoesExtras`, cada uma no máximo uma vez |

Regras gerais:

- Todas as seções, menos `versao`, são opcionais.
- `id`: letras minúsculas, números e `_`, começando com letra, até 32 caracteres, sem parecer um dado (`d6`). É **único no modelo inteiro**, para as fórmulas não ficarem ambíguas.
- `rotulo`: de 1 a 60 caracteres. Cada lista tem no máximo 100 itens.

### Layout da ficha

Diz onde cada seção aparece. Sem `layout`, vale o padrão: Detalhes numa linha; Status, Atributos e Condições numa coluna larga e Perícias numa estreita; Ataques embaixo. Seções que não estão no layout aparecem no fim, uma embaixo da outra. Na tela, as colunas só ficam lado a lado em telas largas; numa linha em que só uma coluna tem conteúdo, ela ocupa a largura toda.

```json
"layout": { "linhas": [
  { "divisao": "duas", "colunas": [["detalhes"], ["pericias"]] },
  { "divisao": "larga-estreita", "colunas": [["barras", "atributos"], ["condicoes"]] },
  { "divisao": "uma", "colunas": [["combate"]] }
] }
```

## Fórmulas de rolagem

Escritas no campo `rolagem` de atributos e perícias (e nas colunas `rolagem` do combate, na ficha).

| Elemento | Exemplo |
| --- | --- |
| Números e operações | `2 + 3 * 4`, `-(1d4)`, parênteses |
| Dados | `1d20`, `d6`, `2d8` |
| Campo do modelo (pelo `id`) | `nivel`, `forca` |
| `mod(x)` | Modificador do atributo, como no 5e: `piso((x - 10) / 2)` |
| `piso(x)`, `teto(x)` | Arredonda para baixo / para cima |
| `max(a, b)`, `min(a, b)` | Maior / menor valor |
| `dados(qtd, lados)` | Quantidade de dados vinda de um campo: `dados(forca, 20)` |
| `treinada(id)` | 1 se a perícia `id` estiver marcada como treinada na ficha, 0 se não: `treinada(furtividade) * 2` |

O analisador (`ModelosFicha/AnalisadorFormula.cs`) só confere a escrita e se os campos existem. O cálculo da rolagem entra com o chat (fase 5).

## Template 5e

Modelos em `Projeto_RPG_Backend/src/ProjetoRpg.Api/Templates/Modelos/` (`5e.json` e `5e-simplificado.json`), incorporados à API e usados no seed do template. Só conteúdo do SRD 5.1 (CC-BY-4.0), com a atribuição na descrição do template e na página `/creditos` do site.

- **Completo** (todos os campos da primeira página da ficha de papel do 5e, só com termos do SRD): nome do personagem, classe, nível, antecedente, nome do jogador, raça, alinhamento, experiência e dado de vida; os 6 atributos (1 a 30, padrão 10); iniciativa, 6 salvaguardas, 18 perícias e a Sabedoria passiva (Percepção), com `1d20 + mod(atributo) + treinada(id) * bonus_proficiencia` (a passiva é `10 + mod(sabedoria) + treinada(percepcao) * bonus_proficiencia`); PV, PV temporários e dados de vida restantes; 15 condições; tabela de ataques. Seções novas: Combate (classe de armadura, deslocamento e bônus de proficiência, de 2 a 6, escrito pelo jogador como na ficha de papel), Testes contra a morte (3 sucessos e 3 falhas), Traços de personalidade, Ideais, Vínculos, Defeitos, Outras proficiências e idiomas, Moedas (PC, PP, PE, PO, PL), Equipamento e Características e talentos. Layout em 4 linhas: Detalhes; 3 colunas (atributos e combate · status, testes contra a morte e condições · traços, ideais, vínculos e defeitos); perícias ao lado de ataques e características; proficiências ao lado de moedas e equipamento.
- **Simplificado (pets e NPCs):** nome, tipo, nível, CA e deslocamento; os 6 atributos; iniciativa, percepção e furtividade; PV; as mesmas condições e a mesma tabela de ataques.

## Dados da ficha

Valores de uma ficha (`Ficha.Dados`), no formato do modelo da mesa:

```json
{
  "detalhes": { "nome": "Lyra", "classe": "Mago", "nivel": 3 },
  "atributos": { "forca": 8 },
  "pericias": { "atletismo": { "valor": 0, "treinada": false, "favorita": true } },
  "barras": { "pv": { "atual": 14, "maximo": 18 } },
  "condicoes": ["envenenado"],
  "combate": [ { "arma": "Cajado", "dano": "1d6" } ],
  "extras": { "moedas": { "ouro": 25 }, "idiomas": ["comum"], "inventario": [ { "item": "Corda", "quantidade": 2 } ], "historia": "..." }
}
```

Valores das seções novas ficam em `extras.<id da seção>`: objeto `{ item: valor }` para campos e valores, `{ item: { atual, maximo } }` para barras, lista de ids para marcadores, lista de linhas para tabela e texto (até 5.000 caracteres) para anotações. No `PATCH`, os caminhos são `extras.<secao>.<item>`, `extras.<secao>.<item>.atual|maximo` e `extras.<secao>` (marcadores, tabela e anotações inteiros). Itens de `valores`, `barras` e campos numéricos das seções novas podem ser usados nas fórmulas.

## Edição simultânea

Vale a última gravação de cada campo. O cliente envia só os campos alterados, agrupados a cada ~400 ms (`PATCH /mesas/{id}/fichas/{fichaId}` com `{ "campos": { "atributos.forca": 9 } }`), e o servidor emite `FichaAtualizada` pelo SignalR para quem pode ver a ficha. Quem recebe aplica os campos, menos o que estiver digitando.

**Implementado na 4.4:**

- O servidor valida cada caminho contra o modelo (`Fichas/ValidacaoCamposFicha.cs`): `detalhes.{id}`, `atributos.{id}`, `pericias.{id}.treinada|favorita|valor`, `barras.{id}.atual|maximo`, `condicoes` (lista inteira) e `combate` (tabela inteira, até 50 linhas). No máximo 50 campos por envio; texto até 200 caracteres.
- Cada campo é gravado com `jsonb_set` direto no banco, dentro de uma transação. Duas pessoas mexendo em campos diferentes ao mesmo tempo não apagam a alteração uma da outra.
- No frontend, o campo editado localmente fica protegido por 1,5 s contra valores que chegam pelo tempo real.
- A ficha mostra o resultado das fórmulas sem rolar (`src/dominio/formula.ts`): dados somados e constante calculada, como `1d20 + 5`. Multiplicar ou dividir dados não é suportado na exibição; nesse caso aparece "—". A rolagem de verdade entra com o chat.
