# 02 — 🐳 Subindo o n8n com Docker + ngrok

[⬅️ Anterior: Arquitetura](01-arquitetura.md) · [Índice](../README.md) · [Próximo: Telegram ➡️](03-telegram.md)

---

## Antes de começar

> 📦 Ainda não baixou os arquivos do projeto? Comece pelo
> **[doc 00 — Como baixar](00-como-baixar.md)**. Não precisa de conta no GitHub.

Instale o **Docker Desktop**: https://www.docker.com/products/docker-desktop/

Abra o app e espere o ícone da baleia 🐳 ficar verde/estável. Depois confirme
no terminal:

```bash
docker --version
```

Se aparecer algo como `Docker version 27.x`, está pronto.

---

## Passo 1 — Criar a conta no ngrok

1. Acesse https://dashboard.ngrok.com/signup e crie uma conta grátis.

2. **Pegue o authtoken** em
   https://dashboard.ngrok.com/get-started/your-authtoken
   → copie o valor (algo como `2xYz...`)

3. **Crie um domínio fixo** em https://dashboard.ngrok.com/domains
   → clique em **+ New Domain** → o plano grátis dá 1 domínio
   → você recebe algo como `meu-agente-vendas.ngrok-free.dev`

> ⚠️ **O erro nº 1 da aula:** confundir o *authtoken* com o *domínio*.
>
> - **Authtoken** = sua senha (`2xYz3aBc...`) → vai em `NGROK_AUTHTOKEN`
> - **Domínio** = seu endereço (`algo.ngrok-free.dev`) → vai em `NGROK_DOMAIN`

---

## Passo 2 — Configurar o arquivo `.env`

Dentro da pasta do projeto:

```bash
cp .env.example .env
```

Abra o `.env` no seu editor e preencha:

```env
NGROK_AUTHTOKEN=2xYz3aBcSeuTokenRealAqui
NGROK_DOMAIN=meu-agente-vendas.ngrok-free.dev
```

### ✅ Regras do `NGROK_DOMAIN`

| | Exemplo |
|---|---|
| ✅ Certo | `meu-agente-vendas.ngrok-free.dev` |
| ❌ Errado | `https://meu-agente-vendas.ngrok-free.dev` |
| ❌ Errado | `meu-agente-vendas.ngrok-free.dev/` |

**Sem `https://` e sem barra no final.**

---

## Passo 3 — Subir os containers

```bash
docker compose up -d
```

O `-d` significa *detached*: roda em segundo plano e devolve seu terminal.

Na primeira vez o Docker vai baixar as imagens (alguns minutos). Depois disso é
instantâneo.

### Conferir se subiu

```bash
docker compose ps
```

Você deve ver os dois containers com status `Up`:

```
NAME            IMAGE                       STATUS
n8n_vendas      docker.n8n.io/n8nio/n8n     Up 30 seconds
ngrok_vendas    ngrok/ngrok:latest          Up 28 seconds
```

---

## Passo 4 — Testar os dois endereços

### A) O n8n local

Abra no navegador: **http://localhost:5678**

Na primeira vez o n8n pede para criar um usuário. Preencha e **anote a senha** —
é uma conta local, não dá para recuperar por e-mail.

### B) A URL pública do ngrok

Abra: **https://SEU-DOMINIO.ngrok-free.dev**

Deve aparecer a **mesma tela do n8n**. Se aparecer, o túnel está funcionando. 🎉

> Se aparecer uma tela de aviso do ngrok ("You are about to visit..."), clique
> em **Visit Site**. Isso é normal no plano grátis e não afeta o Telegram.

---

## 🔧 Comandos que você vai usar sempre

```bash
docker compose up -d
```
Sobe tudo.

```bash
docker compose down
```
Para tudo (os fluxos ficam salvos no volume).

```bash
docker compose restart n8n
```
Reinicia só o n8n — use depois de mudar variáveis de ambiente.

```bash
docker compose logs -f n8n
```
Vê os logs do n8n em tempo real. `Ctrl+C` para sair.

```bash
docker compose logs ngrok
```
Vê os logs do túnel — útil quando o Telegram não chega.

---

## 🩺 Problemas comuns

### "port is already allocated"

Já tem algo na porta 5678 (provavelmente outro n8n de uma aula anterior).

```bash
docker ps
```

Ache o container antigo e pare:

```bash
docker stop nome_do_container_antigo
```

### O ngrok reinicia sem parar

Quase sempre é o `.env`. Veja o motivo:

```bash
docker compose logs ngrok
```

| Mensagem no log | Causa |
|---|---|
| `authentication failed` | Authtoken errado ou com espaço sobrando |
| `domain not found` | O domínio não foi criado no painel do ngrok |
| `is already bound` | O mesmo domínio já está em uso em outro lugar |

### `https://seu-dominio` não abre, mas `localhost:5678` abre

O túnel não conectou. Confirme que:
- `NGROK_DOMAIN` está **sem** `https://`
- o container do ngrok está `Up` em `docker compose ps`

Depois:

```bash
docker compose down && docker compose up -d
```

### Mudei o `.env` e nada mudou

Variáveis de ambiente só são lidas quando o container **nasce**. `restart` não
basta:

```bash
docker compose down && docker compose up -d
```

---

## ✅ Antes de seguir, você deve ter

- [ ] `docker compose ps` mostrando 2 containers `Up`
- [ ] `http://localhost:5678` abrindo o n8n
- [ ] `https://seu-dominio.ngrok-free.dev` abrindo o **mesmo** n8n
- [ ] Usuário e senha do n8n criados e anotados

---

[⬅️ Anterior: Arquitetura](01-arquitetura.md) · [Índice](../README.md) · [Próximo: Telegram ➡️](03-telegram.md)
