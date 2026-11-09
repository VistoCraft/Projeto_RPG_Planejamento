# Identidade visual

Base visual do site. Referência: [referencias/identidade-visual.jpg](referencias/identidade-visual.jpg) (imagem gerada por IA; o nome "Arcana RPG" que aparece nela é só ilustrativo).

## Conceito

Fantasia **arcana e limpa**: fundo neutro e calmo, com o roxo (magia) e o verde-água (energia) como acentos brilhantes. O ornamento fica nos detalhes (runas, logo, hexágonos), nunca no texto corrido. O site tem **tema claro** e **tema escuro ("Místico")** desde o início.

## Paleta

### Cores-base

| Token | Hex | Uso |
| --- | --- | --- |
| `pergaminho` | `#F0F4F8` | Fundo do tema claro; texto principal do tema escuro (variante `#ECF1F6`) |
| `nevoa` | `#E2EAF1` | Cartões, bordas e divisores do tema claro |
| `abismo` | `#1F2937` | Texto principal do tema claro; cartões do tema escuro |
| `noite` | `#131823` (estimado) | Fundo do tema escuro |
| `arcano` | `#7F56D9` | Cor da marca: logo, links, navegação, foco |
| `essencia` | `#00D28E` | Acento: links secundários, sucesso, brilho das runas |
| `essencia-escura` | `#00BFA5` | Estado *hover* / ativo dos elementos verdes |
| `mare` | `#2C6E8F` (estimado) | Cor de apoio (ícones, gráficos, estados neutros) |

### Regras de contraste (WCAG AA, mínimo 4,5:1 para texto)

| Combinação | Contraste | Pode usar em texto? |
| --- | --- | --- |
| `abismo` sobre `pergaminho` | 13,3 | ✅ |
| `pergaminho` sobre `noite` | 15,6 | ✅ |
| `essencia` sobre `noite` | 9,0 | ✅ |
| `essencia` sobre `pergaminho` | 1,8 | ❌ só preenchimento e decoração |
| `arcano` sobre `pergaminho` | 4,5 | ⚠️ só texto grande ou ícone |
| `arcano` sobre `noite` | 3,6 | ⚠️ só texto grande ou ícone |

Para texto pequeno, use as variantes ajustadas:

| Token | Hex | Onde | Contraste |
| --- | --- | --- | --- |
| `arcano-texto-claro` | `#6941C6` | roxo em texto no tema claro | 6,0 |
| `arcano-texto-escuro` | `#9E77ED` | roxo em texto no tema escuro | 5,3 |
| `essencia-texto-claro` | `#007A62` | verde em texto no tema claro | 4,8 |

As cores das barras da ficha (Vida, Sanidade, Esforço) continuam como estão e precisam ser conferidas sobre os dois fundos.

## Tipografia

| Papel | Fonte | Uso |
| --- | --- | --- |
| Títulos | **Almendra** (Google Fonts, serifa medieval) | Títulos de página e de seção, nome da mesa, nome do personagem |
| Texto | **Literata** (Google Fonts, serifa de leitura) | Texto corrido, chat, formulários, tabelas |

- Almendra só em títulos (a partir de ~20px). Em texto pequeno ela fica difícil de ler.
- Hierarquia da referência: título de página grande e com sublinhado fino em `nevoa`; subtítulo médio; parágrafo ~16px com entrelinha folgada.
- Números da ficha (atributos, valores das barras) em Literata com algarismos tabulares, para não "pular" ao mudar.

## Elementos de marca

- **Logo:** dragão roxo (`arcano`) enrolado em volta de uma runa, dentro de um contorno circular. No tema escuro ganha um brilho suave roxo.
- **Divisor rúnico:** linha horizontal fina que vai de verde-água a roxo, com uma sequência de runas brilhantes no centro. Usado para separar grandes seções (ex.: entre a ficha e o chat), no máximo um por tela.
- **Hexágono:** forma de destaque para amostras de cor, avatares, tokens do tabuleiro e resultado de dados.
- **Runas:** marca d'água discreta nos cantos dos cartões (opacidade baixa), nunca sobre o texto.

## Componentes-base

| Componente | Tema claro | Tema escuro |
| --- | --- | --- |
| Cabeçalho | Fundo `pergaminho`, logo à esquerda, navegação à direita em `abismo` | Fundo `noite`, navegação em `arcano-texto-escuro` |
| Cartão | Fundo `pergaminho` ou `nevoa`, borda 1px `nevoa`, cantos arredondados (~8px), sombra leve | Fundo `abismo`, borda 1px mais clara que o fundo, sem sombra |
| Link / ação secundária | `essencia-texto-claro` | `essencia` |
| Botão principal | Fundo `arcano`, texto branco | Fundo `arcano`, texto branco, brilho no *hover* |
| Divisor simples | 1px `nevoa` | 1px `abismo` clareado |
| Rodapé | Texto pequeno centralizado, separado por uma linha fina | Igual |

## Aplicação no frontend

- As cores viram tokens no `@theme` de `Projeto_RPG/app/globals.css` (ex.: `bg-pergaminho`, `text-arcano`), seguindo a regra R8 (Tailwind sempre).
- O tema escuro usa a variante `dark:` do Tailwind, com a escolha salva por usuário.
- As fontes entram pelo `next/font/google` (Almendra e Literata), sem arquivo `.css` novo.
- O logo e as runas ficam em SVG em `public/img/`.

## Pendências

- Desenhar o logo final (a referência é gerada por IA) e exportar em SVG.
- Confirmar as cores estimadas (`noite`, `mare`) com o arquivo de design.
- Definir o nome do site.
- Conferir as cores atuais da ficha contra a nova paleta.
