# Lista de tarefas

Marque `[x]` quando terminar. A ordem das seções é a ordem sugerida de execução.

## 1. Fundação

- [x] Responder as perguntas de [02-duvidas-em-aberto.md](02-duvidas-em-aberto.md) que bloqueiam o backend (stack, banco, tempo real)
- [x] Escolher a stack do backend e iniciar o `Projeto_RPG_Backend`
- [x] Definir o modelo de dados: usuário, sistema, mesa, participante, mensagem, ficha, tabuleiro, token ([04-modelo-de-dados.md](04-modelo-de-dados.md))
- [x] Configurar banco de dados e migrations (PostgreSQL 18 local, senha no User Secrets, migration `Inicial` aplicada)
- [x] Definir o padrão de API (REST, tempo real) e o formato de erros ([05-padrao-de-api.md](05-padrao-de-api.md); Problem Details já ligado no backend)
- [x] Configurar variáveis de ambiente (`.env.example`) nos dois projetos
- [x] CI com `yarn build` (frontend) e `dotnet build` (backend) no GitHub Actions
- [ ] Lint e formatação (ESLint/Prettier no frontend, `dotnet format` no backend) e testes no CI
- [x] Aplicar a [identidade visual](03-identidade-visual.md) no frontend (tokens no `@theme`, fontes, tema claro e escuro)

## 2. Contas e login

- [x] Cadastro de usuário (tela `/cadastro` + `/auth/register`)
- [x] Cadastro com perfil: nickname (único), país e data de nascimento (18+) obrigatórios; como conheceu opcional (`POST /auth/cadastro`)
- [x] Login e logout (`POST /auth/sair`; nickname e botão Sair no cabeçalho)
- [x] Sessão / token de autenticação (cookie do Identity)
- [ ] Login com Google, com etapa para completar o perfil (país, nickname, idade, como conheceu)
- [ ] Login com Discord
- [x] Visitante navega pela home e pelo hub como mostruário (sessão conferida por `GET /perfil`)
- [ ] Exigir login nas ações (criar mesa, entrar, abrir mesa, fichas), conforme elas forem criadas
- [x] Política de privacidade e termos aceitos no cadastro (LGPD), com versão e data do aceite
- [ ] Tela de exclusão de conta (hoje o pedido é por e-mail)
- [ ] Página de perfil (editar dados e avatar)

## 3. Hub e lista de mesas

- [x] Rota do hub no frontend (hub em `/`, ficha em `/ficha`)
- [x] Home do produto em `/`; o hub vai para `/mesas`
- [x] Hub aberto a visitantes, com aviso para criar conta
- [x] API: listar mesas (públicas para visitantes; públicas e as minhas para logados)
- [x] Listagem de mesas com nome, mestre, template, jogadores, espectadores e situação
- [x] Busca e filtro de mesas (nome, situação, só as minhas)
- [x] Criar mesa (nome, descrição, template, visibilidade)
- [x] Entrar numa mesa (direto, se pública, ou por código), sempre como espectador
- [x] Sair de uma mesa
- [x] Mestre: bloquear a entrada, gerar novo código, encerrar, reabrir e excluir (com confirmação)
- [x] Mestre e ajudante: editar nome, descrição e visibilidade pela tela
- [x] Mestre e ajudante: mudar papel (espectador, jogador, ajudante) e remover participante

## 4. Mesa

- [x] Rota da mesa (`/mesas/[id]`) com layout para chat, fichas e tabuleiro (áreas ainda vazias)
- [x] Mostrar os participantes
- [ ] Mostrar quem está online (com o SignalR)
- [x] Permissões por papel (dono, ajudante, jogador, espectador)

## 5. Chat

- [x] Escolher a tecnologia de tempo real: SignalR
- [ ] Enviar e receber mensagens em tempo real
- [ ] Histórico de mensagens com paginação
- [ ] Rolagem de dados pelo chat (ex.: `/r 1d20+3`) com o resultado visível para todos
- [ ] Mensagem privada para o mestre (sussurro)
- [ ] Notificação de mensagem nova

## 6. Fichas

- [ ] Refazer a ficha do zero (a atual era um teste), montada a partir do modelo (`ModeloFicha`)
- [ ] Ligar a ficha a uma mesa e a um participante
- [ ] Template padrão 5e com o conteúdo do SRD 5.1 (CC-BY-4.0, com atribuição)
- [ ] Ficha simplificada para pets e NPCs do jogador, com modelo próprio em cada template
- [ ] Mestre cria fichas e atribui uma ou mais a cada jogador
- [ ] Fórmula de rolagem por atributo cadastrada no template, com o resultado enviado ao chat
- [ ] Biblioteca do jogador: salvar cópia da ficha (manual e automático ao excluir a mesa)
- [ ] Jogador propõe ficha da biblioteca para uma mesa; mestre aceita ou recusa
- [ ] Mestre altera o modelo da ficha da mesa, com backup das fichas e conversão de campos antigos para novos
- [ ] Área de templates do usuário, com editor de template custom
- [ ] Salvar a ficha no backend no lugar do `localStorage`
- [ ] Listar as fichas da mesa
- [ ] Mestre vê e edita todas as fichas; jogador edita só as atribuídas a ele
- [ ] Mestre define, por ficha, se os outros jogadores podem ver aquela ficha
- [ ] Base do SignalR (hub da mesa, grupo por mesa) e ficha sincronizada em tempo real, enviando só os campos alterados
- [ ] Na ficha nova: valor máximo editável nas barras, modos "ver" / "editar", favoritar perícias, tabela de combate editável
- [ ] Layout responsivo da ficha (hoje tem largura fixa de 940px)

## 7. Tabuleiro

- [ ] Escolher a forma de desenhar (canvas, SVG ou biblioteca)
- [ ] Grade com tamanho configurável
- [ ] Imagem de fundo (mapa) enviada pelo mestre
- [ ] Tokens: adicionar, mover e remover
- [ ] Ligar o token a uma ficha (mostrar a Vida na peça)
- [ ] Sincronizar os movimentos em tempo real para todos da mesa
- [ ] Zoom e arrastar o mapa
- [ ] Mestre: esconder partes do mapa (névoa)

## 8. Entrega

- [ ] Upload de imagens (avatar, mapa, token) e onde guardar os arquivos
- [ ] Deploy do frontend
- [ ] Deploy do backend e do banco
- [ ] Domínio e HTTPS
- [ ] Monitoramento de erros

## 9. Depois do lançamento

- [ ] Template Ordem Paranormal (após verificar a licença)
- [ ] Aba no hub para compartilhar templates prontos
