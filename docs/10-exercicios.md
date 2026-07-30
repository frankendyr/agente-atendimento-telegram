# 10 — 🎯 Exercícios para praticar

[⬅️ Anterior: Memória persistente](09-memoria-persistente.md) · [Índice](../README.md)

---

Os exercícios estão em ordem crescente de dificuldade. Faça na ordem — cada um
constrói em cima do anterior.

---

## 🟢 Nível 1 — Mexendo no prompt

### Exercício 1 — Sua própria loja

Troque a persona da Bia para o **seu** negócio (real ou inventado).

No **System Message**, mude:
- O nome da vendedora e da empresa
- Os exemplos de upsell (`ETAPA 4`)
- As objeções típicas do seu mercado (`ETAPA 5`)

Depois troque a planilha `Produtos` pelo seu catálogo, **mantendo os nomes das
colunas**.

**O que você aprende:** que a arquitetura é genérica. O nicho é só conteúdo.

---

### Exercício 2 — Política de desconto

Faça a Bia poder dar até **10% de desconto**, mas só quando o cliente
reclamar do preço **duas vezes** — e nunca de primeira.

💡 Adicione um bloco novo no System Message:

```
# POLÍTICA DE DESCONTO
Você pode oferecer até 10% de desconto, mas SOMENTE se:
- o cliente reclamar do preço pelo menos duas vezes, E
- o pedido for acima de R$ 50,00
Nunca ofereça desconto de primeira. Primeiro tente o argumento do rendimento.
Ao dar desconto, registre o valor já com o desconto aplicado.
```

**Teste:** reclame uma vez (não deve dar), reclame de novo (deve dar).

**O que você aprende:** regras condicionais em linguagem natural.

---

## 🟡 Nível 2 — Mexendo no fluxo

### Exercício 3 — Aceitar fotos

Hoje, se o cliente manda uma foto, o fluxo morre em silêncio.

Adicione uma terceira saída no Switch para `message.photo`, baixe a imagem e
peça ao Gemini para descrever. Depois faça a descrição virar `mensagem`.

💡 Você já fez exatamente isso no **Assistente Nutricional** — copie o padrão de
lá: `Baixar Foto` → `Analyze an image`.

💡 O `file_id` da foto em melhor resolução:
```javascript
{{ $json.message.photo[$json.message.photo.length - 1].file_id }}
```

💡 Não esqueça de ajustar a expressão do `Preparar Mensagem` para aceitar a
terceira origem.

**Caso de uso real:** o cliente fotografa a mancha no box e a Bia recomenda o
produto certo.

---

### Exercício 4 — "Digitando..."

Adicione um nó do Telegram com `resource: chat` e ação **sendChatAction**
(`typing`) logo depois do `Preparar Mensagem`.

O cliente vê "digitando..." enquanto o agente pensa. Parece bobo, mas muda
completamente a percepção de humanidade do bot.

---

### Exercício 5 — Mensagem longa demais

O Telegram corta mensagens acima de 4096 caracteres. Adicione um nó de código
que quebre a resposta em pedaços quando ela for muito longa.

💡 Ou resolva no prompt, limitando o tamanho — e discuta: **qual solução é
melhor?** (Dica: as duas. Prompt para o caso normal, código para a garantia.)

---

## 🟠 Nível 3 — Novas ferramentas

### Exercício 6 — Consultar frete

Crie uma terceira ferramenta que consulta uma aba nova `Fretes`:

| Bairro | Prazo | Valor |
|--------|-------|-------|
| Aldeota | 1 dia | 8.00 |
| Meireles | 1 dia | 8.00 |
| Messejana | 2 dias | 15.00 |

Adicione ao System Message: *"Sempre pergunte o bairro antes de fechar o pedido
e informe o frete."*

**O que você aprende:** que adicionar capacidade a um agente é só adicionar
ferramenta + instrução. Sem tocar na lógica.

---

### Exercício 7 — Avisar o vendedor humano

Quando o pedido for registrado, mande também uma mensagem para **o seu** chat
pessoal do Telegram:

```
🔔 Novo pedido!
Cliente: Kennedy Ribeiro
Total: R$ 39,80
```

💡 Você precisa do **seu** chat ID. Descubra falando com o bot `@userinfobot`.

💡 Pense: isso deve ser uma **ferramenta** do agente ou um **nó normal** depois
do agente? (Resposta: nó normal — não é uma decisão do agente, é uma
consequência automática. Mas para isso você precisaria reestruturar o fluxo.
Discuta em sala.)

---

### Exercício 8 — Ferramenta de horário

Crie uma ferramenta simples (nó **Code Tool**) que retorna a data e hora atual,
e adicione ao prompt: *"Se for depois das 18h, avise que a entrega só sai no
próximo dia útil."*

**O que você aprende:** que dar "consciência de tempo" ao agente exige uma
ferramenta — o modelo sozinho não sabe que horas são.

---

## 🔴 Nível 4 — Arquitetura

### Exercício 9 — Relatório diário automático

Crie um **segundo fluxo** que:

1. Roda todo dia às 20h (nó **Schedule Trigger**)
2. Lê a aba `Pedidos` do dia
3. Manda para o Gemini resumir
4. Envia no seu Telegram:

```
📊 Resumo do dia
Pedidos: 7
Faturamento: R$ 412,30
Produto mais vendido: Desengordurante Cozinha Pesada
```

---

### Exercício 10 — Catálogo grande de verdade

Nosso `consultar_catalogo` lê **todas** as linhas. Com 20 produtos, tudo bem.
Com 5.000, o agente estoura o limite de contexto.

Pesquise e proponha uma solução usando:
- **Vector Store** (Qdrant, Pinecone ou Supabase)
- **Embeddings** do Gemini
- Nó **Vector Store Tool**

Escreva um mini-documento explicando **por que** a busca vetorial resolve isso,
e o que ela troca em precisão.

**O que você aprende:** o limite arquitetural desta aula e o caminho do RAG.

---

### Exercício 11 — Detectar frustração

Faça o agente perceber quando o cliente está irritado (muitas objeções seguidas,
palavrão, "deixa pra lá") e **transferir para um humano**, avisando você no
Telegram e parando de responder aquele cliente.

💡 Dica: isso exige guardar um estado por cliente. Onde você guardaria?
(Planilha? Postgres? Redis?) Justifique a escolha.

---

## 🏆 Desafio final

Monte um agente completo para **outro nicho** do zero, com:

- [ ] Catálogo próprio numa planilha
- [ ] Persona e técnica de vendas específicas do mercado
- [ ] Pelo menos **3 ferramentas**
- [ ] Memória persistente (Postgres)
- [ ] Suporte a texto **e** áudio
- [ ] Registro de pedidos/agendamentos
- [ ] Notificação para o dono do negócio
- [ ] README próprio, publicado no seu GitHub

**Sugestões de nicho:**

| Nicho | Ferramentas interessantes |
|---|---|
| 🏠 Imobiliária | catálogo de imóveis, agendamento de visita, simulação de financiamento |
| 🐕 Pet shop | produtos, agenda do banho e tosa, ficha do pet |
| 🍕 Pizzaria | cardápio, montagem do pedido, cálculo de entrega |
| 💇 Salão | serviços, agenda de horários, histórico do cliente |
| 🚗 Autopeças | catálogo por modelo do carro, consulta de estoque, orçamento |

---

## 📚 Para ir além

| Tema | Onde estudar |
|------|--------------|
| Documentação do n8n | https://docs.n8n.io |
| Nós de IA do n8n | https://docs.n8n.io/advanced-ai/ |
| Google Gemini | https://ai.google.dev/gemini-api/docs |
| API do Telegram Bot | https://core.telegram.org/bots/api |
| Comunidade n8n (PT-BR) | https://community.n8n.io |

---

[⬅️ Anterior: Memória persistente](09-memoria-persistente.md) · [Índice](../README.md)
