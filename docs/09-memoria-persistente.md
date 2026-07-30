# 09 — 💾 Bônus: memória persistente com Postgres

[⬅️ Anterior: Testes](08-testes.md) · [Índice](../README.md) · [Próximo: Exercícios ➡️](10-exercicios.md)

---

## O problema

A **Simple Memory** que usamos guarda a conversa na **RAM do n8n**. Ou seja:

```bash
docker compose restart n8n
```

...e todo mundo virou estranho de novo. 😅

Numa aula isso é ótimo (rápido, zero configuração). Num negócio de verdade, é
inaceitável: o cliente volta no dia seguinte e a Bia não lembra que ele já tinha
escolhido dois produtos.

---

## A solução

Trocar a memória por **Postgres Chat Memory** — o histórico vai para um banco de
dados de verdade, em disco.

| | Simple Memory | Postgres Chat Memory |
|---|---|---|
| Onde guarda | RAM | Disco (banco) |
| Sobrevive a restart | ❌ | ✅ |
| Configuração | Zero | Precisa do banco |
| Dá para consultar depois | ❌ | ✅ (é uma tabela SQL) |
| Bom para | Aula, protótipo | Produção |

E tem um bônus: como fica numa tabela, você consegue **ler todas as conversas
com SQL**. Vira material de análise: quais objeções aparecem mais, quais
produtos são mais perguntados, onde as vendas travam.

---

## Passo 1 — Subir o Postgres

O `docker-compose.yml` já tem o serviço pronto, desligado por um *profile*.

Confirme que o `.env` tem estas três linhas preenchidas:

```env
POSTGRES_USER=n8n
POSTGRES_PASSWORD=uma_senha_boa_aqui
POSTGRES_DB=n8n_memoria
```

Agora suba com o profile ligado:

```bash
docker compose --profile persistente up -d
```

Confira que os **três** containers estão de pé:

```bash
docker compose ps
```

---

## Passo 2 — Criar a credencial no n8n

1. **Credentials** → **Add credential** → busque **Postgres**
2. Preencha:

| Campo | Valor | ⚠️ Atenção |
|-------|-------|------------|
| **Host** | `postgres` | **Não** é `localhost`! |
| **Database** | `n8n_memoria` | o mesmo do `.env` |
| **User** | `n8n` | o mesmo do `.env` |
| **Password** | a senha do `.env` | |
| **Port** | `5432` | |
| **SSL** | `Disable` | é rede interna do Docker |

3. Nome: `Postgres Memoria` → **Save**

### 🤔 Por que `postgres` e não `localhost`?

Este é um conceito importante de Docker.

Cada container é uma máquina isolada. Para o container do n8n, `localhost`
significa **ele mesmo** — e não tem banco nenhum lá dentro.

O Docker Compose cria uma rede interna onde **o nome do serviço vira o
endereço**:

```
Do seu computador  →  localhost:5432   ✅ (por causa do ports:)
De dentro do n8n   →  postgres:5432    ✅ (nome do serviço)
De dentro do n8n   →  localhost:5432   ❌ (é o próprio n8n)
```

É o mesmo motivo do `command: http --domain=... n8n_vendas:5678` no ngrok: ele
chama o n8n pelo **nome do container**, não por localhost.

---

## Passo 3 — Trocar o nó de memória

1. Abra o fluxo `Assistente de Vendas`
2. **Delete** o nó `Memoria da Conversa`
3. Clique no `+` do conector **Memory** embaixo do agente
4. Escolha **Postgres Chat Memory**
5. Configure:

| Campo | Valor |
|-------|-------|
| **Credential** | `Postgres Memoria` |
| **Session ID** | `Define below` |
| **Key** | `{{ $json.sessionId }}` |
| **Table Name** | `n8n_chat_histories` |

6. **Save** e mantenha o fluxo **Active**

A tabela é criada sozinha na primeira mensagem. 🎉

---

## Passo 4 — Provar que funcionou

Este é o teste que fecha a aula com chave de ouro:

```
1. Converse com o bot: "quero algo pra cozinha"
2. Ele responde com o desengordurante
3. No terminal:  docker compose restart n8n
4. Espere ~20 segundos
5. Mande:  "e aquele produto que você falou, quanto era mesmo?"
```

**Antes (Simple Memory):** "Desculpa, qual produto?" 🤡
**Agora (Postgres):** "O Desengordurante Cozinha Pesada, R$ 14,90 😊" ✅

---

## Passo 5 — Bisbilhotar as conversas com SQL

Aqui está o bônus que ninguém espera. Entre no banco:

```bash
docker compose exec postgres psql -U n8n -d n8n_memoria
```

Veja as últimas mensagens:

```sql
SELECT session_id, message FROM n8n_chat_histories ORDER BY id DESC LIMIT 20;
```

Conte quantas mensagens cada cliente trocou:

```sql
SELECT session_id, COUNT(*) AS total
FROM n8n_chat_histories
GROUP BY session_id
ORDER BY total DESC;
```

Ache quem falou em preço (potenciais objeções de valor):

```sql
SELECT session_id, message
FROM n8n_chat_histories
WHERE message::text ILIKE '%caro%';
```

Sair: `\q`

> 💡 **Ideia de projeto avançado:** um segundo fluxo no n8n que roda toda noite,
> lê essa tabela, manda para o Gemini analisar e envia no seu Telegram um resumo:
> *"Hoje: 12 conversas, 3 pedidos fechados, objeção mais comum: preço do kit."*

---

## 🩺 Problemas comuns

| Erro | Causa | Solução |
|------|-------|---------|
| `ECONNREFUSED 127.0.0.1:5432` | Host está como `localhost` | Troque para `postgres` |
| `password authentication failed` | Senha diferente do `.env` | Confira os dois |
| `database "n8n_memoria" does not exist` | Banco criado antes com outro nome | `docker compose down -v` e suba de novo (apaga os dados!) |
| Postgres não sobe | Esqueceu o profile | Use `--profile persistente` |

---

## ⚠️ Sobre o `docker compose down -v`

O `-v` apaga os **volumes** — ou seja, o banco **e os seus fluxos do n8n**.

Use `docker compose down` (sem `-v`) no dia a dia. Só use `-v` quando quiser
mesmo zerar tudo e começar do princípio.

---

[⬅️ Anterior: Testes](08-testes.md) · [Índice](../README.md) · [Próximo: Exercícios ➡️](10-exercicios.md)
