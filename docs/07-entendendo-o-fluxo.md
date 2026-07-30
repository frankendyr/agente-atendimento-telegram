# 07 — 🔍 Entendendo cada nó do fluxo

[⬅️ Anterior: Importar](06-importar-fluxo.md) · [Índice](../README.md) · [Próximo: Testes ➡️](08-testes.md)

---

> Este é o documento mais importante para a **aula**. Aqui a gente para de
> configurar e começa a entender *por que* cada peça está ali.

---

## O mapa completo

```
  Telegram Trigger
         │
         ▼
  Texto ou Audio?  ──── saída 2 (áudio) ──▶ Baixar Audio ──▶ Transcrever Audio
         │                                                          │
         │ saída 1 (texto)                                          │
         └──────────────────┬───────────────────────────────────────┘
                            ▼
                   Preparar Mensagem
                            │
                            ▼
                   🤖 AGENTE DE VENDAS ──────▶ Responder no Telegram
                            │
        ┌───────────┬───────┴────────┬──────────────────┐
        ▼           ▼                ▼                  ▼
   Gemini 2.5   Memória      consultar_catalogo   registrar_pedido
   (cérebro)   (contexto)      (lê Produtos)      (escreve Pedidos)
```

Repare: os 4 nós de baixo se conectam ao agente com uma **linha pontilhada** e
um conector em **losango ⚡**. Isso significa "eu sou um recurso do agente", não
"eu venho depois dele".

---

## 1️⃣ Telegram Trigger

**O que faz:** fica escutando o bot. Toda mensagem recebida dispara o fluxo.

**O que ele entrega:** um JSON grande. As partes que importam:

```json
{
  "message": {
    "text": "tem algo pra tirar mofo?",
    "chat": { "id": 123456789 },
    "from": { "first_name": "Kennedy" },
    "voice": { "file_id": "AwACAgEAAx..." }
  }
}
```

| Campo | Uso no nosso fluxo |
|-------|--------------------|
| `message.text` | A mensagem, quando é texto |
| `message.voice` | Existe só quando é áudio |
| `message.chat.id` | 🔑 **A chave da memória** e o destino da resposta |
| `message.from.first_name` | Para a Bia chamar o cliente pelo nome |

---

## 2️⃣ Texto ou Audio? (Switch)

**O que faz:** separa o caminho conforme o tipo de mensagem.

| Saída | Condição | Vai para |
|-------|----------|----------|
| `Texto` | `message.text` existe | Direto para o Preparar Mensagem |
| `Audio` | `message.voice` existe | Baixar → Transcrever → Preparar |

Este é exatamente o mesmo padrão dos assistentes Financeiro e Nutricional. 👍

> 🎯 **Exercício mental:** o que acontece se o cliente mandar uma foto?
> Nenhuma das duas condições bate, o fluxo para ali e o cliente fica sem
> resposta. Como resolver isso é o [exercício 3](10-exercicios.md).

---

## 3️⃣ Baixar Audio + Transcrever Audio

O Telegram não manda o áudio junto com a mensagem — manda só um **`file_id`**.

```
1. Baixar Audio      → usa o file_id para baixar o arquivo de verdade
2. Transcrever Audio → Gemini ouve e devolve o texto
```

O prompt da transcrição é propositalmente seco:

> "Transcreva este audio para texto em portugues do Brasil. Responda APENAS com
> a transcricao literal, sem comentarios, sem aspas e sem nenhum texto adicional."

**Por que tão restritivo?** Porque a saída aqui não é para o cliente ler — é para
alimentar o agente. Se o Gemini escrevesse "Claro! A transcrição é: ...", esse
lixo entraria na conversa e confundiria a Bia.

> 💡 **Regra de ouro:** quando a saída de uma IA alimenta outra IA, peça o
> formato mais limpo possível.

O resultado sai em `content.parts[0].text`.

---

## 4️⃣ Preparar Mensagem (Set) — o nó mais subestimado

Este é o nó que faz o fluxo funcionar, e ele parece bobo.

**O problema que ele resolve:** os dois caminhos chegam com formatos diferentes.

```
Vindo do texto:   $json.message.text
Vindo do áudio:   $json.content.parts[0].text
```

Se o agente tivesse que lidar com os dois, você precisaria de IFs e gambiarras.

**A solução:** padronizar em 3 campos, sempre iguais:

| Campo | Expressão | Papel |
|-------|-----------|-------|
| `mensagem` | `{{ $json.message?.text \|\| $json.content?.parts?.[0]?.text }}` | O que o cliente disse |
| `sessionId` | `{{ $('Telegram Trigger').item.json.message.chat.id }}` | 🔑 Chave da memória |
| `nome` | `{{ $('Telegram Trigger').item.json.message.from.first_name }}` | Nome do cliente |

### Entendendo as duas expressões

**a) O operador `?.` e o `||`**

```javascript
$json.message?.text || $json.content?.parts?.[0]?.text
```

- `?.` = "se existir, pega; se não existir, devolve vazio **sem quebrar**"
- `||` = "se o da esquerda estiver vazio, usa o da direita"

Traduzindo: *"pega o texto do Telegram; se não tiver, pega a transcrição."*
Um campo, dois caminhos. 🎯

**b) O `$('Nome do Nó')`**

```javascript
$('Telegram Trigger').item.json.message.chat.id
```

Depois da transcrição, o `$json` atual é o do Gemini — ele **não tem** o chat.id.
Essa sintaxe diz: *"volta lá no nó Telegram Trigger e pega de lá."*

É como puxar um dado de um passo anterior do fluxo. Guarde essa sintaxe, você
vai usar em todo projeto de n8n.

---

## 5️⃣ 🤖 Agente de Vendas — o coração

Aqui a lógica muda de natureza. Compare:

```
❌ FLUXO LINEAR (aulas anteriores)
   Você desenha o caminho. O modelo só preenche campos.
   Mensagem → IA extrai JSON → Switch decide → Salva → Responde

✅ AGENTE (esta aula)
   Você descreve as ferramentas e as regras. O modelo monta o caminho.
   Mensagem → Agente pensa → escolhe ferramenta(s) → pensa de novo → Responde
```

### O ciclo de raciocínio (ReAct)

Por dentro, o agente roda um loop:

```
   ┌────────────────────────────────────┐
   │  1. PENSAR                          │
   │     "O cliente quer preço.          │
   │      Não sei o preço.               │
   │      Tenho a ferramenta catálogo."  │
   ├────────────────────────────────────┤
   │  2. AGIR                            │
   │     chama consultar_catalogo        │
   ├────────────────────────────────────┤
   │  3. OBSERVAR                        │
   │     recebe as 20 linhas da planilha │
   ├────────────────────────────────────┤
   │  4. PENSAR DE NOVO                  │
   │     "Achei. Agora escrevo seguindo  │
   │      a técnica de vendas."          │
   ├────────────────────────────────────┤
   │  5. RESPONDER  ✅                    │
   └────────────────────────────────────┘
```

O `maxIterations: 10` é o freio de segurança: se ele ficar preso em loop
chamando ferramentas, para na décima volta.

### Os dois campos que você configura

| Campo | Conteúdo | Analogia |
|-------|----------|----------|
| **Prompt** | `{{ $json.mensagem }}` | O que o cliente **acabou de dizer** |
| **System Message** | as regras de vendas | O **treinamento** que a Bia recebeu |

O System Message é fixo, vale para toda mensagem. O Prompt muda a cada turno.

### Como o prompt de vendas está montado

O System Message não é um texto solto — ele segue uma estrutura que você pode
reaproveitar para qualquer agente:

| Bloco | O que define | Por que importa |
|-------|--------------|-----------------|
| `QUEM VOCÊ É` | Persona, nome, empresa, tom | Sem isso o modelo soa genérico |
| `SUAS FERRAMENTAS` | O que ele pode fazer | Ajuda a escolher a ferramenta certa |
| `REGRAS INEGOCIÁVEIS` | Limites duros | **Evita que invente preço** |
| `COMO CONDUZ A VENDA` | As 7 etapas | Transforma resposta em *venda* |
| `COMO VOCÊ ESCREVE` | Formato e tamanho | Sem isso ele escreve textão |
| `MEMÓRIA` | Como usar o histórico | Evita repetir perguntas |

> 🎯 **A regra mais importante do projeto inteiro:**
>
> *"NUNCA invente produto, preço ou característica. SEMPRE use
> consultar_catalogo antes de citar qualquer preço."*
>
> Sem essa frase, o modelo **alucina preços plausíveis**. Ele não faz por mal:
> ele foi treinado para completar texto, e "R$ 12,90" completa muito bem a frase.
> Teste depois removendo essa regra — é uma demonstração ótima em sala.

---

## 6️⃣ Gemini 2.5 Flash (Chat Model)

O cérebro. `temperature: 0.7` para soar natural sem viajar.

⚠️ Note que é o nó **Google Gemini Chat Model** — diferente do nó
**Google Gemini** usado na transcrição. Só o Chat Model conecta em agentes.

---

## 7️⃣ Memoria da Conversa

**Tipo:** Simple Memory (Buffer Window) — guarda na RAM do n8n.

| Config | Valor | Efeito |
|--------|-------|--------|
| Session Key | `{{ $json.sessionId }}` | Uma memória por cliente |
| Context Window | `30` | Lembra das últimas 30 mensagens |

### Como funciona por dentro

A cada mensagem, o agente recebe isto (você não vê, mas está lá):

```
[System]  ...todo o prompt de vendas...
[Human]   Quero algo pra cozinha           ← veio da memória
[AI]      Claro! O Desengordurante...      ← veio da memória
[Human]   Pode ser, quanto fica com 2?     ← a mensagem de agora
```

É por isso que ele "lembra": o histórico é **reenviado** a cada turno.

### As duas limitações que você precisa conhecer

| Limitação | Consequência |
|-----------|--------------|
| 🧠 Fica na RAM | `docker compose down` apaga tudo |
| 📏 Só 30 mensagens | Numa conversa longa, o começo se perde |

Para resolver a primeira, veja [09 — memória persistente](09-memoria-persistente.md).

> 🎯 **Demonstração matadora em sala:** converse com o bot, depois rode
> `docker compose restart n8n` e continue a conversa. Ele esqueceu tudo.
> Aí você liga o Postgres e mostra a diferença.

---

## 8️⃣ consultar_catalogo (Google Sheets Tool)

**Não é um nó comum de planilha — é uma FERRAMENTA.** A diferença:

| | Nó normal | Nó Tool |
|---|---|---|
| Quando executa | Sempre que o fluxo passa | **Só quando o agente decide** |
| Quem chama | O fluxo | O modelo |
| Campo extra | — | **Description** |

O campo **Description** é o que o modelo lê para decidir se usa ou não:

> "Consulta o catalogo COMPLETO de produtos... Use SEMPRE esta ferramenta antes
> de citar qualquer produto, preco ou caracteristica para o cliente."

> ⚠️ **A descrição da ferramenta é um prompt.** Se ela for vaga ("busca dados"),
> o agente não vai saber quando usar. Escreva como se explicasse para um
> funcionário novo.

Esta ferramenta lê **todas as 20 linhas** e entrega ao agente, que filtra
mentalmente. Com um catálogo de 20 produtos isso é simples e confiável. Com
5.000 produtos você precisaria de filtro ou de um banco vetorial —
assunto da próxima aula. 😉

---

## 9️⃣ registrar_pedido (Google Sheets Tool)

Mesma ideia, mas **escreve**. O interessante aqui é como o agente preenche as
colunas:

```javascript
{{ $fromAI('cliente', 'Nome completo do cliente', 'string') }}
```

O **`$fromAI()`** é uma ponte: ele diz ao modelo *"preencha este campo você
mesmo, com base na conversa"*. Os 3 parâmetros:

| Parâmetro | Exemplo | Papel |
|-----------|---------|-------|
| nome | `'cliente'` | Identificador do campo |
| descrição | `'Nome completo do cliente'` | **O modelo lê isto para saber o que pôr** |
| tipo | `'string'` | string, number ou boolean |

A coluna `Data` é a exceção — ela não usa `$fromAI`:

```javascript
{{ $now.format('dd/MM/yyyy HH:mm') }}
```

**Por quê?** Porque modelos de linguagem são péssimos em saber que dia é hoje.
Data e hora sempre calcule no n8n, nunca peça para a IA.

> 🎯 **Princípio geral:** o que a máquina sabe com certeza (data, ID do chat,
> cotação), calcule no fluxo. O que depende de interpretação (nome, produto
> escolhido, observação), peça ao modelo.

---

## 🔟 Responder no Telegram

Envia a resposta final:

| Campo | Valor |
|-------|-------|
| Chat ID | `{{ $('Telegram Trigger').item.json.message.chat.id }}` |
| Text | `{{ $json.output }}` |

O agente **sempre** devolve a resposta final no campo `output`. Guarde isso.

E `appendAttribution: false` remove o rodapé "This message was sent
automatically with n8n" — num bot de vendas, isso entrega o jogo. 😄

---

## ✅ Checagem de entendimento

1. Por que o nó `Preparar Mensagem` existe? O que quebraria sem ele?
2. Qual a diferença entre um nó Google Sheets normal e um Google Sheets **Tool**?
3. Por que a coluna `Data` não usa `$fromAI()`?
4. O que acontece se você apagar a regra "NUNCA invente preço"?
5. Onde o agente coloca a resposta final?

<details>
<summary>Ver respostas</summary>

1. Ele unifica os dois caminhos (texto e áudio) num formato único e cria o
   `sessionId`. Sem ele, o agente receberia formatos diferentes e a memória não
   teria chave.
2. O nó normal executa sempre que o fluxo passa por ele. O Tool só executa
   quando o **modelo decide** chamá-lo, com base na Description.
3. Porque modelos de linguagem não sabem a data atual com confiabilidade. Data
   é um dado que a máquina sabe com certeza — calcule no fluxo.
4. O modelo passa a inventar preços plausíveis (alucinação), porque completar
   texto é o que ele faz por natureza.
5. No campo `output`.

</details>

---

[⬅️ Anterior: Importar](06-importar-fluxo.md) · [Índice](../README.md) · [Próximo: Testes ➡️](08-testes.md)
