# 04 — 📊 Montando a planilha de produtos

[⬅️ Anterior: Telegram](03-telegram.md) · [Índice](../README.md) · [Próximo: Gemini ➡️](05-gemini.md)

---

Esta é a **base de conhecimento** do agente. Tudo que a Bia sabe sobre produto
vem daqui. Se estiver errado aqui, ela erra com o cliente.

---

## Passo 1 — Criar a planilha

1. Acesse https://sheets.google.com e crie uma planilha em branco
2. Renomeie para **Assistente de Vendas CleanMax**

### Criar as duas abas

Lá embaixo, nas abas:

1. Renomeie a `Página1` para **`Produtos`**
2. Clique no `+` e crie outra aba chamada **`Pedidos`**

> ⚠️ Os nomes precisam ser **exatamente** `Produtos` e `Pedidos` — com P
> maiúsculo, sem acento e sem espaço. O fluxo procura por esses nomes.

---

## Passo 2 — Importar o catálogo

Na aba **`Produtos`**:

1. Menu **Arquivo** → **Importar** → aba **Fazer upload**
2. Escolha o arquivo `planilha/Produtos.csv` deste projeto
3. Em **Importar local**, escolha **Substituir planilha atual**
4. Em **Tipo de separador**, escolha **Vírgula**
5. Clique em **Importar dados**

Você deve ficar com 20 produtos e estas colunas:

| Coluna | Para que serve |
|--------|----------------|
| `SKU` | Código do produto |
| `Produto` | Nome comercial |
| `Categoria` | Cozinha, Lavanderia, Pisos... |
| `Descricao` | O que o produto é |
| `Indicacoes` | **Para que serve** — é o que a IA usa para recomendar |
| `Modo_de_Uso` | Como usar — responde as dúvidas do cliente |
| `Rendimento` | Quantas aplicações — **arma para quebrar a objeção "tá caro"** |
| `Tamanho` | 500ml, 1L, kit... |
| `Preco` | Valor em reais |
| `Estoque` | Quantidade disponível |

### Por que essas colunas exatamente?

Cada coluna existe por um motivo de **venda**, não de banco de dados:

```
Cliente: "esse desengordurante serve pro meu fogão?"
         → a Bia responde com a coluna Indicacoes

Cliente: "como eu uso?"
         → a Bia responde com Modo_de_Uso

Cliente: "tá caro"
         → a Bia usa Rendimento para calcular o custo por uso

Cliente: "quero 3"
         → a Bia checa Estoque antes de confirmar
```

> 💡 Repare que o CSV está **sem acentos** de propósito nos nomes das colunas.
> Isso evita um monte de dor de cabeça com expressões no n8n
> (`$json.Descricao` funciona; `$json['Descrição']` precisa de colchetes).

---

## Passo 3 — Preparar a aba de Pedidos

Na aba **`Pedidos`**, digite estes 7 títulos na **linha 1**, uma coluna cada:

| A | B | C | D | E | F | G |
|---|---|---|---|---|---|---|
| Data | Cliente | Contato | Produtos | Valor_Total | Forma_Pagamento | Observacoes |

Deixe o resto vazio — o agente vai preencher.

> ⚠️ Escreva exatamente assim, com `_` no lugar dos espaços. O fluxo mapeia
> essas colunas pelo nome.

---

## Passo 4 — Pegar o ID da planilha

Olhe a URL da sua planilha:

```
https://docs.google.com/spreadsheets/d/1AbC2DeFgHiJkLmNoPqRsTuVwXyZ_1234567890/edit#gid=0
                                       └──────────── isto é o ID ────────────┘
```

**Copie o trecho entre `/d/` e `/edit`.** Guarde — você vai colar no fluxo no
próximo doc.

---

## Passo 5 — Criar a Service Account (a parte chata, mas é uma vez só)

Vamos dar ao n8n uma "identidade robô" que pode ler e escrever na planilha.
É o mesmo método usado nos assistentes Financeiro e Nutricional.

### 5.1 — Criar o projeto no Google Cloud

1. Acesse https://console.cloud.google.com/
2. No topo, clique no seletor de projeto → **Novo projeto**
3. Nome: `n8n-vendas` → **Criar**
4. **Selecione esse projeto** no seletor do topo antes de continuar

### 5.2 — Ativar as duas APIs

Ative as duas, uma de cada vez:

- **Google Sheets API** → https://console.cloud.google.com/apis/library/sheets.googleapis.com
- **Google Drive API** → https://console.cloud.google.com/apis/library/drive.googleapis.com

Em cada uma, clique em **Ativar**.

> A API do Drive é necessária para o n8n conseguir *localizar* o arquivo, não só
> ler o conteúdo. Esquecer dela causa erro de permissão depois.

### 5.3 — Criar a conta de serviço

1. Vá em **APIs e serviços** → **Credenciais**
2. **+ Criar credenciais** → **Conta de serviço**
3. Nome: `n8n-sheets` → **Criar e continuar** → **Concluir**

### 5.4 — Gerar a chave JSON

1. Clique na conta de serviço recém-criada
2. Aba **CHAVES** → **Adicionar chave** → **Criar nova chave**
3. Escolha **JSON** → **Criar**

Um arquivo `.json` será baixado. **Ele contém a chave privada — trate como
senha.** Não coloque na pasta do projeto e nunca suba no GitHub.

### 5.5 — Copiar os dois valores

Abra o arquivo JSON baixado. Você vai usar dois campos:

```json
{
  "client_email": "n8n-sheets@n8n-vendas.iam.gserviceaccount.com",
  "private_key": "-----BEGIN PRIVATE KEY-----\nMIIEv...\n-----END PRIVATE KEY-----\n"
}
```

---

## Passo 6 — 🔑 Compartilhar a planilha com o robô

**Este é o passo que todo mundo esquece.** A conta de serviço é um "usuário"
separado — ela não enxerga suas planilhas por padrão.

1. Volte para a planilha no Google Sheets
2. Clique em **Compartilhar** (botão azul, canto superior direito)
3. Cole o `client_email` da conta de serviço
4. Dê permissão de **Editor** (precisa escrever na aba Pedidos!)
5. Desmarque "Notificar pessoas" e clique em **Compartilhar**

```
❌ Sem esse passo:  "The caller does not have permission"
✅ Com esse passo:  funciona
```

---

## Passo 7 — Cadastrar a credencial no n8n

1. No n8n: **Credentials** → **Add credential**
2. Busque por **Google Service Account API**
3. Preencha:

| Campo | Valor |
|-------|-------|
| **Service Account Email** | o `client_email` do JSON |
| **Private Key** | o `private_key` **inteiro**, incluindo as linhas `-----BEGIN...` e `-----END...` |

4. Nome da credencial: `Google Sheets Vendas`
5. **Save**

> ⚠️ Copie a private key completa, com as quebras de linha. Se você copiar só o
> meio ou cortar o `-----BEGIN PRIVATE KEY-----`, dá erro de autenticação.

---

## 🩺 Problemas comuns

| Erro | Causa | Solução |
|------|-------|---------|
| `The caller does not have permission` | Não compartilhou a planilha | Passo 6 |
| `Requested entity was not found` | ID da planilha errado | Confira o Passo 4 |
| `Google Sheets API has not been used` | API não ativada | Passo 5.2 |
| `error:1E08010C:DECODER routines` | Private key incompleta | Recopie a chave inteira |
| `Unable to sign without exp` | Relógio do PC desajustado | Sincronize o horário |
| Agente diz que não achou produtos | Nome da aba errado | Precisa ser `Produtos` |

---

## ✅ Antes de seguir, você deve ter

- [ ] Planilha com as abas `Produtos` (20 linhas) e `Pedidos` (só cabeçalho)
- [ ] O **ID da planilha** copiado
- [ ] APIs do Sheets **e** do Drive ativadas
- [ ] Planilha compartilhada como **Editor** com o e-mail da conta de serviço
- [ ] Credencial `Google Sheets Vendas` salva no n8n

---

[⬅️ Anterior: Telegram](03-telegram.md) · [Índice](../README.md) · [Próximo: Gemini ➡️](05-gemini.md)
