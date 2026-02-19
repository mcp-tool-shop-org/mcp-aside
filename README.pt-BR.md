<p align="center">
  <a href="README.md">English</a> | <a href="README.ja.md">日本語</a> | <a href="README.zh.md">中文</a> | <a href="README.es.md">Español</a> | <a href="README.fr.md">Français</a> | <a href="README.hi.md">हिन्दी</a> | <a href="README.it.md">Italiano</a> | <strong>Português</strong>
</p>

<p align="center">
  <img src="logo.png" alt="logo mcp-aside" width="280" />
</p>

<h1 align="center">mcp-aside</h1>

<p align="center">
  Um servidor MCP que dá à sua IA um lugar para anotar coisas durante a conversa — sem atrapalhar a tarefa em andamento.
</p>

<p align="center">
  <a href="#início-rápido">Início Rápido</a> &middot;
  <a href="#como-funciona">Como Funciona</a> &middot;
  <a href="#ferramentas">Ferramentas</a> &middot;
  <a href="#configuração">Configuração</a> &middot;
  <a href="#licença">Licença</a>
</p>

---

## Por quê

LLMs perdem o fio da meada. Um pensamento solto, uma preocupação meio formada, um "devemos voltar a isso" que nunca acontece. **mcp-aside** dá ao modelo uma caixa de entrada dedicada — com limite de taxa, deduplicação e expiração automática para que a caixa não se torne um problema.

Pense nisso como um bloco de post-its ao lado da conversa. O modelo escreve notas. Você (ou o modelo) as lê no momento certo.

## Como Funciona

1. O modelo chama `aside.push` com um pensamento marcado por prioridade.
2. As proteções verificam duplicatas, limites de taxa e tetos de TTL.
3. Se passar, a interjeição é armazenada em uma caixa de entrada em memória.
4. Os clientes são notificados via `notifications/resources/updated`.
5. Qualquer um pode ler a caixa através do recurso `interject://inbox`.

Sem banco de dados. Sem persistência. Se o servidor parar, a caixa desaparece — por design.

## Início Rápido

```bash
npm install
npm run build
node build/index.js
```

O servidor usa MCP via **stdio**. Aponte qualquer cliente compatível com MCP para ele:

```json
{
  "mcpServers": {
    "aside": {
      "command": "node",
      "args": ["build/index.js"]
    }
  }
}
```

## Ferramentas

| Ferramenta | O que faz |
|---|---|
| `aside.push` | Envia uma interjeição para a caixa. Aceita `text`, `priority` (low/med/high), `reason`, `tags`, `expiresAt`, `source` e `meta`. |
| `aside.configure` | Ajusta as proteções em tempo de execução — tetos de TTL, limites de taxa, janelas de deduplicação, limites de notificação. |
| `aside.clear` | Limpa a caixa de entrada. |
| `aside.status` | Snapshot somente leitura do tamanho da caixa e configuração atual das proteções. |

## Recurso

| URI | Descrição |
|---|---|
| `interject://inbox` | Array JSON de interjeições pendentes, mais recentes primeiro. Itens expirados são filtrados na leitura. |

## Proteções

Tudo é configurável via `aside.configure`. Padrões:

| Configuração | Padrão | O que controla |
|---|---|---|
| `defaultTtlSeconds` | 600 (10 min) | Tempo de vida de uma interjeição sem expiração explícita |
| `maxTtlSeconds` | 3600 (1 hora) | Teto máximo de TTL, mesmo que o chamador peça mais |
| `dedupeWindowSeconds` | 300 (5 min) | Mesma prioridade + texto + motivo = suprimido nesta janela |
| `rateLimitWindowSeconds` | 60 | Janela deslizante para limite de taxa |
| `rateLimitMax` | low: 6, med: 3, high: 1 | Máximo de envios por prioridade por janela |
| `notifyAtOrAbove` | high | Enviar notificações de log apenas para itens com esta prioridade ou superior |

## Configuração

### Timer Trigger

Um timer integrado dispara a cada 5 minutos, enviando um check-in de baixa prioridade "algum bloqueio?". Ele respeita as mesmas proteções dos envios manuais (então será deduplicado ou limitado como qualquer outro). Desative-o comentando a chamada `startTimerTrigger` em `index.ts`.

### Inspetor MCP

Para testes locais:

```
Transport: STDIO
Command:   node
Args:      build/index.js
```

## Notas

- Logs vão para **stderr** — stdout é reservado para MCP JSON-RPC.
- A caixa é efêmera. Reiniciar = lousa limpa.
- Interjeições são armazenadas da mais recente para a mais antiga. Itens expirados são limpos a cada leitura e envio.

## Licença

[MIT](LICENSE)
