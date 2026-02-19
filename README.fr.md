<p align="center">
  <a href="README.md">English</a> | <a href="README.ja.md">日本語</a> | <a href="README.zh.md">中文</a> | <a href="README.es.md">Español</a> | <strong>Français</strong> | <a href="README.hi.md">हिन्दी</a> | <a href="README.it.md">Italiano</a> | <a href="README.pt-BR.md">Português</a>
</p>

<p align="center">
  <img src="logo.png" alt="logo mcp-aside" width="280" />
</p>

<h1 align="center">mcp-aside</h1>

<p align="center">
  Un serveur MCP qui offre à votre IA un espace pour noter des idées en cours de conversation — sans interrompre la tâche en cours.
</p>

<p align="center">
  <a href="#démarrage-rapide">Démarrage Rapide</a> &middot;
  <a href="#fonctionnement">Fonctionnement</a> &middot;
  <a href="#outils">Outils</a> &middot;
  <a href="#configuration">Configuration</a> &middot;
  <a href="#licence">Licence</a>
</p>

---

## Pourquoi

Les LLM perdent le fil. Une pensée fugace, une inquiétude à moitié formée, un « on devrait y revenir » qui n'est jamais revisité. **mcp-aside** offre au modèle une boîte de réception dédiée — avec limitation de débit, déduplication et expiration automatique pour que la boîte ne devienne pas un problème en soi.

Imaginez un bloc de post-it à côté de la conversation. Le modèle écrit des notes. Vous (ou le modèle) les lisez au moment opportun.

## Fonctionnement

1. Le modèle appelle `aside.push` avec une pensée étiquetée par priorité.
2. Les garde-fous vérifient les doublons, les limites de débit et les plafonds de TTL.
3. Si c'est validé, l'interjection est stockée dans une boîte de réception en mémoire.
4. Les clients sont notifiés via `notifications/resources/updated`.
5. N'importe qui peut lire la boîte via la ressource `interject://inbox`.

Pas de base de données. Pas de persistance. Si le serveur s'arrête, la boîte disparaît — c'est voulu.

## Démarrage Rapide

```bash
npm install
npm run build
node build/index.js
```

Le serveur utilise MCP via **stdio**. Pointez n'importe quel client compatible MCP vers lui :

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

## Outils

| Outil | Fonction |
|---|---|
| `aside.push` | Envoie une interjection dans la boîte. Accepte `text`, `priority` (low/med/high), `reason`, `tags`, `expiresAt`, `source` et `meta`. |
| `aside.configure` | Ajuste les garde-fous en temps réel — plafonds TTL, limites de débit, fenêtres de déduplication, seuils de notification. |
| `aside.clear` | Vide la boîte de réception. |
| `aside.status` | Instantané en lecture seule de la taille de la boîte et de la configuration actuelle des garde-fous. |

## Ressource

| URI | Description |
|---|---|
| `interject://inbox` | Tableau JSON des interjections en attente, les plus récentes en premier. Les éléments expirés sont filtrés à la lecture. |

## Garde-fous

Tout est configurable via `aside.configure`. Valeurs par défaut :

| Paramètre | Par défaut | Ce qu'il contrôle |
|---|---|---|
| `defaultTtlSeconds` | 600 (10 min) | Durée de vie d'une interjection sans expiration explicite |
| `maxTtlSeconds` | 3600 (1 heure) | Plafond TTL, même si l'appelant demande plus |
| `dedupeWindowSeconds` | 300 (5 min) | Même priorité + texte + raison = supprimé dans cette fenêtre |
| `rateLimitWindowSeconds` | 60 | Fenêtre glissante pour la limitation de débit |
| `rateLimitMax` | low: 6, med: 3, high: 1 | Maximum d'envois par priorité par fenêtre |
| `notifyAtOrAbove` | high | Envoyer les notifications de log uniquement pour les éléments de cette priorité ou supérieure |

## Configuration

### Déclencheur Temporel

Un minuteur intégré se déclenche toutes les 5 minutes, envoyant un check-in de basse priorité « des blocages ? ». Il respecte les mêmes garde-fous que les envois manuels (il sera donc dédupliqué ou limité comme tout le reste). Désactivez-le en commentant l'appel à `startTimerTrigger` dans `index.ts`.

### Inspecteur MCP

Pour les tests locaux :

```
Transport: STDIO
Command:   node
Args:      build/index.js
```

## Notes

- Les logs vont vers **stderr** — stdout est réservé au JSON-RPC MCP.
- La boîte est éphémère. Redémarrage = table rase.
- Les interjections sont stockées du plus récent au plus ancien. Les éléments expirés sont nettoyés à chaque lecture et envoi.

## Licence

[MIT](LICENSE)
