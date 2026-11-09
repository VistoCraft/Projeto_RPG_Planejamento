# Lista de tarefas

Marque `[x]` quando terminar. A ordem das seções é a ordem sugerida de execução.

## 1. Fundação

- [x] Responder as perguntas de [02-duvidas-em-aberto.md](02-duvidas-em-aberto.md) que bloqueiam o backend (stack, banco, tempo real)
- [x] Escolher a stack do backend e iniciar o `Projeto_RPG_Backend`
- [ ] Definir o modelo de dados: usuário, mesa, participante, mensagem, ficha, tabuleiro, token
- [x] Configurar banco de dados e migrations (PostgreSQL 18 local, senha no User Secrets, migration `Inicial` aplicada)
- [ ] Definir o padrão de API (REST, tempo real) e o formato de erros
- [ ] Configurar variáveis de ambiente (`.env.example`) nos dois projetos
- [ ] Configurar lint, formatação e `yarn build` / testes no CI
- [x] Aplicar a [identidade visual](03-identidade-visual.md) no frontend (tokens no `@theme`, fontes, tema claro e escuro)

## 2. Contas e login

- [x] Cadastro de usuário (tela `/cadastro` + `/auth/register`; ainda sem o nome)
- [ ] Login e logout (login pronto na tela `/login`; falta o logout)
- [x] Sessão / token de autenticação (cookie do Identity)
- [ ] Login com Google
- [ ] Login com Discord
- [ ] Proteger as rotas do frontend que exigem login
- [ ] Página de perfil (nome e avatar)

## 3. Hub e lista de mesas

- [x] Rota do hub no frontend (hub em `/`, ficha em `/ficha`)
- [ ] API: listar mesas
- [ ] Listagem de mesas com nome, mestre, sistema, número de jogadores e status
- [ ] Busca e filtro de mesas
- [ ] Criar mesa (nome, sistema, descrição, limite de jogadores)
- [ ] Entrar numa mesa (convite por link ou código)
- [ ] Sair de uma mesa
- [ ] Mestre: editar, fechar e excluir a mesa
- [ ] Mestre: remover jogador

## 4. Mesa

- [ ] Rota da mesa (`/mesas/[id]`) com layout para chat, fichas e tabuleiro
- [ ] Mostrar os participantes e quem está online
- [ ] Permissões por papel (mestre e jogador)

## 5. Chat

- [x] Escolher a tecnologia de tempo real: SignalR
- [ ] Enviar e receber mensagens em tempo real
- [ ] Histórico de mensagens com paginação
- [ ] Rolagem de dados pelo chat (ex.: `/r 1d20+3`) com o resultado visível para todos
- [ ] Mensagem privada para o mestre (sussurro)
- [ ] Notificação de mensagem nova

## 6. Fichas

- [ ] Ligar a ficha atual a uma mesa e a um participante
- [ ] Salvar a ficha no backend no lugar do `localStorage`
- [ ] Listar as fichas da mesa
- [ ] Mestre vê e edita todas as fichas; jogador edita só a própria
- [ ] Terminar o que está pendente na ficha (ver `Projeto_RPG/ai/07-pendencias.md`):
  - [ ] Salvar atributos, perícias, condições e combate
  - [ ] Valor máximo editável nas barras de Vida, Sanidade e Esforço
  - [ ] Botão de rolagem (d20) dos atributos, com o resultado enviado ao chat
  - [ ] Botões "ver" / "editar" das seções
  - [ ] Favoritar perícias
  - [ ] Tabela de combate editável (adicionar e remover armas)
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
