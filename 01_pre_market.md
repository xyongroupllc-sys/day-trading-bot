# Pre-Market — 8:30 AM ET / 5:30 AM PT

**Rutina:** `trig_0192TAKQQiwgKSNNQJ8P9zyd`
**Cron:** `30 12 * * 1-5` (UTC)
**MCPs:** Robinhood-Trading, Notion

---

Corre el análisis pre-apertura del sistema de day trading (Ciclo 'V2 - Ago 2026+', modo totalmente automático sin aprobación): analiza el mercado pre-apertura sobre el universo de acciones líquidas (~$20-100/acción) aptas para la cuenta Agentic (••••2357). Genera una LISTA RANKEADA de 2-3 candidatos concretos con entrada, stop-loss y target (≥1% sobre la entrada planeada) cada uno, ordenados por conviction — el #1 es el que el Ejecutor intentará primero a las 9:35 AM ET, el resto son plan B/C si el mercado no llega a esos niveles. Para cada candidato define el tipo de entrada: 'ruptura' (stop_limit buy por encima del máximo pre-market/nivel de resistencia, para catalizadores con momentum genuino y volumen) o 'retroceso' (limit buy esperando pullback a soporte técnico claro — SOLO si hay soporte confirmado; no uses retroceso si el momentum ya ocurrió, lección en vivo: STM 8/11, SMCI 8/12 — un limit esperando pullback que nunca llegó dejó el día sin operar).

REGLA DE TAMAÑO (obligatoria):
- Objetivo de capital por posición: ~$150 (puede quedar ligeramente por encima si el número de shares lo requiere)
- Tamaño = round(150 / Precio Entrada) → mínimo 1 share
- Ejemplos: $58/acc → round(2.59) = 3 shares ($174 — aceptable) | $48/acc → round(3.13) = 3 shares ($144) | $30/acc → round(5.0) = 5 shares ($150)
- Límite duro del Ejecutor: $200. Si el total llega ahí, hay un error de cálculo, no tolerancia normal.
- NUNCA superar 2 posiciones simultáneas en el mismo día

El Ejecutor colocará las 3 órdenes automáticamente a las 9:35 AM ET (6:35 AM PT) sin intervención manual: stop_limit para 'ruptura', limit para 'retroceso', ambas GFD (se cancelan solas al cierre si no llenan).

Registra cada candidato en la base de Notion "📈 Day Trading — Señales" como registro separado (Ciclo="V2 - Ago 2026+", Estado="Pendiente", Rango="#1"/#2"/#3", Tipo Entrada="ruptura"/"retroceso", Precio Entrada=$, Stop Loss=$, Target Esperado=$, Tamaño=round(150/Precio Entrada) mínimo 1, Catalizador=texto breve) y crea el registro de apertura en "📋 Day Trading Journal" (Tipo="📈 Apertura", Ciclo="V2 - Ago 2026+").

Envía el boletín pre-apertura por Telegram usando Python urllib (NO curl) DIVIDIDO EN 3 MENSAJES SEPARADOS, en este orden:
```python
python3 << 'PYEOF'
import urllib.request, json, os
BOT = 'os.environ['TELEGRAM_BOT_TOKEN']'
CHAT = '526766269'
def tg(msg):
    data = json.dumps({'chat_id': CHAT, 'text': msg}).encode('utf-8')
    req = urllib.request.Request(f'https://api.telegram.org/bot{BOT}/sendMessage', data, {'Content-Type':'application/json'})
    try: urllib.request.urlopen(req, timeout=15)
    except Exception as e: print('TG ERROR:', e)
PYEOF
```

MENSAJE 1 — Resumen Premarket: nivel y variación % de los índices principales (usa get_index_quotes para SPX/NDX/DJI o get_equity_quotes de SPY/QQQ/IWM), sentimiento general (alcista/bajista/mixto), movers relevantes del pre-market, earnings del día (get_earnings_calendar) si hay algo relevante.

MENSAJE 2 — Tabla Resumen de Señales: tabla con los 2-3 candidatos (ticker | tipo | entrada | stop | target | qty | capital estimado | catalizador en una frase). Nota al pie: "✅ Órdenes automáticas — Ejecutor 6:35 AM PT, sin aprobación".

MENSAJE 3 — Análisis Detallado de Cada Señal: por cada candidato, tesis completa, justificación de niveles entrada/stop/target, nivel de invalidación de la tesis. Cierra con resumen de riesgo: capital total comprometido (qty * precio_entrada por candidato) y % del capital en riesgo si todos los stops se activan.

Si algún mensaje supera ~4096 chars, dividirlo en partes dentro del mismo bloque temático pero mantener los 3 bloques separados. TODO en español. Cualquier hora en PT (= ET - 3h en horario de verano), nunca en ET. También intenta la notificación push de Claude como respaldo (puede no llegar, no es bloqueante). Preséntame el boletín completo en este chat.

Si no hay candidatos válidos hoy: enviar un único Telegram con el motivo, NO crear registros en Notion ni Journal, terminar.
Si es feriado NYSE o fin de semana: terminar sin hacer nada ni notificar.
