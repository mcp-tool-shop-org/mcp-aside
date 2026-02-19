<p align="center">
  <a href="README.md">English</a> | <a href="README.ja.md">日本語</a> | <a href="README.zh.md">中文</a> | <strong>Español</strong> | <a href="README.fr.md">Français</a> | <a href="README.hi.md">हिन्दी</a> | <a href="README.it.md">Italiano</a> | <a href="README.pt-BR.md">Português</a>
</p>

<p align="center">
  <img src="logo.png" alt="logo de mcp-aside" width="280" />
</p>

<h1 align="center">mcp-aside</h1>

<p align="center">
  Un servidor MCP que le da a tu IA un lugar para anotar cosas durante la conversación — sin descarrilar la tarea en curso.
</p>

<p align="center">
  <a href="#inicio-rápido">Inicio Rápido</a> &middot;
  <a href="#cómo-funciona">Cómo Funciona</a> &middot;
  <a href="#herramientas">Herramientas</a> &middot;
  <a href="#configuración">Configuración</a> &middot;
  <a href="#licencia">Licencia</a>
</p>

---

## Por qué

Los LLM pierden el hilo. Un pensamiento suelto, una preocupación a medio formar, un "deberíamos volver a esto" que nunca se retoma. **mcp-aside** le da al modelo una bandeja de entrada dedicada — con límite de tasa, deduplicación y expiración automática para que la bandeja no se convierta en un problema.

Piénsalo como un bloc de notas adhesivas junto a la conversación. El modelo escribe notas. Tú (o el modelo) las lees cuando sea oportuno.

## Cómo Funciona

1. El modelo llama a `aside.push` con un pensamiento etiquetado por prioridad.
2. Las barreras de protección verifican duplicados, límites de tasa y topes de TTL.
3. Si pasa, la interjección se almacena en una bandeja de entrada en memoria.
4. Los clientes reciben notificación vía `notifications/resources/updated`.
5. Cualquiera puede leer la bandeja a través del recurso `interject://inbox`.

Sin base de datos. Sin persistencia. Si el servidor se detiene, la bandeja desaparece — por diseño.

## Inicio Rápido

```bash
npm install
npm run build
node build/index.js
```

El servidor usa MCP sobre **stdio**. Apunta cualquier cliente compatible con MCP hacia él:

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

## Herramientas

| Herramienta | Qué hace |
|---|---|
| `aside.push` | Envía una interjección a la bandeja. Acepta `text`, `priority` (low/med/high), `reason`, `tags`, `expiresAt`, `source` y `meta`. |
| `aside.configure` | Ajusta las barreras en tiempo de ejecución — topes de TTL, límites de tasa, ventanas de deduplicación, umbrales de notificación. |
| `aside.clear` | Vacía la bandeja de entrada. |
| `aside.status` | Instantánea de solo lectura del tamaño de la bandeja y la configuración actual de barreras. |

## Recurso

| URI | Descripción |
|---|---|
| `interject://inbox` | Array JSON de interjecciones pendientes, las más recientes primero. Los elementos expirados se filtran al leer. |

## Barreras de Protección

Todo es configurable vía `aside.configure`. Valores predeterminados:

| Configuración | Predeterminado | Qué controla |
|---|---|---|
| `defaultTtlSeconds` | 600 (10 min) | Tiempo de vida de una interjección si no se establece expiración explícita |
| `maxTtlSeconds` | 3600 (1 hora) | Tope máximo de TTL, incluso si el llamador pide más |
| `dedupeWindowSeconds` | 300 (5 min) | Misma prioridad + texto + razón = suprimido dentro de esta ventana |
| `rateLimitWindowSeconds` | 60 | Ventana deslizante para límite de tasa |
| `rateLimitMax` | low: 6, med: 3, high: 1 | Máximo de envíos por prioridad por ventana |
| `notifyAtOrAbove` | high | Solo enviar notificaciones de log para elementos con esta prioridad o superior |

## Configuración

### Disparador por Temporizador

Un temporizador incorporado se activa cada 5 minutos, enviando un check-in de baja prioridad "¿algún bloqueo?". Respeta las mismas barreras que los envíos manuales (así que será deduplicado o limitado como cualquier otro). Desactívalo comentando la llamada a `startTimerTrigger` en `index.ts`.

### Inspector MCP

Para pruebas locales:

```
Transport: STDIO
Command:   node
Args:      build/index.js
```

## Notas

- Los logs van a **stderr** — stdout está reservado para MCP JSON-RPC.
- La bandeja es efímera. Reiniciar = pizarra limpia.
- Las interjecciones se almacenan de más reciente a más antigua. Los elementos expirados se limpian en cada lectura y envío.

## Licencia

[MIT](LICENSE)
