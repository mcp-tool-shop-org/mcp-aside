<p align="center">
  <a href="README.md">English</a> | <a href="README.ja.md">日本語</a> | <a href="README.zh.md">中文</a> | <a href="README.es.md">Español</a> | <a href="README.fr.md">Français</a> | <a href="README.hi.md">हिन्दी</a> | <strong>Italiano</strong> | <a href="README.pt-BR.md">Português</a>
</p>

<p align="center">
  <img src="logo.png" alt="logo mcp-aside" width="280" />
</p>

<h1 align="center">mcp-aside</h1>

<p align="center">
  Un server MCP che offre alla tua IA un posto dove annotare pensieri durante la conversazione — senza interrompere il compito in corso.
</p>

<p align="center">
  <a href="#avvio-rapido">Avvio Rapido</a> &middot;
  <a href="#come-funziona">Come Funziona</a> &middot;
  <a href="#strumenti">Strumenti</a> &middot;
  <a href="#configurazione">Configurazione</a> &middot;
  <a href="#licenza">Licenza</a>
</p>

---

## Perché

Gli LLM perdono il filo. Un pensiero fugace, una preoccupazione a metà, un "dovremmo tornarci" che non viene mai ripreso. **mcp-aside** offre al modello una casella di posta dedicata — con limite di frequenza, deduplicazione e scadenza automatica, così la casella stessa non diventa un problema.

Pensala come un blocco di post-it accanto alla conversazione. Il modello scrive note. Tu (o il modello) le leggi al momento opportuno.

## Come Funziona

1. Il modello chiama `aside.push` con un pensiero etichettato per priorità.
2. Le protezioni verificano duplicati, limiti di frequenza e tetti TTL.
3. Se supera i controlli, l'interjection finisce in una casella in memoria.
4. I client vengono notificati tramite `notifications/resources/updated`.
5. Chiunque può leggere la casella attraverso la risorsa `interject://inbox`.

Nessun database. Nessuna persistenza. Se il server si ferma, la casella scompare — per design.

## Avvio Rapido

```bash
npm install
npm run build
node build/index.js
```

Il server usa MCP su **stdio**. Punta qualsiasi client compatibile MCP verso di esso:

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

## Strumenti

| Strumento | Cosa fa |
|---|---|
| `aside.push` | Invia un'interjection nella casella. Accetta `text`, `priority` (low/med/high), `reason`, `tags`, `expiresAt`, `source` e `meta`. |
| `aside.configure` | Regola le protezioni in tempo reale — tetti TTL, limiti di frequenza, finestre di deduplicazione, soglie di notifica. |
| `aside.clear` | Svuota la casella di posta. |
| `aside.status` | Istantanea in sola lettura della dimensione della casella e della configurazione attuale delle protezioni. |

## Risorsa

| URI | Descrizione |
|---|---|
| `interject://inbox` | Array JSON delle interjection in sospeso, le più recenti per prime. Gli elementi scaduti vengono filtrati alla lettura. |

## Protezioni

Tutto è configurabile tramite `aside.configure`. Valori predefiniti:

| Impostazione | Predefinito | Cosa controlla |
|---|---|---|
| `defaultTtlSeconds` | 600 (10 min) | Durata di vita di un'interjection senza scadenza esplicita |
| `maxTtlSeconds` | 3600 (1 ora) | Tetto massimo TTL, anche se il chiamante chiede di più |
| `dedupeWindowSeconds` | 300 (5 min) | Stessa priorità + testo + motivo = soppresso in questa finestra |
| `rateLimitWindowSeconds` | 60 | Finestra scorrevole per il limite di frequenza |
| `rateLimitMax` | low: 6, med: 3, high: 1 | Massimo invii per priorità per finestra |
| `notifyAtOrAbove` | high | Invia notifiche di log solo per elementi con questa priorità o superiore |

## Configurazione

### Timer Trigger

Un timer integrato si attiva ogni 5 minuti, inviando un check-in a bassa priorità "qualche blocco?". Rispetta le stesse protezioni degli invii manuali (quindi verrà deduplicato o limitato come tutto il resto). Disabilitalo commentando la chiamata a `startTimerTrigger` in `index.ts`.

### Inspector MCP

Per test locali:

```
Transport: STDIO
Command:   node
Args:      build/index.js
```

## Note

- I log vanno su **stderr** — stdout è riservato al JSON-RPC MCP.
- La casella è effimera. Riavvio = tabula rasa.
- Le interjection sono memorizzate dalla più recente alla più vecchia. Gli elementi scaduti vengono ripuliti ad ogni lettura e invio.

## Licenza

[MIT](LICENSE)
