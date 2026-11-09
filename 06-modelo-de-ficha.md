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

Regras gerais:

- Todas as seções, menos `versao`, são opcionais.
- `id`: letras minúsculas, números e `_`, começando com letra, até 32 caracteres, sem parecer um dado (`d6`). É **único no modelo inteiro**, para as fórmulas não ficarem ambíguas.
- `rotulo`: de 1 a 60 caracteres. Cada lista tem no máximo 100 itens.

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

- **Completo:** nome, classe, raça, nível, experiência, antecedente, alinhamento, classe de armadura, deslocamento e dados de vida; os 6 atributos (1 a 30, padrão 10); iniciativa, 6 salvaguardas e 18 perícias, cada uma com `1d20 + mod(atributo) + treinada(id) * (2 + piso((nivel - 1) / 4))`; PV e PV temporários; 15 condições; tabela de ataques (arma, rolagem de ataque, dano, tipo de dano, alcance).
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
  "combate": [ { "arma": "Cajado", "dano": "1d6" } ]
}
```

## Edição simultânea

Vale a última gravação de cada campo. O cliente envia só os campos alterados, agrupados a cada ~400 ms (`PATCH /mesas/{id}/fichas/{fichaId}` com `{ "campos": { "atributos.forca": 9 } }`), e o servidor emite `FichaAtualizada` pelo SignalR para quem pode ver a ficha. Quem recebe aplica os campos, menos o que estiver digitando.

**Implementado na 4.4:**

- O servidor valida cada caminho contra o modelo (`Fichas/ValidacaoCamposFicha.cs`): `detalhes.{id}`, `atributos.{id}`, `pericias.{id}.treinada|favorita|valor`, `barras.{id}.atual|maximo`, `condicoes` (lista inteira) e `combate` (tabela inteira, até 50 linhas). No máximo 50 campos por envio; texto até 200 caracteres.
- Cada campo é gravado com `jsonb_set` direto no banco, dentro de uma transação. Duas pessoas mexendo em campos diferentes ao mesmo tempo não apagam a alteração uma da outra.
- No frontend, o campo editado localmente fica protegido por 1,5 s contra valores que chegam pelo tempo real.
- A ficha mostra o resultado das fórmulas sem rolar (`src/dominio/formula.ts`): dados somados e constante calculada, como `1d20 + 5`. Multiplicar ou dividir dados não é suportado na exibição; nesse caso aparece "—". A rolagem de verdade entra com o chat.
