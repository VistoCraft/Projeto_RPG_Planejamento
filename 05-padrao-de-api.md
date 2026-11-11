# Padrão de API

Contrato entre o `Projeto_RPG` (frontend) e o `Projeto_RPG_Backend`.

## REST

- JSON com propriedades em `camelCase` (padrão do ASP.NET). Enums viajam como texto em `camelCase` (`"publica"`, `"espectador"`). Datas em ISO 8601 UTC (`2026-11-09T18:00:00Z`). IDs são `Guid` em texto.
- Rotas em português, no plural e em `kebab-case`, aninhadas pela mesa quando o recurso pertence a ela:

| Recurso | Rotas |
| --- | --- |
| Login (Identity) | `/auth/*` (nomes em inglês, vêm do Identity) |
| Cadastro e perfil | `POST /auth/cadastro` (e-mail, senha e perfil, no lugar do `/auth/register`), `GET/PUT /perfil`, `POST /auth/sair` |
| Templates | `GET /templates` (padrão + os do usuário), `POST /templates`, `GET/PUT/DELETE /templates/{id}` |
| Mesas | `GET /mesas` (sem login: só públicas), `POST /mesas`, `GET/PUT/DELETE /mesas/{id}`, `POST /mesas/{id}/encerrar`, `.../reabrir`, `.../codigo` |
| Entrar e sair | `POST /mesas/{id}/entrar`, `POST /mesas/entrar-com-codigo`, `POST /mesas/{id}/sair` |
| Participantes | `PUT /mesas/{id}/participantes/{usuarioId}` (papel), `DELETE /mesas/{id}/participantes/{usuarioId}` (os participantes vêm no `GET /mesas/{id}`) |
| Modelo da ficha da mesa | `GET /mesas/{id}/modelo-ficha`, `POST /mesas/{id}/modelo-ficha/previa` (campos atuais e novos, com a origem padrão de cada um), `PUT /mesas/{id}/modelo-ficha` (novo modelo + conversão de campos) |
| Fichas | `GET/POST /mesas/{id}/fichas`, `GET/PUT/DELETE /mesas/{id}/fichas/{fichaId}`, `GET /mesas/{id}/fichas/{fichaId}/backups` e `/backups/{backupId}` |
| Templates | `GET /templates`, `GET /templates/{id}` (padrão: aberto a visitantes), `POST /templates`, `PUT/DELETE /templates/{id}` (só os próprios) |
| Biblioteca | `GET /biblioteca/fichas?templateId=`, `GET/DELETE /biblioteca/fichas/{fichaId}`, `POST /mesas/{id}/fichas/{fichaId}/salvar` |
| Propostas | `GET/POST /mesas/{id}/propostas`, `POST /mesas/{id}/propostas/{propostaId}/aceitar` e `/recusar`, `DELETE /mesas/{id}/propostas/{propostaId}` |
| Propostas de ficha | `GET/POST /mesas/{id}/propostas-ficha`, `POST /mesas/{id}/propostas-ficha/{propostaId}/aceitar`, `POST .../recusar` |
| Mensagens | `GET /mesas/{id}/mensagens?antesDe=<data>&tamanho=50` |
| Tabuleiro | `GET /mesas/{id}/tabuleiro`, `PUT /mesas/{id}/tabuleiro` |

- `GET /mesas` lista as mesas públicas e as que o usuário participa. Filtros na query: `?busca=`, `?templateId=`, `?status=`, `?minhas=true`.
- Status: `200` leitura e edição com corpo, `201` criação (com `Location`), `204` sem corpo, `400` validação, `401` sem login, `403` sem permissão, `404` não existe ou é privado, `409` conflito (mesa encerrada, entrada bloqueada, dono tentando sair).

## Paginação

- Listas: `?pagina=1&tamanho=20`, resposta `{ "itens": [...], "total": 42 }`. Tamanho máximo 100.
- Chat: por cursor (`?antesDe=<EnviadaEm da mensagem mais antiga>`), resposta `{ "itens": [...], "temMais": true }`.

## Erros

Todo erro sai como **Problem Details** (RFC 9457, `application/problem+json`), o mesmo formato que o Identity já usa:

```json
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": { "MesaCheia": ["A mesa atingiu o limite de jogadores."] }
}
```

- **Validação (400) e conflito (409):** a chave de `errors` é um **código** em `PascalCase` (`NomeObrigatorio`, `EntradaBloqueada`, `CodigoConviteInvalido`). O frontend traduz o código para a mensagem na tela, como já faz com os códigos do Identity (`DuplicateEmail`).
- **401, 403 e 404:** sem `errors`, só `status` e `title`.
- **500:** mensagem genérica; o detalhe fica só no log do servidor.

## Tempo real (SignalR)

- Um hub em `/hubs/mesa`, com cookie de login. O cliente chama `EntrarNaMesa(mesaId)`; o servidor confere se a mesa é pública ou se a pessoa participa e põe a conexão no grupo da mesa (`SairDaMesa` tira).
- **Pronto hoje:** `MesaAtualizada(mesaId)`, `MesaExcluida(mesaId)` `FichasAtualizadas(mesaId)`, `PropostasAtualizadas(mesaId)`, `ModeloAtualizado(mesaId)` (o modelo da mesa mudou e as fichas foram convertidas; quem está com uma ficha aberta busca de novo) e `FichaAtualizada({ mesaId, fichaId, campos })` (enviado só aos usuários que podem ver a ficha, com `Clients.Users`). `FichasAtualizadas`: uma ficha foi criada, editada, excluída ou ficou sem jogador; o cliente busca de novo `GET /mesas/{id}/fichas`, que já filtra o que cada um pode ver). Levam só o id; cada cliente busca de novo o `GET /mesas/{id}`, porque o que cada um vê depende do papel (ex.: código de convite). Evento que carregue dados (como `FichaAtualizada`) precisa ir só para quem pode ver aqueles dados.
- **CRUD fica no REST.** Pelo hub vão só as ações de jogo de alta frequência: `EnviarMensagem`, `Rolar`, `MoverToken`.
- Eventos do servidor para o grupo, no passado: `MensagemRecebida`, `TokenMovido`, `FichaAtualizada`, `ParticipanteEntrou`, `ParticipanteSaiu`, `PresencaAlterada`.
- Mudança feita pelo REST que outros precisam ver (ex.: ficha salva) também emite o evento no grupo.
- **Ficha em edição simultânea:** o cliente envia só os campos alterados (`PATCH /mesas/{id}/fichas/{fichaId}` com `{ campos: { caminho: valor } }`), agrupados com espera curta (~400 ms). O servidor grava campo a campo (vale a última gravação de cada campo) e emite `FichaAtualizada` com os mesmos campos, só para quem pode ver a ficha. Quem recebe aplica os campos, menos o campo que estiver editando naquele momento.

## Organização no backend

- Uma pasta por assunto (`Mesas/`, `Fichas/`, `Templates/`...) com a entidade, os DTOs (`<Nome>Requisicao`, `<Nome>Resposta`) e um `<Assunto>Endpoints.cs` com o método `Map<Assunto>()`.
- A entidade nunca sai direto na resposta: sempre um DTO.
- Toda rota exige login (`RequireAuthorization`), menos `/saude`, as públicas do `/auth` e as de leitura do mostruário (`GET /mesas` e `GET /templates` sem login devolvem só mesas públicas e templates padrão, com DTO sem e-mail).

## Organização no frontend

- Toda chamada é um método num repositório de `src/repositories/` (regra R6), que converte o DTO no tipo de domínio de `src/dominio/` e os códigos de erro em mensagens.
