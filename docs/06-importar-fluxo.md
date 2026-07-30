# 06 — 📥 Importando e configurando o fluxo

[⬅️ Anterior: Gemini](05-gemini.md) · [Índice](../README.md) · [Próximo: Entendendo o fluxo ➡️](07-entendendo-o-fluxo.md)

---

## Passo 1 — Importar

1. No n8n, clique em **Create Workflow** (botão no canto superior direito)
2. No canvas vazio, clique nos **três pontinhos `⋯`** do canto superior direito
3. Escolha **Import from File...**
4. Selecione `workflows/assistente-vendas.json` deste projeto

O fluxo aparece com todos os nós e as anotações amarelas explicativas. 🎉

Renomeie o fluxo no topo para `Assistente de Vendas`.

---

## Passo 2 — Ligar as credenciais (5 nós)

Nós sem credencial ficam com um **triângulo vermelho ⚠️**. Vamos resolver um a um.

Abra cada nó com **duplo clique**, escolha a credencial no campo do topo, e
feche.

| # | Nó | Credencial |
|---|-----|-----------|
| 1 | **Telegram Trigger** | `Telegram Vendas` |
| 2 | **Baixar Audio** | `Telegram Vendas` |
| 3 | **Transcrever Audio** | `Gemini Vendas` |
| 4 | **Gemini 2.5 Flash** | `Gemini Vendas` |
| 5 | **Responder no Telegram** | `Telegram Vendas` |
| 6 | **consultar_catalogo** | `Google Sheets Vendas` |
| 7 | **registrar_pedido** | `Google Sheets Vendas` |

> 💡 Nos nós de planilha, o campo **Authentication** já vem como
> *Service Account*. Só escolha a credencial abaixo dele.

---

## Passo 3 — Colocar o ID da planilha (2 nós)

Os dois nós de planilha vêm com um texto de aviso no lugar do ID:

```
COLE_AQUI_O_ID_DA_SUA_PLANILHA
```

### No nó `consultar_catalogo`

1. Duplo clique no nó
2. No campo **Document**, o modo já está em **By ID**
3. Apague o texto e cole o **ID da sua planilha** (do doc 04, passo 4)
4. Confirme que **Sheet** está como **By Name** com o valor `Produtos`

### No nó `registrar_pedido`

Mesma coisa, mas:
- **Sheet** = `Pedidos`
- **Operation** = `Append Row` (já vem assim)

> 🎯 **Dica de aula:** depois de colar o ID, troque o modo do campo para
> **From list**. Se a lista carregar suas planilhas, é prova de que a credencial
> e o compartilhamento estão certos. Se der erro aqui, volte ao doc 04.

---

## Passo 4 — Conferir o cérebro do agente

Duplo clique no nó **Agente de Vendas**. Você vai ver:

| Campo | Valor |
|-------|-------|
| **Source for Prompt** | Define below |
| **Prompt** | `{{ $json.mensagem }}` |
| **System Message** | o prompt de vendas inteiro |

O **System Message** é onde mora a técnica de vendas — persona, etapas da venda,
tratamento de objeções, regras. Leia com calma, é o coração do projeto.

📖 O mesmo texto está em [`prompts/system-prompt-vendedor.md`](../prompts/system-prompt-vendedor.md),
mais confortável de ler e editar.

---

## Passo 5 — Conferir a memória

Duplo clique no nó **Memoria da Conversa**:

| Campo | Valor | Por quê |
|-------|-------|---------|
| **Session ID** | Define below | Vamos usar nossa própria chave |
| **Key** | `{{ $json.sessionId }}` | É o ID do chat do Telegram |
| **Context Window Length** | `30` | Guarda as últimas 30 mensagens |

> ⚠️ **Se o campo Key estiver vazio, todos os clientes vão compartilhar a mesma
> memória.** Esse é o bug mais comum e mais confuso de depurar. Confira.

---

## Passo 6 — Salvar e ativar

1. **Save** (ou `Ctrl+S` / `Cmd+S`)
2. Ligue a chave **Inactive → Active** no canto superior direito
3. Confirme na caixa de diálogo

Ao ativar, o n8n registra o webhook no Telegram automaticamente.

---

## Passo 7 — O primeiro teste

Abra o Telegram, vá no seu bot e mande:

```
oi
```

Em 2 a 5 segundos deve chegar algo como:

> Oi, Kennedy! Aqui é a Bia, da CleanMax 😊
> Me conta: você tá procurando produto pra casa ou pro seu negócio?

**Funcionou? Parabéns, seu agente está no ar!** 🎉

Agora teste o catálogo:

```
tem algo pra tirar mofo do box?
```

Ela deve responder com o **Removedor de Limo e Mofo, R$ 17,90** — informação
que veio da sua planilha, não da cabeça do modelo.

---

## 🩺 Não funcionou? Diagnóstico rápido

Vá em **Executions** (menu lateral) e veja se apareceu alguma execução.

### Nenhuma execução aparece

O problema está **antes** do n8n — a mensagem nem chegou.

- [ ] O fluxo está **Active**?
- [ ] `https://seu-dominio.ngrok-free.dev` abre no navegador?
- [ ] `docker compose ps` mostra o ngrok `Up`?
- [ ] Confira o webhook: `https://api.telegram.org/bot<TOKEN>/getWebhookInfo`

### A execução aparece mas falha em vermelho

Clique nela e veja **qual nó** ficou vermelho:

| Nó vermelho | Causa provável |
|---|---|
| `consultar_catalogo` | Planilha não compartilhada, ID errado ou aba com nome errado |
| `Gemini 2.5 Flash` | API key inválida ou limite `429` estourado |
| `registrar_pedido` | Faltou permissão de **Editor** no compartilhamento |
| `Responder no Telegram` | Token do bot errado |

### A execução é verde mas nada chega no Telegram

Abra o nó **Responder no Telegram** e veja o output. Se o campo `text` estiver
vazio, o agente não gerou resposta — geralmente porque estourou as iterações
tentando usar uma ferramenta quebrada.

📖 Guia de testes completo em [07](07-entendendo-o-fluxo.md) e [08](08-testes.md).

---

## ✅ Antes de seguir, você deve ter

- [ ] Fluxo importado e renomeado
- [ ] 7 nós com credenciais ligadas (sem ⚠️ vermelho)
- [ ] ID da planilha colado nos 2 nós de Sheets
- [ ] Campo **Key** da memória com `{{ $json.sessionId }}`
- [ ] Fluxo **Active**
- [ ] Bot respondendo "oi" no Telegram

---

[⬅️ Anterior: Gemini](05-gemini.md) · [Índice](../README.md) · [Próximo: Entendendo o fluxo ➡️](07-entendendo-o-fluxo.md)
