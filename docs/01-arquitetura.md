# 01 — 🏗️ Entendendo a arquitetura

[⬅️ Anterior: Como baixar](00-como-baixar.md) · [Índice](../README.md) · [Próximo: Instalação ➡️](02-instalacao.md)

---

## O problema que estamos resolvendo

Uma loja de produtos de limpeza recebe dezenas de mensagens por dia no Telegram:

- "Vocês têm alguma coisa boa pra tirar mofo do box?"
- "Quanto tá o detergente?"
- "Aquele desengordurante funciona em fogão mesmo?"

Responder isso manualmente é lento e cansativo. E o pior: **quem responde
precisa saber de cor o preço de tudo.**

Queremos um vendedor digital que:

1. Conheça o catálogo real (sem inventar preço)
2. Conduza a conversa como um vendedor de verdade
3. Lembre do que o cliente já falou
4. Registre o pedido quando fechar

---

## As 4 camadas do sistema

Pense em qualquer agente de IA como **4 camadas empilhadas**. Sempre é assim.

```
┌─────────────────────────────────────────────────────┐
│  4. CANAL       Por onde o cliente fala             │
│                 → Telegram                          │
├─────────────────────────────────────────────────────┤
│  3. CÉREBRO     Quem raciocina e escreve            │
│                 → Google Gemini 2.5 Flash           │
├─────────────────────────────────────────────────────┤
│  2. MEMÓRIA     O que já foi conversado             │
│                 → Buffer Window (ou Postgres)       │
├─────────────────────────────────────────────────────┤
│  1. CONHECIMENTO  Os dados reais do negócio         │
│                 → Google Sheets (catálogo)          │
└─────────────────────────────────────────────────────┘
         Tudo orquestrado pelo n8n, dentro do Docker
```

Se você entender essas 4 camadas, você monta um agente para **qualquer** nicho:
imobiliária, clínica, pet shop, autopeças. Só troca o conteúdo de cada camada.

---

## O caminho de uma mensagem (passo a passo)

Vamos seguir uma mensagem do começo ao fim:

```
1️⃣  Cliente digita "tem algo pra tirar mofo?" no Telegram
              │
              ▼
2️⃣  Telegram envia isso para a URL pública do NGROK
              │   (o Telegram não consegue falar com "localhost")
              ▼
3️⃣  NGROK repassa para o n8n rodando no seu computador
              │
              ▼
4️⃣  Telegram Trigger recebe a mensagem e dispara o fluxo
              │
              ▼
5️⃣  Switch verifica: é texto ou áudio?
              │
              ├─ áudio → baixa o arquivo → Gemini transcreve → vira texto
              │
              └─ texto → segue direto
              │
              ▼
6️⃣  Preparar Mensagem padroniza tudo:
              │   mensagem = "tem algo pra tirar mofo?"
              │   sessionId = 123456789  ← chave da memória!
              │   nome      = "Kennedy"
              ▼
7️⃣  AGENTE DE VENDAS recebe e RACIOCINA:
              │   "Preciso saber quais produtos existem para mofo"
              │   → chama a ferramenta consultar_catalogo
              │   → recebe as 20 linhas da planilha
              │   → acha o "Removedor de Limo e Mofo — R$ 17,90"
              │   → escreve a resposta seguindo a técnica de vendas
              ▼
8️⃣  Responder no Telegram envia a resposta ao cliente
              │
              ▼
9️⃣  A conversa fica salva na MEMÓRIA, ligada ao sessionId
    (na próxima mensagem, o agente lembra de tudo isso)
```

---

## Por que cada peça existe

### 🐳 Docker

Sobe o n8n e o ngrok com **um comando só**, e funciona igual no seu Mac, no
Windows do colega e no Linux do servidor.

Sem Docker, cada aluno instalaria Node.js, dependências, versões diferentes...
e metade da aula viraria suporte técnico.

### 🌐 ngrok

Este é o ponto que mais confunde. Preste atenção:

O n8n está rodando em `http://localhost:5678`. Mas **`localhost` só existe
dentro do seu computador**. Os servidores do Telegram, lá fora na internet, não
têm como chegar até aí.

O ngrok resolve criando um **túnel**:

```
Telegram  ──▶  https://seu-dominio.ngrok-free.dev  ──▶  seu PC :5678
              (endereço público, visível na internet)
```

> 💡 Por isso usamos um **domínio fixo** do ngrok. Se o endereço mudasse a cada
> reinicialização, você teria que reconfigurar o webhook do Telegram toda vez.

### 🔄 n8n

O orquestrador. É onde você desenha o fluxo arrastando blocos, sem código.

### 🤝 AI Agent (a novidade desta aula)

Nos fluxos anteriores, **você** decidia o caminho com nós de `Switch` e `IF`.

Com o nó **AI Agent**, quem decide é o modelo. Você dá a ele:

- **Ferramentas** — o que ele *pode* fazer
- **System Message** — como ele *deve* se comportar
- **Memória** — o que ele já *sabe*

E ele monta o caminho sozinho, a cada mensagem.

```
Ferramentas disponíveis:
  🔧 consultar_catalogo  → "lê os produtos da planilha"
  🔧 registrar_pedido    → "grava um pedido fechado"

Cliente: "quanto custa o desengordurante?"
Agente pensa: preciso do preço → uso consultar_catalogo ✅

Cliente: "bom dia"
Agente pensa: é só um cumprimento → não preciso de ferramenta ✅

Cliente: "pode fechar, sou o João, 85 99999-0000"
Agente pensa: pedido confirmado + tenho os dados → uso registrar_pedido ✅
```

### 💾 Memória

Sem memória, cada mensagem é um recomeço do zero:

```
❌ SEM MEMÓRIA
Cliente: Quero algo pra cozinha
Bia:     Claro! O Desengordurante resolve. R$ 14,90.
Cliente: Pode ser, quanto fica com 2?
Bia:     Desculpa, com 2 de quê? 🤡
```

```
✅ COM MEMÓRIA
Cliente: Quero algo pra cozinha
Bia:     Claro! O Desengordurante resolve. R$ 14,90.
Cliente: Pode ser, quanto fica com 2?
Bia:     2 unidades dão R$ 29,80. Fecho pra você? 😊
```

**O detalhe mais importante da memória é o `sessionId`.**

Ele é o "crachá" que separa uma conversa da outra. Usamos o **ID do chat do
Telegram**, que é único por pessoa:

```
sessionId 111111  → conversa da Maria  (isolada)
sessionId 222222  → conversa do João   (isolada)
```

Se você esquecer de configurar isso, **todos os clientes compartilham a mesma
memória** — e a Bia vai misturar o pedido da Maria com o do João. 😱

### 📊 Google Sheets

Poderia ser um banco de dados. Escolhemos planilha porque:

- A dona da loja consegue mudar o preço sozinha, sem chamar o programador
- Você vê os pedidos chegando em tempo real
- É grátis e todo mundo já sabe usar

Usamos **duas abas**:

| Aba | Papel | O agente faz |
|-----|-------|--------------|
| `Produtos` | Catálogo (o que vendemos) | **lê** |
| `Pedidos` | Registro de vendas | **escreve** |

---

## Por que Gemini 2.5 Flash?

| Motivo | Detalhe |
|--------|---------|
| 🎧 Entende áudio nativamente | Sem precisar de um serviço separado de transcrição |
| ⚡ Rápido | O cliente não fica esperando |
| 💰 Barato | Tem camada gratuita generosa, perfeita para a aula |
| 🔧 Suporta ferramentas | Essencial para o agente decidir o que chamar |

E é o mesmo componente que já usamos nos assistentes Financeiro e Nutricional —
então você já conhece a credencial.

---

## ✅ Checagem de entendimento

Antes de seguir, responda mentalmente:

1. Por que não podemos apontar o Telegram direto para `localhost:5678`?
2. O que aconteceria se o `sessionId` fosse fixo, igual para todos?
3. Qual a diferença entre o Switch (fluxo linear) e o AI Agent?
4. Por que o agente precisa da ferramenta em vez de ter os preços no prompt?

<details>
<summary>Ver respostas</summary>

1. Porque `localhost` só existe dentro da sua máquina. Servidores externos não
   alcançam esse endereço — por isso o túnel do ngrok.
2. Todos os clientes compartilhariam a mesma memória. A Bia trataria o João
   como se fosse a Maria e misturaria os pedidos.
3. No Switch **você** define o caminho antes, com regras fixas. No AI Agent o
   **modelo** escolhe o caminho na hora, com base no contexto.
4. Porque a planilha é a fonte da verdade e muda toda hora. Se os preços
   estivessem no prompt, você teria que editar o fluxo a cada reajuste — e o
   modelo tenderia a inventar valores.

</details>

---

[⬅️ Anterior: Como baixar](00-como-baixar.md) · [Índice](../README.md) · [Próximo: Instalação ➡️](02-instalacao.md)
