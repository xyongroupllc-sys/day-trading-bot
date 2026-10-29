# Day Trading Bot — Prompts CCR

Sistema de day trading automatizado sobre Robinhood usando Claude Code Remote (CCR). Seis rutinas encadenadas cubren el ciclo completo de una sesión: análisis pre-market, ejecución de órdenes, monitoreo intraday y cierre forzado al final del día.

## Ciclo de Vida de una Sesión

```
08:30 ET  Pre-Market      → Análisis, selección de candidatos (#1/#2/#3), Telegram
09:35 ET  Ejecutor        → Coloca órdenes GFD (stop-limit o limit)
09:40 ET  Monitor 9:40    → Primer chequeo de fill; arma OCO al llenar
11:00 ET  Monitor 11:00   → Segundo y último chequeo; cancela pendientes si no llenaron
13:00 ET  Monitor 1:00 PM → Gestión activa de posición abierta
15:45 ET  Cierre          → Cierre forzado + Journal + Scorecard
```

## Rutinas

| Archivo | Rutina | Trigger ID | Cron (UTC) |
|---|---|---|---|
| `01_pre_market.md` | Pre-Market | `trig_0192TAKQQiwgKSNNQJ8P9zyd` | `30 12 * * 1-5` |
| `02_ejecutor.md` | Ejecutor | `trig_01XmxNsyQmLUDuNZ2k2hcMjg` | `35 13 * * 1-5` |
| `03_monitor_940.md` | Monitor 9:40 | `trig_01A7d39uSKb5utzvNENUriB5` | `40 13 * * 1-5` |
| `04_monitor_1100.md` | Monitor 11:00 | `trig_014wjKAvMu1mj8nq8puBPsFT` | `0 15 * * 1-5` |
| `05_monitor_100pm.md` | Monitor 1:00 PM | `trig_01R9HURVTX7qD2SBxUfJcJfL` | `0 17 * * 1-5` |
| `06_cierre.md` | Cierre | `trig_01JfXHtBEcy8Y3Twi3CaPN1q` | `45 19 * * 1-5` |

## Cuenta y MCPs

- **Cuenta Robinhood:** 737702357
- **MCPs requeridos:** `Robinhood-Trading`, `Notion`
- **Notificaciones:** Telegram vía Python `urllib` (único canal HTTP confiable desde CCR)
- **Estado compartido:** Notion DB — `collection://a5807f9a-7cfb-4c61-b643-bcbda50bb463`

## Ciclo Activo

**V2 - Ago 2026+** — Todos los registros de Notion usan este valor en el campo `Ciclo`.

## Lógica Clave

- **Regla de tamaño:** `qty = round(150 / Precio Entrada)`, mín. 1 acción; hard-cap $200 en Ejecutor
- **Breakout filter:** Ejecutor omite órdenes de ruptura si `precio_actual >= entrada_esperada`
- **OCO protection:** `place_advanced_order` (GTC) con stop + target en un solo llamado
- **Regla 1%:** Si `gain_pct >= 1.0` → cancelar OCO → market sell → `Estado="Target Alcanzado - 1%"`
- **Gestión activa:** Si `gain_pct >= 0.5` y caída desde high reciente `>= 0.3%` → cerrar
- **Cierre verificado:** Después de market sell, verificar fill con `get_equity_orders`; reintentar una vez; si no confirma → `Estado="Cierre Forzado - SIN CONFIRMAR"`
