# 08 — ✅ Testando e depurando

[⬅️ Anterior: Entendendo o fluxo](07-entendendo-o-fluxo.md) · [Índice](../README.md) · [Próximo: Memória persistente ➡️](09-memoria-persistente.md)

---

## O roteiro de teste (use na aula, nesta ordem)

Cada teste valida **uma peça específica** da arquitetura. Se um falhar, você
sabe exatamente onde olhar.

---

### ✅ Teste 1 — O canal funciona

```
oi
```

**Esperado:** ela cumprimenta **pelo seu nome** e faz uma pergunta aberta.

**Valida:** Telegram → ngrok → n8n → Gemini → Telegram
**Se falhar:** o problema é de infraestrutura. Volte ao [doc 06](06-importar-fluxo.md).

---

### ✅ Teste 2 — A ferramenta de catálogo funciona

```
quanto custa o desengordurante de cozinha?
```

**Esperado:** **R$ 14,90** — o valor exato da sua planilha.

**Valida:** a ferramenta `consultar_catalogo`

> 🔬 **Prove em sala:** abra a planilha, mude o preço para `99,90`, salve, e
> pergunte de novo no Telegram. Ela responde R$ 99,90. **Isso prova que ela lê
> a planilha ao vivo e não decorou nada.** É o momento "aha!" da aula.

---

### ✅ Teste 3 — A memória funciona

Mande as três mensagens **separadas**:

```
1. quero algo pra limpar o box do banheiro
2. quanto custa?
3. e se eu levar 3?
```

**Esperado:** na 2ª ela sabe que "quanto custa" se refere ao produto de mofo. Na
3ª ela multiplica sem você repetir qual produto.

**Valida:** memória + `sessionId`
**Se falhar:** o campo **Key** da memória está vazio. Veja [doc 06, passo 5](06-importar-fluxo.md).

---

### ✅ Teste 4 — Ela não inventa (o teste mais importante)

```
vocês vendem ração pra cachorro?
```

**Esperado:** ela diz honestamente que não trabalha com isso e oferece algo do
catálogo (provavelmente o desinfetante, que serve para ambientes com pets).

**❌ Falhou se:** ela disser "sim, temos ração premium por R$ 89,90". Isso é
**alucinação** e significa que a regra do prompt não está sendo respeitada.

---

### ✅ Teste 5 — A técnica de vendas funciona

```
tá muito caro isso
```

**Esperado:** ela **não** dá desconto (não pode!) e **não** desiste. Ela usa o
rendimento para mostrar o custo por uso.

**Valida:** o bloco `ETAPA 5 — TRATAR OBJEÇÕES` do prompt

---

### ✅ Teste 6 — O áudio funciona

Grave um áudio no Telegram:

> "Oi, eu preciso de alguma coisa pra tirar gordura do meu fogão"

**Esperado:** ela responde normalmente, como se você tivesse digitado.

**Valida:** Baixar Audio → Transcrever → Preparar Mensagem
**Se falhar:** olhe a saída do nó `Transcrever Audio` em Executions.

---

### ✅ Teste 7 — O pedido é registrado (o teste final)

Faça a conversa completa:

```
1. quero comprar o desengordurante de cozinha
2. pode fechar
3. meu nome é Kennedy Ribeiro, telefone 85 99999-0000
```

**Esperado:**
- Ela faz um **resumo** com produto, quantidade e total
- Pede confirmação
- Depois do "sim", confirma que registrou
- **Uma linha nova aparece na aba `Pedidos`** da sua planilha 🎉

**Valida:** a ferramenta `registrar_pedido` + o `$fromAI`

---

### ✅ Teste 8 — A memória é isolada por cliente

Peça para um colega mandar mensagem para o **mesmo bot**.

**Esperado:** a Bia trata vocês dois de forma independente, sem misturar.

**❌ Falhou se:** ela responder ao seu colega falando do *seu* pedido. Isso
significa que o `sessionId` está fixo ou vazio.

---

## 🔬 Como investigar uma execução

Vá em **Executions** no menu lateral e clique numa execução.

### Ver o raciocínio do agente

Clique no nó **Agente de Vendas** e depois na aba de **logs**. Você vê a
sequência de pensamento:

```
→ Chamou consultar_catalogo
← Recebeu 20 linhas
→ Gerou a resposta final
```

Se o agente **não chamou nenhuma ferramenta** quando deveria, o problema está
na **Description** da ferramenta ou no System Message.

Se ele chamou a ferramenta **várias vezes seguidas**, ele não entendeu o
resultado — geralmente a planilha voltou vazia.

### Testar sem gastar mensagem no Telegram

1. Faça uma execução real (mande "oi" pelo Telegram)
2. Em Executions, abra ela
3. Clique nos `⋯` do nó `Telegram Trigger` → **Copy** o output
4. No canvas, clique no nó → **Edit Output** → cole e ajuste o texto
5. Clique em **Test workflow**

Assim você testa dezenas de variações do prompt rapidinho, sem ficar mandando
mensagem.

---

## 🩺 Tabela de sintomas

| Sintoma | Causa mais provável | Onde resolver |
|---------|---------------------|---------------|
| Bot mudo, sem execuções | Fluxo inativo ou ngrok caído | [doc 02](02-instalacao.md) e [03](03-telegram.md) |
| "The caller does not have permission" | Planilha não compartilhada com a service account | [doc 04, passo 6](04-google-sheets.md) |
| Agente diz que não achou produtos | Nome da aba diferente de `Produtos` | [doc 04, passo 1](04-google-sheets.md) |
| Inventa preços | Regra do System Message removida ou enfraquecida | [prompts/](../prompts/system-prompt-vendedor.md) |
| Esquece tudo a cada mensagem | Campo **Key** da memória vazio | [doc 06, passo 5](06-importar-fluxo.md) |
| Mistura clientes diferentes | `sessionId` fixo em vez de `{{ $json.sessionId }}` | [doc 06, passo 5](06-importar-fluxo.md) |
| Esquece após reiniciar o Docker | Comportamento normal do Simple Memory | [doc 09](09-memoria-persistente.md) |
| Erro `429` | Limite gratuito do Gemini por minuto | espere 1 min |
| Respostas gigantes | Bloco `COMO VOCÊ ESCREVE` fraco | reforce "máximo 4 linhas" |
| Responde em inglês | Falta reforço de idioma | adicione "Responda SEMPRE em português do Brasil" |
| Áudio não funciona | Credencial do Gemini faltando no `Transcrever Audio` | [doc 06, passo 2](06-importar-fluxo.md) |

---

## 🎓 Demonstrações que funcionam bem em sala

| # | O que fazer | O que os alunos entendem |
|---|-------------|--------------------------|
| 1 | Mudar um preço na planilha ao vivo e perguntar de novo | A IA consulta dados reais, não decorou |
| 2 | Apagar a regra "NUNCA invente preço" e perguntar de um produto inexistente | O que é alucinação, na prática |
| 3 | Apagar o campo **Key** da memória e conversar em duplas | Por que o `sessionId` existe |
| 4 | `docker compose restart n8n` no meio de uma conversa | Memória em RAM vs. persistente |
| 5 | Trocar `temperature` de `0.7` para `0` e mandar "oi" 3 vezes | Como a temperatura afeta o tom |
| 6 | Mudar a persona da Bia para um vendedor rude | Como o System Message controla tudo |

---

## ✅ Seu agente está pronto quando

- [ ] Responde texto e áudio
- [ ] Cita preços que **batem com a planilha**
- [ ] Lembra do contexto ao longo da conversa
- [ ] Admite quando não tem o produto
- [ ] Contorna a objeção "tá caro" com argumento de rendimento
- [ ] Registra o pedido na aba `Pedidos`
- [ ] Trata clientes diferentes de forma isolada

---

[⬅️ Anterior: Entendendo o fluxo](07-entendendo-o-fluxo.md) · [Índice](../README.md) · [Próximo: Memória persistente ➡️](09-memoria-persistente.md)
