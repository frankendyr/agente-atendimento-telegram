# 00 — 📦 Como baixar os arquivos do projeto

[Índice](../README.md) · [Próximo: Arquitetura ➡️](01-arquitetura.md)

---

## 🎯 Primeiro, a boa notícia

> **Você NÃO precisa de conta no GitHub para baixar este projeto.**

O repositório é público. Isso significa que qualquer pessoa, sem login e sem
cadastro, consegue baixar tudo.

Muita gente confunde duas coisas diferentes:

| | Precisa de conta no GitHub? |
|---|---|
| **Baixar** um projeto público | ❌ Não |
| **Publicar** um projeto seu | ✅ Sim |

Nesta aula você só vai **baixar**. Conta no GitHub fica para depois, se você
quiser publicar o seu próprio agente.

---

## Escolha o seu caminho

| Caminho | Para quem | Precisa instalar algo? |
|---------|-----------|------------------------|
| **A — Baixar o ZIP** ⭐ | Todo mundo. É o mais simples. | Nada |
| **B — Git clone** | Quem já usa git no dia a dia | Git |
| **C — Sem baixar nada** | Quem não pode instalar/baixar arquivos | Nada |

**Na dúvida, vá pelo caminho A.**

---

# 🅰️ Caminho A — Baixar o ZIP (recomendado)

Funciona em qualquer computador, sem conta e sem instalar nada além do Docker.

### Passo 1 — Abrir o repositório

Acesse no navegador:

**https://github.com/frankendyr/agente-atendimento-telegram**

### Passo 2 — Baixar

1. Clique no botão verde **`< > Code`** (fica acima da lista de arquivos)
2. No menu que abrir, clique em **Download ZIP**

```
┌──────────────────────────────┐
│  < > Code  ▾                 │  ← 1. clique aqui
├──────────────────────────────┤
│  🔗 Clone                     │
│     HTTPS  SSH  GitHub CLI   │
│                              │
│  📦 Download ZIP    ← 2. aqui│
└──────────────────────────────┘
```

> 💡 **Atalho:** se preferir, cole isto direto no navegador e o download começa
> sozinho:
> `https://github.com/frankendyr/agente-atendimento-telegram/archive/refs/heads/main.zip`

### Passo 3 — Descompactar

Vai baixar um arquivo chamado `agente-atendimento-telegram-main.zip`.

| Sistema | Como fazer |
|---------|-----------|
| **Windows** | Botão direito → **Extrair tudo...** → escolha a pasta → **Extrair** |
| **Mac** | Duplo clique no arquivo. Pronto, ele extrai sozinho. |
| **Linux** | Botão direito → **Extrair aqui** |

> ⚠️ **Windows:** não pule a extração! Se você abrir o ZIP com duplo clique e
> trabalhar "dentro" dele, o Docker não enxerga os arquivos. Precisa
> **extrair de verdade** para uma pasta.

### Passo 4 — Colocar num lugar fácil de achar

Mova a pasta extraída para um caminho **curto e sem acentos ou espaços**:

| | Windows | Mac / Linux |
|---|---|---|
| ✅ Bom | `C:\projetos\agente-vendas` | `~/projetos/agente-vendas` |
| ❌ Evite | `C:\Users\João\Área de Trabalho\Nova pasta (2)` | `~/Meus Projetos/Aula nº3` |

Acentos, espaços e parênteses no caminho causam erros bem chatos de diagnosticar
no Docker.

### Passo 5 — Abrir o terminal nessa pasta

| Sistema | Como fazer |
|---------|-----------|
| **Windows** | Abra a pasta no Explorer, clique na barra de endereço, digite `cmd` e dê Enter |
| **Mac** | Botão direito na pasta → **Novos Serviços** → **Novo Terminal na Pasta** |
| **Qualquer um** | Abra o terminal e use `cd` até a pasta |

Confirme que você está no lugar certo:

```bash
ls
```

(No Windows use `dir`.) Você deve ver `docker-compose.yml`, `docs`, `workflows`...

**✅ Pronto. Siga para o [doc 02 — Instalação](02-instalacao.md).**

---

# 🅱️ Caminho B — Git clone

Só use se você **já tem git instalado**. Confira:

```bash
git --version
```

Se aparecer a versão, pode clonar:

```bash
git clone https://github.com/frankendyr/agente-atendimento-telegram.git
```

```bash
cd agente-atendimento-telegram
```

> 💡 Repare: **também não pede login.** Clonar repositório público é livre.
> Só na hora de dar `git push` é que o GitHub pede autenticação — e você não vai
> precisar disso nesta aula.

**Vantagem do clone:** se eu corrigir algo no repositório, você atualiza com um
comando só:

```bash
git pull
```

Quem baixou o ZIP precisa baixar de novo.

**✅ Siga para o [doc 02 — Instalação](02-instalacao.md).**

---

# 🅲 Caminho C — Sem baixar nada

Serve para quem está num computador com restrição de download (laboratório da
faculdade, máquina do trabalho) ou só quer entender o que cada arquivo faz.

A ideia: você **cria os 2 arquivos essenciais na mão** e puxa o resto direto
pela internet.

### Passo 1 — Criar a pasta

Crie uma pasta chamada `agente-vendas` onde preferir e abra o terminal nela.

### Passo 2 — Criar o `docker-compose.yml`

Crie um arquivo com esse nome exato e cole o conteúdo de:
[docker-compose.yml](../docker-compose.yml)

> ⚠️ **Windows + Bloco de Notas:** na hora de salvar, escolha
> **Tipo: Todos os arquivos** e escreva o nome com a extensão
> `docker-compose.yml`. Senão ele salva como `docker-compose.yml.txt` e o Docker
> não encontra.

> ⚠️ **YAML é sensível a espaços.** A indentação usa **espaços, nunca Tab**.
> Copie e cole exatamente como está, sem "arrumar" o alinhamento.

### Passo 3 — Criar o `.env`

Crie outro arquivo chamado exatamente `.env` (com o ponto na frente, sem
extensão nenhuma) com este conteúdo:

```env
NGROK_AUTHTOKEN=cole_seu_authtoken_aqui
NGROK_DOMAIN=seu-dominio.ngrok-free.dev

POSTGRES_USER=n8n
POSTGRES_PASSWORD=troque_esta_senha
POSTGRES_DB=n8n_memoria
```

Só isso. Esses **dois arquivos** já sobem o ambiente inteiro.

### Passo 4 — Importar o fluxo direto da internet 🪄

Aqui está o truque que dispensa baixar o JSON:

1. No n8n, crie um workflow novo
2. Clique nos **três pontinhos `⋯`** do canto superior direito
3. Escolha **Import from URL...**
4. Cole este endereço:

```
https://raw.githubusercontent.com/frankendyr/agente-atendimento-telegram/main/workflows/assistente-vendas.json
```

5. **Import**

O fluxo aparece completo, com as anotações. Zero download. 🎉

### Passo 5 — Levar o catálogo para o Google Sheets

1. Abra o CSV no navegador:
   https://raw.githubusercontent.com/frankendyr/agente-atendimento-telegram/main/planilha/Produtos.csv
2. Selecione tudo (`Ctrl+A` / `Cmd+A`) e copie
3. No Google Sheets, clique na célula **A1** da aba `Produtos` e cole
4. Vai colar tudo numa coluna só. Para separar:
   **Dados** → **Dividir texto em colunas** → escolha **Vírgula**

### Passo 6 — Ler a documentação

Os documentos `docs/01` a `docs/10` você lê direto no site do GitHub, sem baixar.
Cada arquivo `.md` abre formatadinho no navegador.

**✅ Siga para o [doc 02 — Instalação](02-instalacao.md).**

---

## 📋 Comparando os três caminhos

| | A — ZIP | B — Git | C — Manual |
|---|---|---|---|
| Precisa de conta GitHub | ❌ | ❌ | ❌ |
| Precisa instalar git | ❌ | ✅ | ❌ |
| Precisa baixar arquivo | ✅ | ✅ | ❌ |
| Recebe atualizações fácil | ❌ | ✅ `git pull` | ❌ |
| Tempo | ~2 min | ~1 min | ~10 min |
| Dificuldade | 🟢 Fácil | 🟢 Fácil | 🟡 Média |

---

## 🩺 Problemas comuns

### "Não acho o botão Code"

Você pode estar olhando um arquivo específico em vez da página inicial do
repositório. Clique no **nome do repositório** lá no topo para voltar.

### O terminal diz "no such file or directory" no `docker compose up`

Você não está na pasta certa. Rode `ls` (ou `dir` no Windows) e confirme que
o `docker-compose.yml` aparece na lista. Se não aparecer, use `cd` até a pasta
correta.

### Baixei mas a pasta tem outra pasta dentro

Normal — o ZIP do GitHub cria uma pasta `agente-atendimento-telegram-main` e
dentro dela é que ficam os arquivos. Entre nela:

```bash
cd agente-atendimento-telegram-main
```

### Windows: o arquivo virou `.env.txt`

O Bloco de Notas adiciona `.txt` sozinho. Duas saídas:

- Salve escolhendo **Tipo: Todos os arquivos** e digite `.env` com aspas: `".env"`
- Ou use um editor melhor, como o [VS Code](https://code.visualstudio.com/) (grátis)

Para conferir se deu certo, habilite a exibição de extensões:
Explorer → aba **Exibir** → marque **Extensões de nomes de arquivos**.

---

## ✅ Antes de seguir, você deve ter

- [ ] Uma pasta no seu computador com o `docker-compose.yml` dentro
- [ ] O terminal aberto **dentro** dessa pasta
- [ ] `ls` (ou `dir`) mostrando os arquivos do projeto

---

[Índice](../README.md) · [Próximo: Arquitetura ➡️](01-arquitetura.md)
