# 05 — 🧠 Pegando a chave do Google Gemini

[⬅️ Anterior: Google Sheets](04-google-sheets.md) · [Índice](../README.md) · [Próximo: Importar o fluxo ➡️](06-importar-fluxo.md)

---

Esta é a parte mais rápida do projeto. 3 minutos.

---

## Passo 1 — Gerar a API Key

1. Acesse https://aistudio.google.com/apikey
2. Faça login com sua conta Google
3. Clique em **Create API key**
4. Escolha o projeto (pode ser o mesmo `n8n-vendas` do doc anterior)
5. Copie a chave gerada (começa com `AIza...`)

> ⚠️ A chave aparece **uma vez**. Guarde num lugar seguro.
> Se perder, é só gerar outra — mas lembre de atualizar no n8n.

---

## Passo 2 — Cadastrar no n8n

1. No n8n: **Credentials** → **Add credential**
2. Busque por **Google Gemini(PaLM) Api**
3. Cole a chave no campo **API Key**
4. Deixe o **Host** como está: `https://generativelanguage.googleapis.com`
5. Nome: `Gemini Vendas`
6. **Save**

É a mesma credencial que você já usou nos assistentes Financeiro e Nutricional.
Se ela já existe no seu n8n, pode reaproveitar.

---

## 🧠 Onde o Gemini é usado neste projeto

Ele aparece em **dois lugares diferentes**, com papéis distintos:

```
┌──────────────────────────────────────────────────────────┐
│  1. Transcrever Audio                                    │
│     Nó: Google Gemini (resource: audio, analyze)         │
│     Papel: transformar o áudio do cliente em texto       │
│     → é uma tarefa única, sem memória, sem ferramenta    │
├──────────────────────────────────────────────────────────┤
│  2. Gemini 2.5 Flash                                     │
│     Nó: Google Gemini Chat Model                         │
│     Papel: ser o CÉREBRO do agente                       │
│     → raciocina, escolhe ferramentas, escreve a resposta │
└──────────────────────────────────────────────────────────┘
```

**Note a diferença de nó**, ela é importante:

| | Google Gemini | Google Gemini **Chat Model** |
|---|---|---|
| Usado sozinho | ✅ | ❌ |
| Usado dentro de um agente | ❌ | ✅ |
| Tem memória | ❌ | ✅ (via nó de memória) |
| Usa ferramentas | ❌ | ✅ |
| Ícone de conexão | quadrado (fluxo normal) | ⚡ losango (por baixo do agente) |

Nos assistentes anteriores você só usou o **primeiro tipo**. O `Chat Model` é o
que permite montar um agente.

---

## 💰 Sobre custos e limites

O plano gratuito do Gemini é generoso e dá tranquilamente para a aula inteira.

| | Free tier (aproximado) |
|---|---|
| Requisições por minuto | ~15 |
| Requisições por dia | ~1.500 |

Cada mensagem do cliente consome **de 2 a 4 requisições** (o agente pensa, chama
a ferramenta, pensa de novo, responde).

### Se aparecer erro `429 Too Many Requests`

Significa que você passou do limite por minuto. Espere 1 minuto e tente de novo.
Numa sala com muitos alunos, o ideal é **cada um usar a própria chave**.

---

## 🎛️ Por que `temperature: 0.7`?

No nó `Gemini 2.5 Flash` deixamos a temperatura em **0.7**. Esse número controla
o quanto o modelo "improvisa":

| Temperatura | Comportamento | Bom para |
|---|---|---|
| `0` a `0.3` | Sempre a mesma resposta, previsível | Extrair dados, classificar |
| `0.6` a `0.8` | Varia o jeito de falar, soa natural | 👈 **Vendas, conversa** |
| `1.0`+ | Criativo demais, começa a viajar | Brainstorm, texto criativo |

Num assistente financeiro você quer `0.2` — precisão. Num vendedor você quer
`0.7`, senão ele repete a mesma frase para todo mundo e soa robótico.

> 🎯 **Isso não afeta os preços!** Os preços vêm da planilha, não da criatividade
> do modelo. A temperatura muda o *jeito de falar*, não os *fatos*.

---

## ✅ Antes de seguir, você deve ter

- [ ] API Key do Gemini gerada
- [ ] Credencial `Gemini Vendas` salva no n8n

---

[⬅️ Anterior: Google Sheets](04-google-sheets.md) · [Índice](../README.md) · [Próximo: Importar o fluxo ➡️](06-importar-fluxo.md)
