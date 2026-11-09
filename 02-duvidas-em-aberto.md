# Dúvidas em aberto

Quando uma dúvida for respondida, escreva a resposta e a data embaixo dela e atualize [01-lista-de-tarefas.md](01-lista-de-tarefas.md) se for o caso.

## Bloqueiam o backend

- **Stack do backend:** Node (NestJS, Express, Fastify), outra linguagem, ou um serviço pronto (Supabase, Firebase)?
  - **Resposta (2026-10-21):** ASP.NET Core com .NET 10.
- **Banco de dados:** relacional (PostgreSQL) ou documento (MongoDB)?
  - **Resposta (2026-10-21):** relacional, PostgreSQL com Entity Framework Core.
- **Tempo real** (chat e tabuleiro): WebSocket próprio, Socket.IO ou serviço pronto?
  - **Resposta (2026-10-21):** SignalR, com um grupo por mesa. Serve ao chat e à sincronização dos tokens do tabuleiro.
- **Login:** e-mail e senha, login social (Google, Discord) ou os dois?
  - **Resposta (2026-10-21):** os dois. ASP.NET Core Identity com e-mail e senha agora; Google e Discord depois, como login externo.

## Produto

- **Sistema de RPG:** a ficha é de um sistema só (a atual tem Sanidade e Exposição Paranormal) ou cada mesa escolhe o sistema?
- **Mesas públicas ou privadas:** qualquer um vê a lista de mesas, ou só as mesas em que participa?
- **Limite de jogadores** por mesa?
- **Rolagens:** o resultado é público para a mesa toda? O mestre pode rolar escondido?
- **Tabuleiro:** é obrigatório em toda mesa ou opcional?
- **Fichas fora de mesa:** um jogador pode ter uma ficha sem estar numa mesa?

- **Nome do site:** a referência visual usa "Arcana RPG" só como exemplo. Qual será o nome?

## Técnica

- **Monorepo ou repositórios separados:** manter `Projeto_RPG` e `Projeto_RPG_Backend` separados ou juntar?
- **Onde guardar imagens:** disco do servidor, S3 ou outro storage?
- **Hospedagem:** Vercel para o frontend? Onde fica o backend?
