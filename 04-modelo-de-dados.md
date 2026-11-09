# Modelo de dados

Entidades do backend (PostgreSQL via EF Core). Chaves são `Guid`, datas em UTC (`timestamptz`). Os nomes ficam em português, como no código.

## Diagrama

```
Template  1──N Mesa
Mesa      1──N Participante N──1 Usuario
Mesa      1──N Ficha        N──1 Usuario (jogador atribuído; ficha sem mesa = biblioteca)
Template  1──N Ficha
Ficha     1──N FichaBackup
Mesa      1──N PropostaFicha N──1 Ficha (da biblioteca)
Mesa      1──N Mensagem     N──1 Usuario (autor)
Mesa      1──1 Tabuleiro    1──N Token
Token     N──1 Ficha (opcional)
```

## Entidades

### Usuario (já existe, Identity)

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | Chave do Identity |
| `Email` | texto | Login (e `UserName`) |
| `Apelido` | texto | Nickname exibido no site; obrigatório e único |
| `Pais` | texto | Código ISO do país; obrigatório |
| `DataNascimento` | data | No lugar da idade, que muda com o tempo; obrigatória, mínimo de 18 anos |
| `ComoConheceu` | texto? | Opção de uma lista ou texto livre; o único campo opcional |
| `VersaoTermosAceita`, `TermosAceitosEm` | texto?, data? | Versão dos termos aceita no cadastro e quando |
| `PerfilCompleto` | bool | `false` depois do login pelo Google, até a etapa de perfil ser preenchida |

No banco, os campos do perfil aceitam nulo (contas antigas e a etapa do Google); a obrigatoriedade é validada no `POST /auth/cadastro`.
| `AvatarUrl` | texto? | Entra com o upload de imagens |

### Template

Modelo de ficha: diz quais campos a ficha tem e como cada atributo é rolado. Pode ser **padrão** (do site) ou **custom** (criado por um usuário, na área de templates dele).

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `Nome` | texto | |
| `Descricao` | texto? | |
| `ModeloFicha` | jsonb | Atributos, perícias, barras, condições, colunas de combate e a **fórmula de rolagem** de cada atributo (ex.: `1d20 + mod(forca)`). Formato em [06-modelo-de-ficha.md](06-modelo-de-ficha.md) |
| `ModeloFichaSimplificada` | jsonb | Os campos da ficha simplificada (pets e NPCs do jogador) desse template |
| `CriadorId` | Guid? → Usuario | `null` = template padrão do site; preenchido = template custom, visível só para o criador |

- No lançamento há um template padrão, **5e**, feito só com o conteúdo do SRD 5.1 (licença CC-BY-4.0, com o texto de atribuição e sem a marca "D&D"), criado por seed. Ordem Paranormal fica para depois.
- A ficha atual do frontend era um teste e será refeita do zero.
- Um template **nunca é alterado por uma mesa**.
- Depois do lançamento: compartilhar templates prontos numa aba do hub (campo de visibilidade no template).

### Mesa

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `Nome` | texto | Até 80 caracteres |
| `Descricao` | texto? | |
| `TemplateId` | Guid → Template | Template de origem; não muda depois de criada |
| `ModeloFicha`, `ModeloFichaSimplificada` | jsonb | Cópias dos modelos do template, feitas na criação da mesa; o mestre pode alterar sem mexer no template |
| `VersaoModelo` | int | Sobe a cada alteração do `ModeloFicha` da mesa |
| `Visibilidade` | enum | `Publica` ou `Privada` |
| `Status` | enum | `Ativa` ou `Encerrada` (encerrada = só leitura; o dono pode reabrir) |
| `EntradaBloqueada` | bool | O mestre ou ajudante impede a entrada de novas pessoas (direta ou por código). Não há limite de jogadores |
| `CodigoConvite` | texto | 8 caracteres, único, pode ser gerado de novo pelo mestre |
| `CriadaEm` | data | |

### Participante

Liga usuário e mesa. Chave composta (`MesaId`, `UsuarioId`).

| Campo | Tipo | Observação |
| --- | --- | --- |
| `MesaId` | Guid → Mesa | |
| `UsuarioId` | Guid → Usuario | |
| `Papel` | enum | `Dono` (o mestre, exatamente um, não transferível), `CoMestre`, `Assistente`, `Jogador` ou `Espectador` (papel de quem entra) |
| `EntrouEm` | data | |

O mestre é o participante com papel `Dono` (não há `MestreId` na mesa, para não duplicar a informação).

### Ficha

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `MesaId` | Guid? → Mesa | `null` = ficha salva na biblioteca do jogador |
| `TemplateId` | Guid → Template | Template de origem; decide em quais mesas a ficha pode ser usada |
| `DonoId` | Guid? → Usuario | Jogador a quem o mestre atribuiu a ficha; `null` = NPC do mestre. Na biblioteca, o dono da cópia |
| `Tipo` | enum | `Completa` ou `Simplificada` (pets e NPCs do jogador) |
| `NomePersonagem` | texto | Para listar sem abrir o JSON |
| `Dados` | jsonb | Valores da ficha, no formato do `ModeloFicha` (ou do `ModeloFichaSimplificada`) da mesa |
| `VersaoModelo` | int | Versão do `ModeloFicha` da mesa em que os `Dados` estão |
| `VisivelParaJogadores` | bool | Definido pelo mestre: se os outros jogadores da mesa podem ver esta ficha (só leitura) |
| `AtualizadaEm` | data | Última gravação; em edição simultânea, vale a última |

- `Dados` em `jsonb` porque cada mesa tem campos diferentes. O backend valida os dados contra o `ModeloFicha` da mesa.
- **Só o mestre cria fichas.** Ele atribui uma ou mais a um jogador, que pode editar e salvar as suas. Um jogador pode ter várias fichas na mesma mesa.
- **Biblioteca:** salvar cria uma cópia independente com `MesaId = null`. Acontece pelo botão do jogador e automaticamente para todas as fichas atribuídas quando a mesa é excluída.

**Implementado na 4.3** (`Fichas/` no backend):

- Ficha nova: dados montados a partir do modelo da mesa (`ModeloFicha` ou `ModeloFichaSimplificada`, conforme o tipo): detalhes vazios (o `nome` recebe o nome do personagem), atributos no padrão, perícias não treinadas, barras cheias, sem condições e sem ataques.
- Só quem tem papel **Jogador** recebe ficha. Espectadores veem só as fichas marcadas como visíveis, como os outros jogadores.
- Quando o jogador sai ou é removido da mesa, as fichas dele continuam na mesa, sem jogador (o mestre pode entregar a outro).
- Ao excluir a mesa, as fichas com jogador vão para a biblioteca (`MesaId = null`) ligadas ao template "(modelo da mesa)" criado na exclusão; as fichas sem jogador (NPCs do mestre) são apagadas junto.

### FichaBackup

Versão antiga de uma ficha, guardada antes de uma conversão.

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `FichaId` | Guid → Ficha | |
| `Dados` | jsonb | Os dados como estavam |
| `ModeloFicha` | jsonb | O modelo da mesa naquela versão, para conseguir exibir o backup |
| `VersaoModelo` | int | |
| `CriadoEm` | data | |

### PropostaFicha

Pedido de um jogador para levar uma ficha da biblioteca a uma mesa.

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `MesaId` | Guid → Mesa | Precisa ter o mesmo `TemplateId` da ficha |
| `FichaId` | Guid → Ficha | A ficha da biblioteca (`MesaId = null`) |
| `JogadorId` | Guid → Usuario | Quem propôs; precisa ser participante da mesa |
| `Status` | enum | `Pendente`, `Aceita` ou `Recusada` |
| `CriadaEm` | data | |

Ao aceitar, o mestre gera uma cópia da ficha na mesa, já atribuída ao jogador. Se o modelo da mesa for diferente do modelo em que a ficha foi salva, o mestre usa a mesma conversão de campos de quando altera o modelo.

**Alterar o modelo da mesa:** o mestre edita o `ModeloFicha` e, no mesmo passo, indica como converter os campos antigos nos novos (ex.: `forca` → `poder`). Para cada ficha da mesa, o backend grava um `FichaBackup`, aplica a conversão nos `Dados` (campo sem destino sai, campo novo entra vazio) e atualiza a `VersaoModelo`.

### Mensagem

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `MesaId` | Guid → Mesa | |
| `AutorId` | Guid → Usuario | |
| `Tipo` | enum | `Texto` ou `Rolagem` |
| `Conteudo` | texto | Texto da mensagem ou a expressão rolada (`1d20+3`) |
| `Resultado` | jsonb? | Dados de cada dado e total, quando `Tipo = Rolagem` |
| `DestinatarioId` | Guid? → Usuario | Sussurro; `null` = mensagem para a mesa |
| `EnviadaEm` | data | Índice (`MesaId`, `EnviadaEm`) para a paginação |

### Tabuleiro

Um por mesa (1:1) por enquanto.

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `MesaId` | Guid → Mesa | Único |
| `ImagemUrl` | texto? | Mapa de fundo |
| `TamanhoCelula` | int | Pixels por célula da grade |
| `Colunas`, `Linhas` | int | Tamanho da grade |

### Token

| Campo | Tipo | Observação |
| --- | --- | --- |
| `Id` | Guid | |
| `TabuleiroId` | Guid → Tabuleiro | |
| `FichaId` | Guid? → Ficha | Liga o token a um personagem (mostra a Vida) |
| `Nome` | texto | |
| `ImagemUrl` | texto? | |
| `X`, `Y` | int | Posição em células |
| `Tamanho` | int | Células ocupadas (1 = normal) |
| `Oculto` | bool | Só o mestre vê |

## Regras de acesso

| Ação | Quem pode |
| --- | --- |
| Ver mesa pública na lista | Qualquer pessoa, até visitante; abrir a mesa pede login |
| Ver mesa privada | Só participantes; para os outros a API responde 404 |
| Entrar em mesa pública | Qualquer usuário logado, como espectador, se a entrada não estiver bloqueada e a mesa estiver ativa |
| Entrar em mesa privada | Quem tem o código de convite, nas mesmas condições |
| Editar dados da mesa, bloquear entrada, gerar novo código, encerrar e reabrir, alterar o modelo da ficha da mesa, mudar papel entre espectador e jogador, remover jogador ou espectador | Dono ou co-mestre |
| Nomear, trocar ou remover co-mestres e assistentes; excluir a mesa | Dono |
| Sair da mesa | Qualquer participante, menos o dono |
| Criar e editar template custom | Qualquer usuário logado, na própria área de templates |
| Criar ficha, atribuir ficha a jogador, cuidar do tabuleiro | Dono, co-mestre ou assistente |
| Ver ficha | Mestre, ajudante e o jogador da ficha; os outros jogadores só se `VisivelParaJogadores` estiver ligado |
| Editar e salvar ficha | Jogador a quem ela foi atribuída, ou mestre |
| Mudar a visibilidade da ficha para os outros jogadores | Dono, co-mestre ou assistente |
| Salvar ficha na biblioteca; propor ficha da biblioteca para uma mesa | Jogador a quem ela foi atribuída / dono da cópia |
| Aceitar ou recusar proposta de ficha | Mestre |
| Ver backups de uma ficha | Mestre e o jogador da ficha |
| Chat e tabuleiro | Participantes; mover token alheio ou oculto só o mestre |

## Implementação

As entidades entram no backend na fase em que são usadas: `Template`, `Mesa` e `Participante` no hub; `Ficha`, `FichaBackup` e `PropostaFicha` na fase de fichas; `Mensagem` no chat; `Tabuleiro` e `Token` no tabuleiro.
