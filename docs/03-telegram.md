# 03 — 🤖 Criando o bot no Telegram

[⬅️ Anterior: Instalação](02-instalacao.md) · [Índice](../README.md) · [Próximo: Google Sheets ➡️](04-google-sheets.md)

---

## Passo 1 — Falar com o BotFather

O Telegram tem um bot oficial que cria outros bots. O nome dele é **BotFather**.

1. Abra o Telegram e busque por **@BotFather**
   → confirme que tem o **selo azul de verificado** ✅
2. Clique em **Iniciar** / **Start**
3. Envie:

```
/newbot
```

4. Ele pergunta o **nome de exibição** (pode ter espaço e acento):

```
CleanMax Vendas
```

5. Ele pergunta o **username**, que precisa ser único no mundo e terminar em
   `bot`:

```
cleanmax_vendas_kennedy_bot
```

> 💡 Se der "Sorry, this username is already taken", acrescente seu nome ou
> alguns números no final.

6. Pronto! Ele responde com o seu **token**:

```
Use this token to access the HTTP API:
8123456789:AAHdqTcvbXXXXXXXXXXXXXXXXXXXXXXXXXX
```

**Copie esse token.** É a senha do seu bot.

> ⚠️ Nunca coloque esse token no GitHub. Se vazar, qualquer pessoa controla seu
> bot. Se acontecer, use `/revoke` no BotFather para gerar outro.

---

## Passo 2 — Cadastrar a credencial no n8n

1. No n8n, no menu lateral, vá em **Credentials** → **Add credential**
2. Busque por **Telegram** e escolha **Telegram API**
3. Cole o token no campo **Access Token**
4. Dê um nome fácil de achar depois, como `Telegram Vendas`
5. **Save**

Se aparecer ✅ *Connection tested successfully*, está tudo certo.

---

## Passo 3 — Encontrar o seu bot

No Telegram, busque pelo username que você criou
(ex.: `@cleanmax_vendas_kennedy_bot`) e clique em **Iniciar**.

Ele ainda não vai responder — o fluxo no n8n nem existe. Isso é esperado.

---

## 🧠 Como o Telegram entrega as mensagens (importante entender)

Existem duas formas de um bot receber mensagens:

| Modo | Como funciona | Onde usamos |
|------|---------------|-------------|
| **Polling** | O bot pergunta de tempos em tempos: "chegou algo?" | Não usamos |
| **Webhook** | O Telegram *empurra* a mensagem assim que ela chega | ✅ É o nosso |

O n8n usa **webhook**. Quando você **ativa** o fluxo, ele avisa o Telegram:

> "Ei Telegram, quando chegar mensagem para esse bot, manda para
> `https://seu-dominio.ngrok-free.dev/webhook/...`"

Por isso o ngrok é obrigatório: sem uma URL pública, o Telegram não tem para
onde empurrar.

```
Cliente digita ──▶ Servidores do Telegram ──▶ POST na sua URL ngrok ──▶ n8n
```

### 3 consequências práticas disso

1. **O fluxo precisa estar ATIVO** (botão *Active* ligado) para o webhook ser
   registrado. Em modo de teste, ele só escuta enquanto você clica em
   "Test workflow".
2. **Se o `NGROK_DOMAIN` mudar**, o webhook antigo aponta para o vazio. Solução:
   desativar e reativar o fluxo.
3. **Um bot só pode ter um webhook por vez.** Se você usar o mesmo token em dois
   fluxos, um vai roubar o outro. Crie um bot novo para cada projeto.

---

## 🩺 Problemas comuns

### O bot não responde nada

Confira nesta ordem:

1. O fluxo está **Active**? (canto superior direito do n8n)
2. `https://seu-dominio.ngrok-free.dev` abre o n8n no navegador?
3. Você usou esse token em outro fluxo? Desative o outro.

### Ver o que o Telegram acha do seu webhook

Cole no navegador, trocando pelo seu token:

```
https://api.telegram.org/bot<SEU_TOKEN>/getWebhookInfo
```

Você deve ver a sua URL do ngrok em `"url"`.

| O que aparece | O que significa |
|---|---|
| `"url": ""` | Nenhum webhook registrado → ative o fluxo no n8n |
| `"url"` com um domínio antigo | Sobrou de outro projeto → desative e reative o fluxo |
| `"last_error_message"` preenchido | O Telegram tentou e falhou → leia a mensagem |

### Zerar o webhook (quando tudo trava)

```
https://api.telegram.org/bot<SEU_TOKEN>/deleteWebhook
```

Depois desative e reative o fluxo no n8n.

---

## ✅ Antes de seguir, você deve ter

- [ ] Um bot criado no BotFather
- [ ] O token guardado em local seguro
- [ ] A credencial `Telegram Vendas` salva no n8n
- [ ] O bot iniciado no seu Telegram

---

[⬅️ Anterior: Instalação](02-instalacao.md) · [Índice](../README.md) · [Próximo: Google Sheets ➡️](04-google-sheets.md)
