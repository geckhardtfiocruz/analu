# Registro de dados de uso

Nota para o time técnico. O jogo já grava e mostra os dados. Falta um serviço que guarde os registros de
todos os jogadores num lugar comum.

## Decisões do produto

- **Anônimo.** Sem nome, e-mail, localização ou identificador do aparelho. Cada abertura do jogo recebe um
  código aleatório (`sessao`).
- **Guarda tudo:** acessos, casos iniciados, cada resposta, o placar de cada caso e o jogo completo.
- **Painel aberto a todos**, no botão "Dados de uso" (tela inicial e tela de fontes).
- **Envio depois, sem prejuízo.** Todo registro fica primeiro no aparelho e é enviado quando houver internet.
  Se o envio falhar, o jogo continua igual e o registro fica na fila para a próxima tentativa.

## Como ligar

Em `index.html`, preencher `var DADOS_URL = "";` com o endereço do serviço. Com o campo vazio, o painel
mostra só as partidas do próprio aparelho e avisa que o painel geral ainda não está ligado.

O jogo está publicado no GitHub Pages (`https://geckhardtfiocruz.github.io/analu/`), então o serviço
precisa aceitar chamadas vindas dessa origem, sem login.

## O que o serviço precisa fazer

**Receber** — `POST DADOS_URL`, corpo em texto com JSON (`Content-Type: text/plain`, para não exigir
pré-verificação do navegador):

```json
{ "registros": [ { "id": "...", "sessao": "...", "tipo": "resposta", "quando": "2026-09-30T13:05:12.000Z",
                   "caso": "marta", "consulta": 2, "letra": "B", "pontos": 25 } ] }
```

Deve responder com status 2xx depois de gravar. Lotes de até 200 registros. O mesmo `id` pode chegar mais
de uma vez (por exemplo, se a resposta se perder). O painel já descarta duplicados, mas o serviço também
pode ignorá-los.

**Devolver** — `GET DADOS_URL` retorna todos os registros: `{ "registros": [ ... ] }` ou uma lista
simples. O painel faz as contas no navegador e ignora registros que não correspondem a caso, consulta e
alternativa existentes no jogo.

## Tipos de registro

| tipo | quando | campos além de id, sessao, tipo, quando |
|---|---|---|
| `acesso` | ao abrir o jogo | — |
| `inicio_caso` | ao escolher um paciente (inclusive ao refazer) | `caso` |
| `resposta` | a cada escolha de conduta | `caso`, `consulta` (1 em diante), `letra` (A–D), `pontos` |
| `fim_caso` | ao terminar a última consulta | `caso`, `pontos`, `pct` (0–100), `acertos` |
| `jogo_completo` | quando o 5º caso diferente é concluído | — |

Caso abandonado = `inicio_caso` sem `fim_caso` na mesma sessão.

## Pontos a validar com Engenharia

- Onde guardar e como impedir que alguém envie registros falsos em massa. Como o envio é aberto, o painel
  pode ser inflado por quem quiser; o jogo só filtra registros com formato inválido.
- Volume: o painel baixa todos os registros a cada abertura. Com muitos acessos, pode ser preciso o serviço
  devolver os números já somados.
- A fila fica no navegador do jogador. Se ele limpar os dados do navegador ou nunca mais abrir o jogo, os
  registros que não foram enviados se perdem. Essa perda é aceita pela decisão "sem prejuízo se não enviar".
