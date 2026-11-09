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
  - **Resposta (2026-11-09):** cada mesa tem o seu sistema. O sistema define o modelo da ficha (ver [04-modelo-de-dados.md](04-modelo-de-dados.md)).
  - **Resposta (2026-11-09):** os dois. Haverá templates de ficha padrão (D&D 5e e Ordem Paranormal) e fichas custom criadas pelo mestre. Dentro de uma mesa o mestre pode alterar a ficha padrão, mas o template original não muda.
  - **Resposta (2026-11-09):** a ficha atual era um teste e será refeita do zero. O lançamento terá só o template 5e, feito com o conteúdo de licença aberta (SRD 5.1), e a criação de ficha própria; Ordem Paranormal fica para depois. A rolagem de cada atributo é uma fórmula cadastrada no template. As colunas "nun." da tabela de combate ficam de fora por enquanto.
  - **Resposta (2026-11-09):** quando o mestre altera o modelo da mesa, cada ficha ganha um backup e a ficha ativa é convertida, com o mestre indicando como os campos antigos viram os novos. Templates custom ficam numa área de templates do usuário; depois, poderão ser compartilhados numa aba do hub.
- **Mesas públicas ou privadas:** qualquer um vê a lista de mesas, ou só as mesas em que participa?
  - **Resposta (2026-11-09):** o mestre escolhe. Mesa pública aparece no hub para todos e qualquer usuário logado entra; mesa privada só aparece para os participantes e se entra pelo código de convite.
- **Limite de jogadores** por mesa?
  - **Resposta (2026-11-09):** sem limite; o mestre pode bloquear a entrada de novas pessoas.
- **Entrada e papéis na mesa:** como se entra e quem manda?
  - **Resposta (2026-11-09):** entra-se direto (mesa pública) ou por código, sempre como espectador; o mestre promove a jogador. O mestre pode dar permissões a ajudantes, mas não passa a posse da mesa.
  - **Resposta (2026-11-09):** excluir a mesa pede confirmação e apaga tudo, menos as fichas dos jogadores (vão para a biblioteca de cada um) e o modelo de ficha (vai para os templates do mestre). Mesa encerrada fica só para leitura; o mestre pode reabrir ou excluir.
  - **Ainda em aberto:** o que exatamente o ajudante pode fazer (hoje: gerenciar entrada, código, jogadores e espectadores; não encerra, não exclui, não nomeia ajudantes)?
- **Rolagens:** o resultado é público para a mesa toda? O mestre pode rolar escondido?
- **Tabuleiro:** é obrigatório em toda mesa ou opcional?
- **Fichas fora de mesa:** um jogador pode ter uma ficha sem estar numa mesa?
  - **Resposta (2026-11-09):** a ficha vive na mesa, mas o jogador pode salvá-la na própria biblioteca, para não perdê-la se a mesa for excluída ou para usar o mesmo personagem em outra mesa com o mesmo template.
  - **Resposta (2026-11-09):** salvar é manual e também automático na exclusão da mesa; a cópia é independente. Só o mestre cria fichas e atribui uma ou mais a um jogador, que edita e salva as suas. Há fichas simplificadas para pets e NPCs do jogador.
  - **Resposta (2026-11-09):** para levar uma ficha da biblioteca a outra mesa, o jogador propõe e o mestre aceita. Cada template define também o modelo da ficha simplificada.

- **Hub para visitantes:** quem não está logado vê o hub?
  - **Resposta (2026-11-09):** sim, com limitações, como mostruário. Uma home em `/` apresenta o produto.
  - **Resposta (2026-11-09):** o visitante vê a lista de mesas públicas e os templates padrão; criar mesa, entrar, abrir mesa e chat pedem login. O hub fica em `/mesas`.
- **Dados do cadastro:** o que se pede ao criar a conta?
  - **Resposta (2026-11-09):** o perfil é preenchido no cadastro. No login pelo Google, uma etapa extra pede país, nickname, idade e como conheceu o site.
  - **Resposta (2026-11-09):** idade mínima de 18 anos por enquanto, por segurança. Nickname (único), país e data de nascimento são obrigatórios; só "como conheceu" é opcional.

- **Nome do site:** a referência visual usa "Arcana RPG" só como exemplo. Qual será o nome?

## Técnica

- **Monorepo ou repositórios separados:** manter `Projeto_RPG` e `Projeto_RPG_Backend` separados ou juntar?
- **Onde guardar imagens:** disco do servidor, S3 ou outro storage?
- **Hospedagem:** Vercel para o frontend? Onde fica o backend?
