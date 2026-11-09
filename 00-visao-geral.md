# Visão geral

## O que é

Um site para jogar RPG de mesa online. Os jogadores entram num **hub**, escolhem uma **mesa** e, dentro dela, conversam, cuidam das fichas e jogam num tabuleiro.

## Estrutura do site

```
Hub
└─ Lista de mesas
   └─ Mesa
      ├─ Chat
      ├─ Fichas
      └─ Tabuleiro
```

| Tela / área | O que faz |
| --- | --- |
| **Hub** | Página inicial depois do login. Mostra a lista de mesas e permite criar uma mesa ou entrar em uma. |
| **Lista de mesas** | Cada item mostra nome, mestre, sistema, número de jogadores e status (aberta, em sessão, encerrada). |
| **Mesa** | Espaço de uma campanha. Reúne o chat, as fichas e o tabuleiro dos participantes. |
| **Chat** | Mensagens em tempo real entre os participantes da mesa, incluindo resultados de rolagens de dados. |
| **Fichas** | As fichas dos personagens da mesa. A ficha do jogador do `Projeto_RPG` é a base. |
| **Tabuleiro** | Mapa compartilhado com grade, onde ficam os tokens dos personagens e as imagens de cena. |

## Conceitos (entidades)

| Entidade | Descrição |
| --- | --- |
| Usuário | Pessoa com conta no site. Pode ser mestre em uma mesa e jogador em outra. |
| Mesa | Campanha com nome, sistema, mestre e participantes. |
| Participante | Ligação entre usuário e mesa, com papel (`mestre` ou `jogador`). |
| Mensagem | Item do chat de uma mesa: texto ou resultado de rolagem. |
| Ficha | Personagem de um participante numa mesa (detalhes, status, atributos, perícias, combate). |
| Tabuleiro | Mapa de uma mesa: imagem de fundo, grade e tokens. |
| Token | Peça no tabuleiro, ligada ou não a uma ficha, com posição (x, y). |

## Ponto de partida

O `Projeto_RPG` já tem a **ficha do jogador** funcionando no navegador:
- é uma página só;
- salva os dados no `localStorage`;
- não tem backend nem login.

Para virar o site, a ficha passa a pertencer a uma mesa e a ser salva no backend.
