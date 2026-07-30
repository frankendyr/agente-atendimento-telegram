# 🧼 Assistente de Vendas com IA — n8n + Telegram + Gemini

Um agente de IA que **vende produtos de limpeza pelo Telegram**.

Ele conduz a conversa como um vendedor de verdade: entende a necessidade do
cliente, consulta o catálogo real na planilha, recomenda produtos, responde
dúvidas, contorna objeções, fecha o pedido e registra tudo — **lembrando de
toda a conversa anterior**.

> 📚 Projeto didático da aula de **Agentes de IA com n8n** — Digital College.

---

## 🎬 Como fica na prática

```
Cliente:  Oi, boa tarde
Bia:      Oi, Kennedy! Aqui é a Bia, da CleanMax 😊
          Me conta: você tá procurando produto pra casa ou pro seu negócio?

Cliente:  Pra casa. Minha coifa tá muito engordurada
Bia:      Ah, gordura de coifa é osso mesmo!
          O Desengordurante Cozinha Pesada resolve isso — ele dissolve a
          gordura queimada em 1 minuto, sem esfregar. Sai R$ 14,90 o 500ml.
          Quer que eu já separe?

Cliente:  Tá caro né
Bia:      Entendo! Mas repara: esse frasco rende umas 60 aplicações.
          Dá menos de 25 centavos por limpeza 😉
          E olha, se você levar o Kit Completo (6 produtos) sai R$ 79,90...
```

E o pedido cai automaticamente na sua planilha do Google. 📊

---

## 🧭 Índice do passo a passo

| # | Etapa | Tempo |
|---|-------|-------|
| [00](docs/00-como-baixar.md) | 📦 **Como baixar os arquivos** (sem precisar de conta no GitHub) | 3 min |
| [01](docs/01-arquitetura.md) | 🏗️ Entendendo a arquitetura | 10 min |
| [02](docs/02-instalacao.md) | 🐳 Subindo o n8n com Docker + ngrok | 15 min |
| [03](docs/03-telegram.md) | 🤖 Criando o bot no Telegram | 5 min |
| [04](docs/04-google-sheets.md) | 📊 Montando a planilha de produtos | 15 min |
| [05](docs/05-gemini.md) | 🧠 Pegando a chave do Google Gemini | 5 min |
| [06](docs/06-importar-fluxo.md) | 📥 Importando e configurando o fluxo | 15 min |
| [07](docs/07-entendendo-o-fluxo.md) | 🔍 Entendendo cada nó do fluxo | 20 min |
| [08](docs/08-testes.md) | ✅ Testando e depurando | 15 min |
| [09](docs/09-memoria-persistente.md) | 💾 Bônus: memória persistente com Postgres | 10 min |
| [10](docs/10-exercicios.md) | 🎯 Exercícios para praticar | — |

---

## 🏗️ A arquitetura em uma imagem

```
                        ┌─────────────────────────────┐
     📱 Telegram ──────▶│         NGROK               │
        (cliente)       │  (túnel público → local)    │
                        └──────────────┬──────────────┘
                                       │
                        ┌──────────────▼──────────────┐
                        │           n8n               │
                        │      (Docker, local)        │
                        │                             │
                        │   ┌─────────────────────┐   │
                        │   │   AGENTE DE VENDAS  │   │
                        │   └──┬───┬───┬───┬──────┘   │
                        └──────┼───┼───┼───┼──────────┘
                               │   │   │   │
                 ┌─────────────┘   │   │   └─────────────┐
                 │                 │   │                 │
          🧠 Gemini 2.5     💾 Memória   📊 Catálogo   📝 Pedidos
          (raciocínio)     (contexto)   (Sheets)      (Sheets)
```

**Por que essa arquitetura?**

| Peça | Papel | Por que ela existe |
|------|-------|--------------------|
| **Docker** | Empacota tudo | Sobe o ambiente inteiro com 1 comando, igual em qualquer máquina |
| **ngrok** | Túnel público | O Telegram precisa de uma URL da internet, mas o n8n roda no seu PC |
| **n8n** | Orquestrador | Conecta tudo sem escrever código |
| **AI Agent** | Cérebro | Decide *sozinho* quando consultar o catálogo e quando registrar o pedido |
| **Gemini 2.5 Flash** | Modelo | Rápido, barato e entende áudio nativamente |
| **Memória** | Contexto | Sem ela o agente esquece tudo a cada mensagem |
| **Google Sheets** | Banco de dados | Qualquer pessoa da equipe edita preços sem saber programar |

📖 Explicação completa em **[docs/01-arquitetura.md](docs/01-arquitetura.md)**

---

## ⚡ Início rápido

> Se você é da aula e quer só rodar, siga aqui. Se quer **entender**,
> comece pelo [docs/01](docs/01-arquitetura.md).

### 1. Baixe os arquivos

> 💡 **Você não precisa de conta no GitHub para baixar.** O repositório é
> público — qualquer pessoa baixa, sem login.

**Sem git (mais simples):** clique no botão verde **`< > Code`** aqui em cima
→ **Download ZIP** → extraia a pasta.

**Com git:**

```bash
git clone https://github.com/frankendyr/agente-atendimento-telegram.git
```

📖 Passo a passo detalhado, com as três formas de baixar (inclusive **sem baixar
nada**): **[docs/00-como-baixar.md](docs/00-como-baixar.md)**

### 2. Configure

Abra o terminal **dentro da pasta do projeto** e rode:

```bash
cp .env.example .env
```

No Windows (Prompt de Comando), use:

```bash
copy .env.example .env
```

Abra o `.env` e preencha com seu token e domínio do ngrok.

### 3. Suba o ambiente

```bash
docker compose up -d
```

### 4. Acesse o n8n

Abra **http://localhost:5678** e crie sua conta local.

### 5. Conecte as contas

Siga os docs [03](docs/03-telegram.md) → [04](docs/04-google-sheets.md) →
[05](docs/05-gemini.md) → [06](docs/06-importar-fluxo.md).

### 6. Teste

Mande "oi" para o seu bot no Telegram. 🎉

---

## 📁 O que tem em cada pasta

```
assistente-vendas-n8n/
│
├── docker-compose.yml          ← a infraestrutura (n8n + ngrok + postgres)
├── .env.example                ← modelo das variáveis (copie para .env)
│
├── workflows/
│   └── assistente-vendas.json  ← o fluxo pronto para importar no n8n
│
├── planilha/
│   ├── Produtos.csv            ← catálogo com 20 produtos de limpeza
│   └── Pedidos.csv             ← cabeçalho da aba de pedidos
│
├── prompts/
│   └── system-prompt-vendedor.md  ← a técnica de vendas do agente
│
└── docs/                       ← o passo a passo completo (00 a 10)
```

---

## 🔑 O que você precisa ter antes

| Item | Onde conseguir | Custo |
|------|----------------|-------|
| Docker Desktop | [docker.com](https://www.docker.com/products/docker-desktop/) | Grátis |
| Conta ngrok + domínio fixo | [ngrok.com](https://dashboard.ngrok.com/) | Grátis |
| Conta no Telegram | App do Telegram | Grátis |
| Chave da API do Gemini | [aistudio.google.com](https://aistudio.google.com/apikey) | Grátis (com limites) |
| Conta Google (Sheets) | [sheets.google.com](https://sheets.google.com) | Grátis |

> ✅ **Conta no GitHub NÃO está na lista.** Ela só é necessária se você quiser
> publicar o *seu* projeto depois. Para baixar este aqui, não precisa.

---

## 🧠 O grande salto desta aula

Nos projetos anteriores (Assistente Financeiro e Nutricional) o fluxo era
**linear** — nós fixos, um atrás do outro:

```
Mensagem → IA extrai dados → Salva na planilha → Responde
```

O modelo só **preenchia campos**. Quem decidia o caminho era você, no Switch.

Aqui a lógica se inverte. Usamos o nó **AI Agent**:

```
Mensagem → AGENTE decide o que fazer → (consulta? registra? só responde?) → Responde
```

Agora o modelo **decide sozinho** qual ferramenta usar e quando. Você não
programa o caminho — você **descreve as ferramentas e as regras**, e o agente
escolhe.

Some isso à **memória**, e ele deixa de ser um robô de comando único para virar
alguém que **conduz uma conversa** com começo, meio e fechamento.

| | Fluxo linear (aulas anteriores) | Agente (esta aula) |
|---|---|---|
| Quem decide o caminho | Você (Switch/IF) | O modelo |
| Lembra do que foi dito | ❌ | ✅ |
| Consulta dados sob demanda | ❌ | ✅ (ferramentas) |
| Conduz uma conversa | ❌ | ✅ |

---

## ⚠️ Avisos importantes

- **Nunca suba o arquivo `.env`** para o GitHub. Ele já está no `.gitignore`.
- Se você já publicou um token por engano, **revogue e gere outro** — vale para
  ngrok, Telegram e Gemini.
- Este projeto é para **estudo local**. Para produção você ainda precisa de:
  autenticação forte no n8n, backup do banco, tratamento de erros, limite de
  requisições e um domínio de verdade (sem ngrok).

---

## 📄 Licença

Material didático livre para uso educacional.
